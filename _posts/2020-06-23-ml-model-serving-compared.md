---
layout: post
title:  ML model serving frameworks compared
date:   2020-06-23 00:00:00 -0700
categories:
tags:
---

Comparisons of a few machine learning model serving tools in the market.

### SageMaker

[https://aws.amazon.com/sagemaker/](https://aws.amazon.com/sagemaker/)

SageMaker provides the entire ML lifecycle management from data preparation to model serving. In terms of model serving, it has native support for models created within the SageMaker ecosystem. It also supports [bring-your-own-inference-code](https://docs.aws.amazon.com/sagemaker/latest/dg/your-algorithms-inference-code.html). The end user deploys a docker container to SagaMaker with a web server that serves inference code. SageMaker manages the docker container by providing availability and auto scaling.

**Platform dependency**: AWS.

**Focus**: The entire ML lifecycle.

**Features**:

- Native support in AWS.

**Concerns**:

- Cost. SageMaker charges a premium on top of the bare EC2 instances.
- Lack of support on deployment and creation of inference service.

### TensorFlow Serving

[https://www.tensorflow.org/tfx/guide/serving](https://www.tensorflow.org/tfx/guide/serving)

TensorFlow Serving is a high-performance model sever. The model server itself is written in C++. It focuses on the specific problem of model serving and relies on other tools for deployment and service management support.

**Platform dependency**: TensorFlow.

**Focus**: Model serving.

**Features**:

- Highly performant. It is optimized for TensorFlow models.
- Support for batching to optimize throughput.
- Model version management.

**Concerns**:

- Works best for Tensorflow. It is unclear on its support for other ML frameworks such as PyTorch.
- Unclear on its server extensibility for custom features.

### BentoML

[https://bentoml.org](https://bentoml.org/)

BentoML focuses mainly on the web service and service management aspect of the model serving. It provides a set of python base classes for end users to override in order to create web service code. It automates some of the deployment activities by automatically figuring out dependencies and creates service packages. BentoML has integration support for various platforms including but not limited to SageMaker, ECS, Google Cloud, KubeFlow and etc.

**Platform dependency**: None.

**Focus**: Model serving.

**Features**:

- Micro-batching support with added performance benefit.
- Can be integrated with a wide range of cloud providers.

**Concerns**:

- Everything is in python including configurations. This can be an advantage for scientists though.

### KFServing / KubeFlow

[https://github.com/kubeflow/kfserving](https://github.com/kubeflow/kfserving)

KFServing is a serving framework under the umbrella of [KubeFlow](https://www.kubeflow.org/), which is a machine learning toolkit built for Kubernetes. Kubeflow manages the entire lifecycle of a machine learning workflow. It can be integrated with multiple serving frameworks including KFServing and TensorFlow Serving.

KubeFlow can be configured to handle auto-scaling, deployment other DevOps activities under the Kubernetes ecosystem. KFServing per se does not do anything additional other than serving as a tool to integrate with the larger Kubeflow / Kubenetes  ecosystem.

**Platform dependency**: Kubernetes.

**Focus**: The entire ML lifecycle.

**Features**:

- Well integrated with Kubenetes / Kubeflow ecosystem.

**Concerns**:

- Cannot be used as a independent framework outside of Kubernetes.
- Relatively steep learning curve.

### Seldon core

[https://www.seldon.io/tech/products/core/](https://www.seldon.io/tech/products/core/)

Seldon core is another tool built on top of Kubernetes. Seldon core prepackages other 3rd party model servers like TensorFlow Serving and MLflow server in order to bring them into the platform. Seldon core itself does not provide any in-house model server. Seldon core serving can be used with KubeFlow and there seems to be a feature convergence with KFServing in the future.

**Platform dependency**: Kubernetes.

**Focus**: The entire ML lifecycle.

### Cortex

[https://www.cortex.dev/](https://www.cortex.dev/)

Cortex is similar to KubeFlow. It takes a different approach by supporting a narrower set of features with a customized command line tool. It focuses more on the problem of service deployment and management.

**Platform dependency**: AWS.

**Focus**: Service deployment and management.

**Features**:

- Simplicity over KubeFlow. Smooth learning curve.

**Concerns**:

- Does not provide any support for web service creation.
- Currently only supports AWS stack. Although there are plans to support Google Cloud and Azure support soon in future.
- Poorer set of features as compared to KubeFlow.

### MLFLow

[https://mlflow.org/](https://mlflow.org/)

MLFlow is another frameworks that manages the whole lifecycle of machine learning. A machine learning model is wrapped under a "MLFlow model" which is a format defined by the project. It supports all common frameworks (TensorFlow, Keras, Pytorch etc).

**Platform dependency**: None.

**Focus**: The entire ML lifecycle.

**Concerns**:

- Models are wrapped by ML custom classes. Unclear on its extensibility and debugging support.