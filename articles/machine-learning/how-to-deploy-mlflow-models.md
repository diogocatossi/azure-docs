---
title: Guidelines for deploying MLflow models
titleSuffix: Azure Machine Learning
description: Learn to deploy your MLflow model to the deployment targets supported by Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: santiagxf
ms.author: fasantia
ms.reviewer: mopeakande
ms.date: 06/06/2022
ms.topic: how-to
ms.custom: deploy, mlflow, devplatv2, no-code-deployment, devx-track-azurecli, cliv2, event-tier1-build-2022
ms.devlang: azurecli
---

# Guidelines for deploying MLflow models

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

> [!div class="op_single_selector" title1="Select the version of Azure Machine Learning CLI extension you are using:"]
> * [v1](./v1/how-to-deploy-mlflow-models.md)
> * [v2 (current version)](how-to-deploy-mlflow-models.md)

In this article, learn how to deploy your [MLflow](https://www.mlflow.org) model to Azure Machine Learning for both real-time and batch inference. Learn also about the different tools you can use to perform management of the deployment.


## Deploying MLflow models vs custom models

When deploying MLflow models to Azure Machine Learning, you don't have to provide a scoring script or an environment for deployment as they are automatically generated for you. We typically refer to this functionality as no-code deployment.

For no-code-deployment, Azure Machine Learning:

* Ensures all the package dependencies indicated in the MLflow model are satisfied.
* Provides a MLflow base image/curated environment that contains the following items:
    * Packages required for Azure Machine Learning to perform inference, including [`mlflow-skinny`](https://github.com/mlflow/mlflow/blob/master/README_SKINNY.rst).
    * A scoring script to perform inference.

> [!WARNING]
> Online Endpoints dynamically installs Python packages provided MLflow model package during container runtime. deploying MLflow models to online endpoints with no-code deployment in a private network without egress connectivity is not supported by the moment. If that's your case, either enable egress connectivity or indicate the environment to use in the deployment as explained in [Customizing MLflow model deployments (Online Endpoints)](how-to-deploy-mlflow-models-online-endpoints.md#customizing-mlflow-model-deployments). This limitation is not present in Batch Endpoints.

### Implications of models with signatures

MLflow models can include a signature that indicates the expected inputs and their types. For those models containing a signature, Azure Machine Learning enforces compliance with it, both in terms of the number of inputs and their types. This means that your data input should comply with the types indicated in the model signature. If the data can't be parsed as expected, the invocation will fail. This applies for both online and batch endpoints.

__MLmodel__

:::code language="yaml" source="~/azureml-examples-main/sdk/python/endpoints/online/mlflow/sklearn-diabetes/model/MLmodel" highlight="13-19":::

You can inspect the model signature of your model by opening the MLmodel file associated with your MLflow model. For more details about how signatures work in MLflow see [Signatures in MLflow](concept-mlflow-models.md#signatures).

> [!TIP]
> Signatures in MLflow models are optional but they are highly encouraged as they provide a convenient way to early detect data compatibility issues. For more information about how to log models with signatures read [Logging models with a custom signature, environment or samples](how-to-log-mlflow-models.md#logging-models-with-a-custom-signature-environment-or-samples).


### Python packages and dependencies

Azure Machine Learning automatically generates environments to run inference of MLflow models. Those environments are built by reading the conda dependencies specified in the MLflow model. Azure Machine Learning also adds any required package to run the inferencing server, which will vary depending on the type of deployment you are doing.

__conda.yaml__

:::code language="yaml" source="~/azureml-examples-main/sdk/python/endpoints/online/mlflow/sklearn-diabetes/model/conda.yaml" highlight="13-19":::

> [!WARNING]
> MLflow performs automatic package detection when logging models, and pins their versions in the conda dependencies of the model. However, such action is performed at the best of its knowledge and there may be cases when the detection doesn't reflect your intentions or requirements. On those cases consider [logging models with a custom conda dependencies definition](how-to-log-mlflow-models.md?#logging-models-with-a-custom-signature-environment-or-samples).


## Deployment tools

Azure Machine Learning offers many ways to deploy MLflow models into Online and Batch endpoints. You can deploy models using the following tools:

> [!div class="checklist"]
> - MLflow SDK
> - Azure ML CLI and Azure ML SDK for Python
> - Azure Machine Learning studio

Each workflow has different capabilities, particularly around which type of compute they can target. The following table shows them.

| Scenario | MLflow SDK | Azure ML CLI/SDK | Azure ML studio |
| :- | :-: | :-: | :-: |
| Deploy to managed online endpoints | [See example](how-to-deploy-mlflow-models-online-progressive.md)<sup>1</sup> | [See example](how-to-deploy-mlflow-models-online-endpoints.md)<sup>1</sup> | [See example](how-to-deploy-mlflow-models-online-endpoints.md?tabs=studio)<sup>1</sup> |
| Deploy to managed online endpoints (with a scoring script) |  | [See example](how-to-deploy-mlflow-models-online-endpoints.md#customizing-mlflow-model-deployments) |  |
| Deploy to batch endpoints |  | [See example](how-to-mlflow-batch.md) | [See example](how-to-mlflow-batch.md?tab=studio) |
| Deploy to batch endpoints (with a scoring script) |  | [See example](how-to-mlflow-batch.md#customizing-mlflow-models-deployments-with-a-scoring-script) |   |
| Deploy to web services (ACI/AKS) | Legacy support<sup>2</sup> | <sup>2</sup> | <sup>2</sup> |
| Deploy to web services (ACI/AKS - with a scoring script) | <sup>2</sup> | <sup>2</sup> | Legacy support<sup>2</sup> |

> [!NOTE]
> - <sup>1</sup> Deployment to online endpoints in private link-enabled workspaces is not supported as public network access is required for package installation. We suggest to deploy with a scoring script on those scenarios.
> - <sup>2</sup> We recommend switching to our [managed online endpoints](concept-endpoints.md) instead.

### Which option to use?

If you are familiar with MLflow or your platform support MLflow natively (like Azure Databricks) and you wish to continue using the same set of methods, use the MLflow SDK. On the other hand, if you are more familiar with the [Azure ML CLI v2](concept-v2.md), you want to automate deployments using automation pipelines, or you want to keep deployments configuration in a git repository; we recommend you to use the [Azure ML CLI v2](concept-v2.md). If you want to quickly deploy and test models trained with MLflow, you can use [Azure Machine Learning studio](https://ml.azure.com) UI deployment.


## Differences between models deployed in Azure Machine Learning and MLflow built-in server

MLflow includes built-in deployment tools that model developers can use to test models locally. For instance, you can run a local instance of a model registered in MLflow server registry with `mlflow models serve -m my_model`. Since Azure Machine Learning online endpoints run our influencing server technology, the behavior of these two services is different.

### Input formats

| Input type | MLflow built-in server | Azure ML Online Endpoints |
| :- | :-: | :-: |
| JSON-serialized pandas DataFrames in the split orientation | **&check;** | **&check;** |
| JSON-serialized pandas DataFrames in the records orientation | Deprecated |  |
| CSV-serialized pandas DataFrames | **&check;** | Use batch<sup>1</sup> |
| Tensor input format as JSON-serialized lists (tensors) and dictionary of lists (named tensors) | **&check;** | **&check;** |
| Tensor input formatted as in TF Serving’s API | **&check;** |  |

> [!NOTE]
> - <sup>1</sup> We suggest you to explore batch inference for processing files. See [Deploy MLflow models to Batch Endpoints](how-to-mlflow-batch.md).

### Input structure

Regardless of the input type used, Azure Machine Learning requires inputs to be provided in a JSON payload, within a dictionary key `input_data`. The following section shows different payload examples and the differences between MLflow built-in server and Azure Machine Learning inferencing server.

> [!WARNING]
> Note that such key is not required when serving models using the command `mlflow models serve` and hence payloads can't be used interchangeably.

> [!IMPORTANT]
> **MLflow 2.0 advisory**: Notice that the payload's structure has changed in MLflow 2.0.

#### Payload example for a JSON-serialized pandas DataFrames in the split orientation

# [Azure Machine Learning](#tab/azureml)

```json
{
    "input_data": {
        "columns": [
            "age", "sex", "trestbps", "chol", "fbs", "restecg", "thalach", "exang", "oldpeak", "slope", "ca", "thal"
        ],
        "index": [1],
        "data": [
            [1, 1, 145, 233, 1, 2, 150, 0, 2.3, 3, 0, 2]
        ]
    }
}
```

# [MLflow built-in server](#tab/builtin)

```json
{
    "dataframe_split": {
        "columns": [
            "age", "sex", "trestbps", "chol", "fbs", "restecg", "thalach", "exang", "oldpeak", "slope", "ca", "thal"
        ],
        "index": [1],
        "data": [
            [1, 1, 145, 233, 1, 2, 150, 0, 2.3, 3, 0, 2]
        ]
    }
}
```

The previous payload corresponds to MLflow server 2.0+.

---


#### Payload example for a tensor input

# [Azure Machine Learning](#tab/azureml)

```json
{
    "input_data": [
          [1, 1, 0, 233, 1, 2, 150, 0, 2.3, 3, 0, 2],
          [1, 1, 0, 233, 1, 2, 150, 0, 2.3, 3, 0, 2]
          [1, 1, 0, 233, 1, 2, 150, 0, 2.3, 3, 0, 2],
          [1, 1, 145, 233, 1, 2, 150, 0, 2.3, 3, 0, 2]
    ]
}
```

# [MLflow built-in server](#tab/builtin)

```json
{
    "inputs": [
          [1, 1, 0, 233, 1, 2, 150, 0, 2.3, 3, 0, 2],
          [1, 1, 0, 233, 1, 2, 150, 0, 2.3, 3, 0, 2]
          [1, 1, 0, 233, 1, 2, 150, 0, 2.3, 3, 0, 2],
          [1, 1, 145, 233, 1, 2, 150, 0, 2.3, 3, 0, 2]
    ]
}
```

---

#### Payload example for a named-tensor input

# [Azure Machine Learning](#tab/azureml)

```json
{
    "input_data": {
        "tokens": [
          [0, 655, 85, 5, 23, 84, 23, 52, 856, 5, 23, 1]
        ],
        "mask": [
          [0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0]
        ]
    }
}
```

# [MLflow built-in server](#tab/builtin)

```json
{
    "inputs": {
        "tokens": [
          [0, 655, 85, 5, 23, 84, 23, 52, 856, 5, 23, 1]
        ],
        "mask": [
          [0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0]
        ]
    }
}
```

---

For more information about MLflow built-in deployment tools see [MLflow documentation section](https://www.mlflow.org/docs/latest/models.html#built-in-deployment-tools).

## How to customize inference when deploying MLflow models

You may be used to author scoring scripts to customize how inference is executed for your models. This is particularly the case when you are using features like `autolog` in MLflow that automatically log models for you as the best of the knowledge of the framework. However, you may need to run inference in a different way.

For those cases, you can either [change how your model is being logged in the training routine](#change-how-your-model-is-logged-during-training) or [customize inference with a scoring script](#customize-inference-with-a-scoring-script)


### Change how your model is logged during training

When you log a model using either `mlflow.autolog` or using `mlflow.<flavor>.log_model`, the flavor used for the model decides how inference should be executed and what gets returned by the model. MLflow doesn't enforce any specific behavior in how the `predict()` function generates results. There are scenarios where you probably want to do some pre-processing or post-processing before and after your model is executed.

A solution to this scenario is to implement machine learning pipelines that moves from inputs to outputs directly. Although this is possible (and sometimes encourageable for performance considerations), it may be challenging to achieve. For those cases, you probably want to [customize how your model does inference using a custom models](how-to-log-mlflow-models.md?#logging-custom-models).

### Customize inference with a scoring script

If you want to customize how inference is executed for MLflow models (or opt-out for no-code deployment) you can refer to [Customizing MLflow model deployments (Online Endpoints)](how-to-deploy-mlflow-models-online-endpoints.md#customizing-mlflow-model-deployments) and [Customizing MLflow model deployments (Batch Endpoints)](how-to-mlflow-batch.md#customizing-mlflow-models-deployments-with-a-scoring-script).

> [!IMPORTANT]
> When you opt-in to indicate a scoring script, you also need to provide an environment for deployment.

## Next steps

To learn more, review these articles:

- [Deploy MLflow models to online endpoints](how-to-deploy-mlflow-models-online-endpoints.md)
- [Progressive rollout of MLflow models](how-to-deploy-mlflow-models-online-progressive.md)
- [Deploy MLflow models to Batch Endpoints](how-to-mlflow-batch.md)
