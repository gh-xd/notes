# TFJob部署

部署TFJob组件到Google Kubernetes Engine

## 目标

- 部署TFJob组件到Google Kubernetes Engine
- 配置使用TFJob的multi-worker distributed训练任务
- 提交以及观察TFJob jobs



## 步骤

1. 创建计算区、设置环境变量和集群
2. 获取集群证书 Credentials
3. 从Kubeflow获取TFJob清单
4. 创建Kubernetes命名空间并安装（给TFJob一个可以host的地方）
5. 创建云端储存
   - 分布式训练脚本会将SavedModel储存到一个位置。代码也需要存放
   - 所以，开一个storage bucket，作为Google Cloud的全局资源
6. 准备TFJob
   - 从Github获取训练代码和manifest模板
   - 训练代码需要打包到docker镜像
   - docker镜像需要推送到项目的容器注册
7. 更新TFJob的manifest
   - `tfjob.yaml`文件
   - 关键要定义tfReplicaSpecs中的Worker区域定义pod的数量
   - 也要在container区域定义其中的镜像位置
8. 提交TFJob
9. 监测TFJob



## 学习资料

- Cousera MLOps 课程