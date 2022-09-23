# Canary部署

使用Kubernetes和Istio的Tensorflow模型

## 目标

- 了解谷歌云平台中Canary部署的步骤
- 操作一个Canary部署

### 工具

- Google Cloud Platform - Google Kubernetes Engine 集群 (GKE Cluster)
- 在GKE cluster中的Istio add-on
- TensorFlow Serving
- ResNet50和Resnet101模型

### 步骤

1. 创建GKE集群，并使用Istio插件
2. 使用TensorFlow Serving部署ResNet模型
3. 配置Istio入口网关 (Istio Ingress Gateway)
4. 配置Istio虚拟服务和目标规则 (Istio virtual services and destination rules)
5. 配置基于权重的路由
6. 配置基于内容的路由



## 1. 配置

### 1.1 获取实验文件

**目的**：从Github获取TFServing Canary文件到根目录下

**结果**：创建的`/home/student_04_xxx/tfserving-canary`目录及其文件

```bash
cd
SRC_REPO=https://github.com/GoogleCloudPlatform/mlops-on-gcp
kpt pkg get $SRC_REPO/workshops/mlep-qwiklabs/tfserving-canary-gke tfserving-canary
cd tfserving-canary
```



### 1.2 计算区和集群

**目的**：在Google Cloud中创建计算区，配置集群名字

```bash
gcloud config set compute/zone us-central1-f
PROJECT_ID=$(gcloud config get-value project)
CLUSTER_NAME=cluster-1
```

**目的**：开启一个带Istio插件的集群

结果：指定`project id`、`addons=Istio`、节点数量`3`的集群

```bash
gcloud beta container clusters create $CLUSTER_NAME \
  --project=$PROJECT_ID \
  --addons=Istio \
  --istio-config=auth=MTLS_PERMISSIVE \
  --cluster-version=latest \
  --machine-type=n1-standard-4 \
  --num-nodes=3
```



### 1.3 获取集群证书

**目的**：可以使用`kubectl`命令

```bash
gcloud container clusters get-credentials $CLUSTER_NAME
```



## 2. Istio配置检查

### 2.1 检查Kubernetes服务

**目的**：查看Istio相关的Kubernetes服务是否已经部署，包含：

- `istio-citadel`
- `istio-galley`
- `istio-pilot`
- `istio-ingressgateway`
- `istio-policy`
- `istio-sidecar-injector`
- `istio-telemetry`

**结果**：控制台打印信息

```bash
kubectl get service -n istio-system
```



### 2.2 检查Kubernetes的Pods

**目的**：查看Pods是否已经在Running

```bash
kubectl get pods -n istio-system
```



### 2.3 配置自动注入

为了利用Istio的所有功能，Istio网格中的pod必须运行一个Istio sidecar代理。

有两种方法将Istio sidecar注入到pod中：

- 手动使用istioctl命令或通过在pod的命名空间中启用自动Istio sidecar注入。手动注入直接修改配置，如部署，并将代理配置注入其中。当在pod的命名空间中启用时，自动注入会在pod创建时使用一个接纳控制器注入代理配置。

- 在这个实验室中，使用的是自动侧车注入。要配置自动sidecar注入，请执行以下命令。

```bash
kubectl label namespace default istio-injection=enabled
```



## 3. 部署ResNet50模型

### 3.1 创建bucket

**目的**：为Google Cloud创建一个储存的区域

**结果**：可以往里面放资源了

```bash
export MODEL_BUCKET=${PROJECT_ID}-bucket
gsutil mb gs://${MODEL_BUCKET}
```



### 3.2 拷贝

**目的**：将准备好的ResNet模型拷贝到储存区

**结果**：ResNet50/101两个模型存在于Google全局资源

```bash
gsutil cp -r gs://workshop-datasets/models/resnet_101 gs://${MODEL_BUCKET}
gsutil cp -r gs://workshop-datasets/models/resnet_50 gs://${MODEL_BUCKET}
```



## 4. 使用yaml进行部署

一共有三个步骤：

1. 创建一个Kubernetes ConfigMap，目的是指向模型存放的地点
2. 创建Kubenertes Deployment，使用TensorFlow Serving的镜像
3. 部署成功后，创建Kubernetes Service，目的是为模型服务提供接口

### 4.1 ConfigMap

修改`MODEL_PATH`，指向模型所储存的位置，这里`qwiklabs-gcp-03-4b91a600a7a2-bucket`指的是Storage的路径

```yaml
# YAML文件截取
apiVersion: v1
kind: ConfigMap
metadata:
  name: tfserving-configs
data:
  MODEL_NAME: image_classifier
  MODEL_PATH: gs://qwiklabs-gcp-03-4b91a600a7a2-bucket/resnet_50
```

然后，创建ConfigMap

```bash
kubectl apply -f tf-serving/configmap-resnet50.yaml
```



### 4.2 Deployment

需要注意，deployment有两个labels：

- app: image-classifier
- version: resnet50

这两个labels在配置Istio流量路由(traffic routing)时==非常重要==

```yaml
# YAML文件截取
...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: image-classifier-resnet50
  namespace: default
  labels:
    app: image-classifier
    version: resnet50
...
```

创建deployment

```bash
kubectl apply -f tf-serving/deployment-resnet50.yaml
```

也可以查看deployments状态

```bash
kubectl get deployments
```



### 4.3 Service

需要注意的是：

- `selector`指向`app-classifier`的意思是：服务将在所有带此`label`的pod上进行负载均衡（在部署ResNet50的情况下，这些pods指的就是部署了ResNet50的pods）
- 服务类型是`ClusterIP`，意思就是，服务所暴露的IP只在集群内可见

```yaml
# YAML文件截取
apiVersion: v1
kind: Service
metadata:
  name: image-classifier
  namespace: default
  labels:
    app: image-classifier
    service: image-classifier
spec:
  type: ClusterIP
  ports:
  - port: 8500
    protocol: TCP
    name: tf-serving-grpc
  - port: 8501
    protocol: TCP
    name: tf-serving-http
  selector:
    app: image-classifier
```



## 5. 配置Istio

### 5.1 打开网关

你使用Istio Ingress网关来管理网状结构的入站和出站流量，让你指定哪些流量要进入和离开网状结构。

与其他控制流量进入系统的机制不同，如Kubernetes Ingress APIs，Istio网关让你使用Istio流量路由的全部功能和灵活性。

`tf-serving/gateway.yaml`的作用：文件中的网关清单将网关配置为为HTTP流量打开80端口。

```yaml
# gateway.yaml文件截取
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: image-classifier-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

创建gateway

```bash
kubectl apply -f tf-serving/gateway.yaml
```



### 5.2 配置虚拟服务以及目的地规则

现在，网关已经创建，但是ResNet50的部署还不能被集群外访问。

##### Virtual Service 和 Destination Rule

虚拟服务以及目的地规则是Istio流量路由功能的关键构建块。通过虚拟服务，你可以配置请求如何被路由到Istio服务网中的服务。每个虚拟服务由一组路由规则组成，这些规则按顺序进行评估，让Istio将每个给定的虚拟服务请求匹配到网状结构中的特定真实目的地。

你将首先配置一个虚拟服务，将所有通过网关80端口发送的请求转发给8501端口的`image-classifier`。回顾一下，`image-classifier`服务被配置为在标有`app: image-classifer`标签的pod之间实现负载平衡。

```yaml
# virtualservice.yaml文件截取
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: image-classifier
spec:
  hosts:
  - "*"
  gateways:
  - image-classifier-gateway
  http:
  - route:
    - destination:
        host: image-classifier
        port:
          number: 8501
```

创建虚拟服务

```bash
kubectl apply -f tf-serving/virtualservice.yaml
```



## 6. 测试ResNet50服务

为了发送请求，需要一个使用Istio gateway暴露的外部IP和端口

设置IP和端口

```bash
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
```

设置gateway的URL

```bash
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
echo $GATEWAY_URL
```

TensorFlow Serving REST的endpoint是：

```bash
http://host:port/v1/models/${MODEL_NAME}[/versions/${VERSION}|/labels/${LABEL}]:predict
```

在这个例子中，endpoint是

```bash
http://$GATEWAY_URL/v1/models/image_classifier:predict.
```



## 7. ResNet101的Canary部署

### 7.1 部署权重为0的ResNet101

在我们的案例中，destination rule使用`version`标签作为选择器，配置了`image-classifier`服务的两个不同子集，以区分ResNet50和ResNet101的部署。

```yaml
# destinationrule.yaml文件截取
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: image-classifier
spec:
  host: image-classifier
  subsets:
  - name: resnet101
    labels:
      version: resnet101
  - name: resnet50
    labels:
      version: resnet50
```

destination rule允许virtual service的两个部署之间路由流量

创建destination rule

```bash
kubectl apply -f tf-serving/destinationrule.yaml
```

然后，重新配置virtual service（5.2节部署过一次）

这一次，给`ResNet50`的权重分配的是100，给`ResNet101`分配的权重是0

```yaml
# virtualservice-weight-100.yaml文件截取
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: image-classifier
spec:
  hosts:
  - "*"
  gateways:
  - image-classifier-gateway
  http:
  - route:
    - destination:
        host: image-classifier
        subset: resnet50
        port:
          number: 8501
      weight: 100
    - destination:
        host: image-classifier
        subset: resnet101
        port:
          number: 8501
      weight: 0
```

创建virtual service

```bash
kubectl apply -f tf-serving/virtualservice-weight-100.yaml
```

然后，部署ResNet101

步骤和第4节一样：

1. 更新ConfigMap `tf-serving/configmap-resnet101.yaml` 指向 `gs://[YOUR_BUCKET]/resnet_101` location.
2. 应用 `tf-serving/configmap-resnet101.yaml` 
3. 应用 `tf-serving/deployment-resnet101.yaml` 



### 7.2 部署权重为30的ResNet101

现在重新配置Istio虚拟服务，使用加权负载平衡在ResNet50和ResNet101模型之间分配流量--70%的请求将进入ResNet50，30%进入ResNet101。

配置`tf-serving/virtualservice-weight-70.yaml`文件

```yaml
# virtualservice-weight-70.yaml文件截取
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: image-classifier
spec:
  hosts:
  - "*"
  gateways:
  - image-classifier-gateway
  http:
  - route:
    - destination:
        host: image-classifier
        subset: resnet50
        port:
          number: 8501
      weight: 70
    - destination:
        host: image-classifier
        subset: resnet101
        port:
          number: 8501
      weight: 30
```

应用

```bash
kubectl apply -f tf-serving/virtualservice-weight-70.yaml
```

然后，当使用请求时，有30%的概率是ResNet101预测的结果

```bash
curl -d @payloads/request-body.json -X POST http://$GATEWAY_URL/v1/models/image_classifier:predict
```



## 8. 请求头部信息的Canary部署

在前面的步骤中，你学到了如何控制细粒度的流量百分比。Istio路由规则允许更复杂的canary测试场景。

在这一节中，你将重新配置虚拟服务，根据**请求头将流量路由到canary部署**。这种方法允许各种场景，包括将来自特定用户组的请求路由到canary发布。让我们假设，来自canary用户的请求将携带一个自定义的头文件`user-group`。如果这个头被设置为`canary`，请求将被路由到ResNet101。

在`match`区域，定义了match的模式，也就是，**如果headers中user-group的内容是canary**，则使用ResNet101模型。

```yaml
# virtualservice-focused-routing.yaml文件截取
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: image-classifier
spec:
  hosts:
  - "*"
  gateways:
  - image-classifier-gateway
  http:
  - match:
    - headers:
        user-group:
          exact: canary
    route:
      - destination:
          host: image-classifier
          subset: resnet101
          port:
            number: 8501
  - route:
    - destination:
        host: image-classifier
        subset: resnet50
        port:
          number: 8501
```

应用

```bash
kubectl apply -f tf-serving/virtualservice-focused-routing.yaml
```



### 8.1 测试不同请求

1. 头部信息不带canary

```bash
curl -d @payloads/request-body.json -X POST http://$GATEWAY_URL/v1/models/image_classifier:predict
```

2. 头部信息带canary

```bash
curl -d @payloads/request-body.json -H "user-group: canary" -X POST http://$GATEWAY_URL/v1/models/image_classifier:predict
```

使用2的时候，自动使用ResNet101模型



## 学习资料

- Coursera的MLOps课程 [Implementing Canary Releases of TensorFlow Model Deployments with Kubernetes and Istio](https://www.cloudskillsboost.google/focuses/18471?parent=catalog)