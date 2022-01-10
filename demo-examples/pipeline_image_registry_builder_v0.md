# About this Jupyter Notebook

@author: Yingding Wang

This notebook demonstrates fetching image from a private container registry from a Python KF pipeline SDK


```python
import sys
```

# Install kfp to build a pipeline
* Build KF pipeline with SDK: https://www.kubeflow.org/docs/components/pipelines/sdk/build-pipeline/
* Current KFP SDK version on pypi.org: https://pypi.org/project/kfp/ 


```python
!{sys.executable} -m pip install --upgrade --user kfp==1.8.10
```

## Examining K8s Environment


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
# from kubernetes import config as k8s_config
#from kubernetes import client as k8s_client
K8_NAME_SPACE = 'kubeflow-kindfor'
```


```python
secret = k8s_client.V1ObjectReference(name="lrzgit-kindfor-playground", namespace=K8_NAME_SPACE)
```


```python
print(type(secret))
print(secret)
print(secret.namespace)
```

    <class 'kubernetes.client.models.v1_object_reference.V1ObjectReference'>
    {'api_version': None,
     'field_path': None,
     'kind': None,
     'name': 'lrzgit-kindfor-playground',
     'namespace': 'kubeflow-kindfor',
     'resource_version': None,
     'uid': None}
    kubeflow-kindfor


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


```python
# Define k8s secret constance, which will be used to get pipeline_config
# k8s_client.V1ObjectReference(name=K8_GIT_SECRET_NAME, namespace=K8_NAME_SPACE)
K8_GIT_SECRET_NAME = 'lrzgit-kindfor-playground'
```

## Getting started with KubeFlow Python SDK

* Python SDK V2: https://www.kubeflow.org/docs/components/pipelines/sdk-v2/python-function-components/
* Python SDK Overview: https://www.kubeflow.org/docs/components/pipelines/sdk/sdk-overview/


```python
EXPERIMENT_NAME = 'test containers pulling'        # Name of the experiment groups runs in the GUI
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



```python
import kfp
import kfp.dsl as dsl
'''
from kfp.v2.dsl import (
    component,
    Input,
    Output,
    Dataset,
    Metrics
)
'''
```




    '\nfrom kfp.v2.dsl import (\n    component,\n    Input,\n    Output,\n    Dataset,\n    Metrics\n)\n'




```python
def nothing():
    pass

def add(a: float, b: float) -> float:
    '''Calculates sum of two arguments'''
    print(a, '+', b, '=', a + b)
    return a + b

custom_image_op = kfp.components.create_component_from_func(
    func=add,
    output_component_file='playground_component.yaml', # This is optional. It saves the component spec for future use.
    base_image=BASE_IMAGE,
    #packages_to_install=['']
)
```


```python
@dsl.pipeline(
    name = EXPERIMENT_NAME,
    description = EXPERIMENT_DESC
)
def custom_image_pipeline(
    a: float = 0,
    b: float = 7
):
    first_task = custom_image_op(a, b)
```

### Setting imagePullSecretes in K8s with Pipeline config
* Setting imagePullSecretes for Pipeline with SDK: https://github.com/kubeflow/pipelines/issues/5843#issuecomment-859799181


```python
# from kubernetes import client as k8s_client
pipeline_config = kfp.dsl.PipelineConf()
if (use_custom_image):
    pipeline_config.set_image_pull_secrets([k8s_client.V1ObjectReference(name=K8_GIT_SECRET_NAME, namespace=K8_NAME_SPACE)])
    # pipeline_config.set_image_pull_policy("IfNotPresent")
    pipeline_config.set_image_pull_policy("Always")
arguments = {'a': '7', 'b': '8'}
```


```python
client = kfp.Client()
client.create_run_from_pipeline_func(
    pipeline_func=custom_image_pipeline,
    arguments = arguments,
    pipeline_conf=pipeline_config,
    experiment_name=EXPERIMENT_NAME,
    namespace=K8_NAME_SPACE,
    # mode=kfp.dsl.PipelineExecutionMode.V2_COMPATIBLE # can not run wiht V2_COMPATIBLE
)
```


<a href="/pipeline/#/experiments/details/a2ad1ecc-359c-4706-8eca-e64dadaff507" target="_blank" >Experiment details</a>.



<a href="/pipeline/#/runs/details/1812de52-78ec-4885-844b-ece2a216c38e" target="_blank" >Run details</a>.





    RunPipelineResult(run_id=1812de52-78ec-4885-844b-ece2a216c38e)




```python

```
