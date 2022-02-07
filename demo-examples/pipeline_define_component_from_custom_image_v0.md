# About this Jupyter Notebook

@author: Yingding Wang

This notebook demonstrates how to generated a component.yaml definition with kfp sdk and load this component definition in a kfp sdk defined pipeline later on.


```python
import sys
```

# Install kfp to build a pipeline
* Build KF pipeline with SDK: https://www.kubeflow.org/docs/components/pipelines/sdk/build-pipeline/
* Current KFP SDK version on pypi.org: https://pypi.org/project/kfp/ 


```python
!{sys.executable} -m pip install --upgrade --user kfp==1.8.11
```


```python
!{sys.executable} -m pip show kubernetes
```

    Name: kubernetes
    Version: 18.20.0
    Summary: Kubernetes python client
    Home-page: https://github.com/kubernetes-client/python
    Author: Kubernetes
    Author-email: 
    License: Apache License Version 2.0
    Location: /home/jovyan/.local/lib/python3.8/site-packages
    Requires: websocket-client, urllib3, pyyaml, setuptools, google-auth, six, python-dateutil, requests, requests-oauthlib, certifi
    Required-by: kfp, kfserving


Name: kubernetes
Version: 12.0.1
Summary: Kubernetes python client
Home-page: https://github.com/kubernetes-client/python
Author: Kubernetes
Author-email: 
License: Apache License Version 2.0
Location: /opt/conda/lib/python3.8/site-packages
Requires: pyyaml, requests-oauthlib, setuptools, certifi, six, urllib3, websocket-client, google-auth, requests, python-dateutil
Required-by: kfp, kfserving

## Restart the Kernal


```python
from platform import python_version
print (f"current platform python version: {python_version()}")
```

    current platform python version: 3.8.10



```python
# run kubectl command line to see the quota in the name space
!kubectl describe quota
```

## Create K8s secret
* k8s python client API doc: https://github.com/kubernetes-client/python


```python
from kubernetes import client as k8s_client
from kubernetes import config as k8s_config

K8_NAME_SPACE = 'kubeflow-kindfor'
K8_GIT_SECRET_NAME = 'lrzgit-kindfor-playground'
```

### Construct an empty k8s V1Object Reference

Use the following line to construct an empty K8s V1Object for KFP Python SDK.
```python
secret = k8s_client.V1ObjectReference(name=K8_GIT_SECRET_NAME, namespace=K8_NAME_SPACE)
```


```python
secret = k8s_client.V1ObjectReference(name=K8_GIT_SECRET_NAME, namespace=K8_NAME_SPACE)

print(type(secret))
print(secret)
print(secret.namespace)
```

### create a secret with kubectl

1. Create a Deployment Token on GIT LAB with an optional user name, and readonly_registory to pull image
2. SSH Login to your k8s cluster and manually create a docker-registry secret with
```console
kubectl -n kubeflow-kindfor create secret docker-registry <K8_GIT_SECRET_NAME> --docker-server=<GIT_URI> --docker-email=<an_abitrary_email> --docker-password=<deployment_token_string> --docker-username=<deployment_token_user_name>
```

Reference:
* https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

Notice:\
use the following command to examin the docker-registry secret you created in terminal.
```console
kubectl -n <kubeflow_namespace> get secret <K8_GIT_SECRET_NAME> --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
```

## Getting started with KubeFlow Python SDK

* Python SDK V2: https://www.kubeflow.org/docs/components/pipelines/sdk-v2/python-function-components/
* Python SDK Overview: https://www.kubeflow.org/docs/components/pipelines/sdk/sdk-overview/


```python
EXPERIMENT_NAME = 'customcontainer'        # Name of the experiment groups runs in the GUI
EXPERIMENT_DESC = 'Test pulling container with kfp SDK from a private image registory'
use_custom_image = True
#use_custom_image = False
if (use_custom_image):
    BASE_IMAGE = f"gitlab.lrz.de:5005/dzkj/playground:branch-docker-cd"
else:
    BASE_IMAGE = f"library/python:{python_version()}"
print(f"BASE_IMAGE: {BASE_IMAGE}")
```

    BASE_IMAGE: gitlab.lrz.de:5005/dzkj/playground:branch-docker-cd


### Generate Component Yaml definition

* Generate Component Specification: https://github.com/kubeflow/pipelines/issues/3748#issuecomment-627698554


```python
from kfp.components.structures import (
    ComponentSpec, InputSpec, OutputSpec, ContainerImplementation, 
    ContainerSpec, InputValuePlaceholder, InputPathPlaceholder, OutputPathPlaceholder,
)
    
# Define a custom-component.yaml
component_spec = ComponentSpec(
    name='custom op',
    description='custom op from a container registry',
    inputs=[
        InputSpec(name='input1', type='String'),
    ],
    outputs=[
        OutputSpec(name='output1', type='GCSPath'),
    ],
    implementation=ContainerImplementation(container=ContainerSpec(
        image=BASE_IMAGE,
        command=[
            "python", "/app/app.py",
            "--input1", InputValuePlaceholder('input1'),
            "--output1-path", OutputPathPlaceholder('output1') # use the output definition as input arg for the python script file
        ],
    ))
)

component_spec.save('custom-component.yaml')
```

Load Component from Yaml for pipeline
* https://www.kubeflow.org/docs/components/pipelines/sdk/component-development/


```python
import kfp
import kfp.dsl as dsl
import kfp.components as components
```


```python
custom_image_op_nothing = components.load_component_from_file("./custom-component.yaml")
```


```python
@dsl.pipeline(
    name = EXPERIMENT_NAME,
    description = EXPERIMENT_DESC
)
def custom_image_pipeline_nothing(input1: str = "bob"):
    first_task = custom_image_op_nothing(input1)    
```

### Setting imagePullSecretes in K8s with Pipeline config
* Setting imagePullSecretes for Pipeline with SDK: https://github.com/kubeflow/pipelines/issues/5843#issuecomment-859799181


```python
# from kubernetes import client as k8s_client
pipeline_config = dsl.PipelineConf()
if (use_custom_image):
    pipeline_config.set_image_pull_secrets([k8s_client.V1ObjectReference(name=K8_GIT_SECRET_NAME, namespace=K8_NAME_SPACE)])
    pipeline_config.set_image_pull_policy("IfNotPresent")
    # pipeline_config.set_image_pull_policy("Always")

arguments_nothing = {"input1": "kubeflow"}
```


```python
client = kfp.Client()

client.create_run_from_pipeline_func(
    pipeline_func=custom_image_pipeline_nothing,
    arguments = arguments_nothing,
    pipeline_conf=pipeline_config,
    experiment_name=EXPERIMENT_NAME,
    namespace=K8_NAME_SPACE,
    # mode=kfp.dsl.PipelineExecutionMode.V2_COMPATIBLE # can not run wiht V2_COMPATIBLE
)
```


<a href="/pipeline/#/experiments/details/9ca26f1f-fbde-400b-b460-526ad2ab4f28" target="_blank" >Experiment details</a>.



<a href="/pipeline/#/runs/details/12b0f904-da59-4cd0-8929-c47b03ba3906" target="_blank" >Run details</a>.





    RunPipelineResult(run_id=12b0f904-da59-4cd0-8929-c47b03ba3906)




```python

```
