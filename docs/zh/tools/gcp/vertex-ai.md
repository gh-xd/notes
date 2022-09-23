

# Vertex Pipelines

## 目标

- 学习使用Vertext AI集成ML服务的步骤



### 什么是Vertext AI

Vertex AI跨Google Cloud集成了ML服务

以前，AutoML训练模型，不同的自定义模型可以通过单独的服务访问

现在，Vertex AI将两者结合到一个单独的API。它也包含了MLOps产品，例如Vertex Pipelines

从这个任务可以学到如何使用Vertex Pipelines创建和运行ML pipelines：

1. 使用Kubeflow Pipeline SDK构建一个可扩展的ML pipelines
2. 创建和运行一个3步的例子
3. 创建和运行一个训练、评估、部署AutoML分类模型的pipeline
4. 使用google_cloud_pipeline_components库内建的组件与Vertex AI服务互动，
5. 使用Cloud Scheduler调度pipeline任务



## 1. 创建Notebook

在Sidebar导航栏选择`Vertext AI > Workbench > Open JupyterLab`



## 2. 设置Vertex管道

### 2.1 安装依赖

代码与含义：

```python
# 全局变量
USER_FLAG = "--user"

# 安装google-cloud-apiplatform和kfp google-cloud-piepeline-components
!pip3 install {USER_FLAG} google-cloud-aiplatform==1.0.0 --upgrade
!pip3 install {USER_FLAG} kfp google-cloud-pipeline-components==0.1.1 --upgrade

# 自动重启kernel
import os
if not os.getenv("IS_TESTING"):
    # Automatically restart kernel after installs
    import IPython
    app = IPython.Application.instance()
    app.kernel.do_shutdown(True)
    
# 检查安装是否成功，KFP SDK版本应该要大于1.6
!python3 -c "import kfp; print('KFP SDK version: {}'.format(kfp.__version__))"
!python3 -c "import google_cloud_pipeline_components; print('google_cloud_pipeline_components version: {}'.format(google_cloud_pipeline_components.__version__))"
```



google-cloud-apiplatform

KFP google-cloud-pipeline-components



### 2.2 设置项目ID和bucket

如果不知道项目ID，则打印：

```python
import os
PROJECT_ID = ""
# Get your Google Cloud project ID from gcloud
if not os.getenv("IS_TESTING"):
    shell_output=!gcloud config list --format 'value(core.project)' 2>/dev/null
    PROJECT_ID = shell_output[0]
    print("Project ID: ", PROJECT_ID)
```



设置`BUCKET_NAME`，后面要用到：

```python
BUCKET_NAME="gs://" + PROJECT_ID + "-bucket"
```



### 2.3 导入构建Pipeline的依赖

```python
from typing import NamedTuple
import kfp
from kfp import dsl
from kfp.v2 import compiler
from kfp.v2.dsl import (Artifact, Dataset, Input, InputPath, Model, Output,
                        OutputPath, ClassificationMetrics, Metrics, component)
from kfp.v2.google.client import AIPlatformClient
from google.cloud import aiplatform
from google_cloud_pipeline_components import aiplatform as gcc_aip
```



### 2.4 设置常量

设置需要用的常量：

- `PIPELINE_ROOT`是Cloud Storage储存artifacts的路径
- `us-central1`是创建云服务时选择的region

```python
PATH=%env PATH
%env PATH={PATH}:/home/jupyter/.local/bin
REGION="us-central1"
PIPELINE_ROOT = f"{BUCKET_NAME}/pipeline_root/"
PIPELINE_ROOT
```



## 3. 一个简单的练习

### 3.1 内容摘要

目标：

- 学习如何在KFP SDK里创建自定义的组件
- 学习如何运行和监测Vertext管道

内容：

- **功能组件**：创建一个`product_name()`
  - **输入**：为`product name`
  - **处理**：没有处理
  - **输出**：product name的`string`
- **功能组件**：创建一个`emoji()`
  - **输入**：表情包的`text description`
  - **处理**：将表情包的文本转化为表情包，例如sparkle 到 :sparkles:
  - **输出**：表情包，例如:sparkles:
- **打包组件**：创建一个`build_sentence()`
  - **输入**：前两个功能函数的输出
  - **功能**：用这两个输出（包含emoji表情包）构建一个句子
  - **输出**：带emoji表情的句子



### 3.2 组件和函数

#### 功能组件 product_name()

```python
# 解释：@component装饰器作用：当执行pipeline时，自动编译这个函数
# 解释：base_image是组件会使用的容器镜像名
# 解释：output_component_file是可选项，会自动生成yaml文件。当需要分享这个组件给别人，可以使用这个yaml文件来导入组件
@component(base_image="python:3.9", output_component_file="first-component.yaml")
def product_name(text: str) -> str:
    return text

# 分享yaml和导入组件
product_name_component = kfp.components.load_component_from_file('./first-component.yaml')
```

#### 功能组件 emoji()

```python
# 解释：packages_to_install是这个组件需要安装的包
# 解释：这个组件返回一个NamedTuple作为输出
@component(packages_to_install=["emoji"])
def emoji(
    text: str,
) -> NamedTuple(
    "Outputs",
    [
        ("emoji_text", str),  # Return parameters
        ("emoji", str),
    ],
):
    import emoji
    emoji_text = text
    emoji_str = emoji.emojize(':' + emoji_text + ':', use_aliases=True)
    print("output one: {}; output_two: {}".format(emoji_text, emoji_str))
    return (emoji_text, emoji_str)
```

#### 打包组件 build_sentence()

```python
@component
def build_sentence(
    product: str,
    emoji: str,
    emojitext: str
) -> str:
    print("We completed the pipeline, hooray!")
    end_str = product + " is "
    if len(emoji) > 0:
        end_str += emoji
    else:
        end_str += emojitext
    return(end_str)
```



### 3.3 组件到Pipeline

每一个定义好的组件可以为Pipeline创建步骤

设置pipeline需要使用`@dsl.pipeline`装饰器，给一个`name`、一个`description`以及给一个`path`来写入pipeline所创建的artifacts

Artifact指的是任何pipeline输出的文件（这个练习中不会输出任何文件）

```python
# from kfp import dsl

@dsl.pipeline(
    name="hello-world",
    description="An intro pipeline",
    pipeline_root=PIPELINE_ROOT,
)
# You can change the `text` and `emoji_str` parameters here to update the pipeline output
# 指定pipeline初始的输入，定义各个步骤如何连接
def intro_pipeline(text: str = "Vertex Pipelines", emoji_str: str = "sparkles"):
    product_task = product_name(text)
    emoji_task = emoji(emoji_str)
    consumer_task = build_sentence(
        product_task.output,
        emoji_task.outputs["emoji"],
        emoji_task.outputs["emoji_text"],
    )
```



### 3.4 编译运行pipeline

#### 3.4.1 编译

创建一个json文件，需要使用它来运行pipeline

```bash
# from kfp.v2 import compiler

# 指定用dsl.pipeline装饰的组件
compiler.Compiler().compile(
    pipeline_func=intro_pipeline, package_path="intro_pipeline_job.json"
)
```



#### 3.4.2 实例化API客户端

```bash
# from kfp.v2.google.client import AIPlatformClient

# 指定项目ID和Region
api_client = AIPlatformClient(
    project_id=PROJECT_ID,
    region=REGION,
)
```



#### 3.4.3 运行pipeline

```bash
response = api_client.create_run_from_job_spec(

		# 使用前面生成的JSON来运行pipeline
    job_spec_path="intro_pipeline_job.json",
    # pipeline_root=PIPELINE_ROOT  # this argument is necessary if you did not specify PIPELINE_ROOT as part of the pipeline definition.
)
```



## 4. 创建端到端的ML管道

#### 4.1 模型评估的自定义组件

以下代码做了几件事情：

1. 从训练的AutoML分类模型获取模型评估的**metrics**
2. 解析**metrics**并且渲染到Vertex Pipelines UI
3. 将**metrics**与设定的**threshold**进行比较，以确定模型是否应该被部署

```python
@component(
    base_image="gcr.io/deeplearning-platform-release/tf2-cpu.2-3:latest",
    output_component_file="tables_eval_component.yaml", # Optional: you can use this to load the component later
    packages_to_install=["google-cloud-aiplatform"],
)
def classif_model_eval_metrics(
    project: str,
    location: str,  # "us-central1",
    api_endpoint: str,  # "us-central1-aiplatform.googleapis.com",
    thresholds_dict_str: str,
    model: Input[Model],
    metrics: Output[Metrics],
    metricsc: Output[ClassificationMetrics],
) -> NamedTuple("Outputs", [("dep_decision", str)]):  # Return parameter.
    """This function renders evaluation metrics for an AutoML Tabular classification model.
    It retrieves the classification model evaluation generated by the AutoML Tabular training
    process, does some parsing, and uses that info to render the ROC curve and confusion matrix
    for the model. It also uses given metrics threshold information and compares that to the
    evaluation results to determine whether the model is sufficiently accurate to deploy.
    """
    import json
    import logging
    from google.cloud import aiplatform
    
    # Fetch model eval info
    def get_eval_info(client, model_name):
        from google.protobuf.json_format import MessageToDict
        response = client.list_model_evaluations(parent=model_name)
        metrics_list = []
        metrics_string_list = []
        for evaluation in response:
            print("model_evaluation")
            print(" name:", evaluation.name)
            print(" metrics_schema_uri:", evaluation.metrics_schema_uri)
            metrics = MessageToDict(evaluation._pb.metrics)
            for metric in metrics.keys():
                logging.info("metric: %s, value: %s", metric, metrics[metric])
            metrics_str = json.dumps(metrics)
            metrics_list.append(metrics)
            metrics_string_list.append(metrics_str)
        return (
            evaluation.name,
            metrics_list,
            metrics_string_list,
        )
    # Use the given metrics threshold(s) to determine whether the model is
    # accurate enough to deploy.
    def classification_thresholds_check(metrics_dict, thresholds_dict):
        for k, v in thresholds_dict.items():
            logging.info("k {}, v {}".format(k, v))
            if k in ["auRoc", "auPrc"]:  # higher is better
                if metrics_dict[k] < v:  # if under threshold, don't deploy
                    logging.info(
                        "{} < {}; returning False".format(metrics_dict[k], v)
                    )
                    return False
        logging.info("threshold checks passed.")
        return True
    def log_metrics(metrics_list, metricsc):
        test_confusion_matrix = metrics_list[0]["confusionMatrix"]
        logging.info("rows: %s", test_confusion_matrix["rows"])
        # log the ROC curve
        fpr = []
        tpr = []
        thresholds = []
        for item in metrics_list[0]["confidenceMetrics"]:
            fpr.append(item.get("falsePositiveRate", 0.0))
            tpr.append(item.get("recall", 0.0))
            thresholds.append(item.get("confidenceThreshold", 0.0))
        print(f"fpr: {fpr}")
        print(f"tpr: {tpr}")
        print(f"thresholds: {thresholds}")
        metricsc.log_roc_curve(fpr, tpr, thresholds)
        # log the confusion matrix
        annotations = []
        for item in test_confusion_matrix["annotationSpecs"]:
            annotations.append(item["displayName"])
        logging.info("confusion matrix annotations: %s", annotations)
        metricsc.log_confusion_matrix(
            annotations,
            test_confusion_matrix["rows"],
        )
        # log textual metrics info as well
        for metric in metrics_list[0].keys():
            if metric != "confidenceMetrics":
                val_string = json.dumps(metrics_list[0][metric])
                metrics.log_metric(metric, val_string)
        # metrics.metadata["model_type"] = "AutoML Tabular classification"
    logging.getLogger().setLevel(logging.INFO)
    aiplatform.init(project=project)
    # extract the model resource name from the input Model Artifact
    model_resource_path = model.uri.replace("aiplatform://v1/", "")
    logging.info("model path: %s", model_resource_path)
    client_options = {"api_endpoint": api_endpoint}
    # Initialize client that will be used to create and send requests.
    client = aiplatform.gapic.ModelServiceClient(client_options=client_options)
    eval_name, metrics_list, metrics_str_list = get_eval_info(
        client, model_resource_path
    )
    logging.info("got evaluation name: %s", eval_name)
    logging.info("got metrics list: %s", metrics_list)
    log_metrics(metrics_list, metricsc)
    thresholds_dict = json.loads(thresholds_dict_str)
    deploy = classification_thresholds_check(metrics_list[0], thresholds_dict)
    if deploy:
        dep_decision = "true"
    else:
        dep_decision = "false"
    logging.info("deployment decision is %s", dep_decision)
    return (dep_decision,)
```



#### 4.2 Google Cloud内置的组件

使用timestamp设置一个展示名

```python
import time
DISPLAY_NAME = 'automl-beans{}'.format(str(int(time.time())))
print(DISPLAY_NAME)
```



以下代码做了一些事情：

1. 定义pipeline需要的输入参数（需要手动设置，因为它们与其他组件的输出无关）
2. 一些与Vertext AI交互的内置组件
   - **TabularDatasetCreateOp**
     - 输入：指定的数据源（来自Cloud Storage或者BigQuery）
     - 这个练习中，通过BigQuery table URL来传递数据
   - **AutoMLTabularTrainingJobRunOp**
     - 功能：为输入的数据开启AutoML训练工作
     - 输入：一些配置参数例如模型类型（这里是分类）、数据列的一些内容、希望训练多久、数据源配置等等
   - **ModelDeployOp**
     - 部署一个指定模型到Vertex AI的endpoint
     - 需要指定一些额外配置项，例如endpoint的机器类型、项目、模型等

```python
@kfp.dsl.pipeline(name="automl-tab-beans-training-v2",
                  pipeline_root=PIPELINE_ROOT)
def pipeline(
    bq_source: str = "bq://aju-dev-demos.beans.beans1",
    display_name: str = DISPLAY_NAME,
    project: str = PROJECT_ID,
    gcp_region: str = "us-central1",
    api_endpoint: str = "us-central1-aiplatform.googleapis.com",
    thresholds_dict_str: str = '{"auRoc": 0.95}',
):
  
  	# 内置组件 TabularDatasetCreateOp
    dataset_create_op = gcc_aip.TabularDatasetCreateOp(
        project=project, display_name=display_name, bq_source=bq_source
    )
    
    # 内置组件 AutoMLTabularTrainingJobRunOp
    training_op = gcc_aip.AutoMLTabularTrainingJobRunOp(
        project=project,
        display_name=display_name,
        optimization_prediction_type="classification",
        budget_milli_node_hours=1000,
        column_transformations=[
            {"numeric": {"column_name": "Area"}},
            {"numeric": {"column_name": "Perimeter"}},
            {"numeric": {"column_name": "MajorAxisLength"}},
            {"numeric": {"column_name": "MinorAxisLength"}},
            {"numeric": {"column_name": "AspectRation"}},
            {"numeric": {"column_name": "Eccentricity"}},
            {"numeric": {"column_name": "ConvexArea"}},
            {"numeric": {"column_name": "EquivDiameter"}},
            {"numeric": {"column_name": "Extent"}},
            {"numeric": {"column_name": "Solidity"}},
            {"numeric": {"column_name": "roundness"}},
            {"numeric": {"column_name": "Compactness"}},
            {"numeric": {"column_name": "ShapeFactor1"}},
            {"numeric": {"column_name": "ShapeFactor2"}},
            {"numeric": {"column_name": "ShapeFactor3"}},
            {"numeric": {"column_name": "ShapeFactor4"}},
            {"categorical": {"column_name": "Class"}},
        ],
        dataset=dataset_create_op.outputs["dataset"],
        target_column="Class",
    )
    model_eval_task = classif_model_eval_metrics(
        project,
        gcp_region,
        api_endpoint,
        thresholds_dict_str,
        training_op.outputs["model"],
    )
    with dsl.Condition(
        model_eval_task.outputs["dep_decision"] == "true",
        name="deploy_decision",
    ):
      
      	# 内置组件 ModelDeployOp
        deploy_op = gcc_aip.ModelDeployOp(  # noqa: F841
            model=training_op.outputs["model"],
            project=project,
            machine_type="n1-standard-4",
        )
```



#### 4.3 编译运行ML管道

编译：

```bash
compiler.Compiler().compile(
    pipeline_func=pipeline, package_path="tab_classif_pipeline.json"
)
```

开启pipeline

```python
response = api_client.create_run_from_job_spec(
    "tab_classif_pipeline.json", pipeline_root=PIPELINE_ROOT,
    parameter_values={"project": PROJECT_ID,
                      "display_name": DISPLAY_NAME}
)
```



## 5. 对比指标

如果运行了这个pipeline多次，会希望对比不同的metrics

可以使用`aiplatform.get_pipeline_df()`方法来访问metadata

通过以下代码可以获取pipeline的一个DataFrame

```python
pipeline_df = aiplatform.get_pipeline_df(pipeline="automl-tab-beans-training-v2")
small_pipeline_df = pipeline_df.head(2)
small_pipeline_df
```



## 学习资料

- Cousera MLOps课程
