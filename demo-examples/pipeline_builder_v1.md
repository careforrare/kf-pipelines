# About this Jupyter Notebook

@author: Yingding Wang

### Useful JupyterLab Basic

Before start, you may consider to update the jupyterlab with the command

<code>python
!{sys.executable} -m pip install --upgrade --user jupyterlab    
</code>  

1. Autocomplete syntax with "Tab"
2. View Doc String with "Shift + Tab"
3. mark the code snippet -> select with right mouse -> Show Contextual Help (see the function code)


```python
# import sys, os
# %env
```


```python
import sys, os
print(f"Sys version: {sys.version}")

# os.environ["KF_PIPELINES_SA_TOKEN_PATH"]="/var/run/secrets/kubernetes.io/serviceaccount/token"
# os.environ["KF_PIPELINES_SA_TOKEN_PATH"]="/var/run/secrets/kubeflow/pipelines/token"
```

    Sys version: 3.8.10 | packaged by conda-forge | (default, May 11 2021, 07:01:05) 
    [GCC 9.3.0]



```python
!{sys.executable} -m pip show jupyterlab # 3.0.16
# !{sys.executable} -m pip show jupyter_contrib_nbextensions
```

    Name: jupyterlab
    Version: 3.2.8
    Summary: JupyterLab computational environment
    Home-page: https://jupyter.org
    Author: Jupyter Development Team
    Author-email: jupyter@googlegroups.com
    License: UNKNOWN
    Location: /home/jovyan/.local/lib/python3.8/site-packages
    Requires: ipython, packaging, jupyter-server, jinja2, tornado, jupyter-core, jupyterlab-server, nbclassic
    Required-by: 



```python
# update the jupyter lab
#!{sys.executable} -m pip install --upgrade --user jupyterlab
```


```python
"""upgrade the kfp server api version to 1.7.0 for KF 1.4"""
# !{sys.executable} -m pip uninstall -y kfp-server-api
# !{sys.executable} -m pip install --user --upgrade kfp-server-api==1.7.0
```




    'upgrade the kfp server api version to 1.7.0 for KF 1.4'




```python
import sys
!{sys.executable} -m pip install --upgrade --user kfp==1.8.12
!{sys.executable} -m pip install --upgrade --user kubernetes==18.20.0
#!{sys.executable} -m pip install --upgrade --user kubernetes==21.7.0
```

# Restart the kernal
After update the kfp, restart this notebook kernel

Jupyter notebook: Meun -> Kernel -> restart kernel

## Check the KubeFlow Pipeline version on the server side


```python
!{sys.executable} -m pip list | grep kfp
```

    kfp                          1.8.12
    kfp-pipeline-spec            0.1.14
    kfp-server-api               1.7.0


### Check my KubeFlow namespace total resource limits


```python
# run command line to see the quota
!kubectl describe quota
```

    Name:                                                         kf-resource-quota
    Namespace:                                                    kubeflow-kindfor
    Resource                                                      Used    Hard
    --------                                                      ----    ----
    basic-csi.storageclass.storage.k8s.io/persistentvolumeclaims  3       5
    basic-csi.storageclass.storage.k8s.io/requests.storage        11Gi    50Gi
    cpu                                                           2110m   128
    longhorn.storageclass.storage.k8s.io/persistentvolumeclaims   0       10
    longhorn.storageclass.storage.k8s.io/requests.storage         0       500Gi
    memory                                                        2108Mi  512Gi


## Setup
Example Pipeline from

https://github.com/kubeflow/examples/tree/master/pipelines/simple-notebook-pipeline

## Getting started with Python function-based components

https://www.kubeflow.org/docs/components/pipelines/sdk/python-function-components/


```python
from platform import python_version

EXPERIMENT_NAME = 'core kf test'        # Name of the experiment in the UI
EXPERIMENT_DESC = 'testing KF platform'
# BASE_IMAGE = f"library/python:{python_version()}" # Base image used for components in the pipeline, which has not root
BASE_IMAGE = "python:3.8.13"
NAME_SPACE = "kubeflow-kindfor" # change namespace if necessary
```


```python
import kfp
import kubernetes
import kfp.dsl as dsl
import kfp.compiler as compiler
import kfp.components as components
```

## Connecting KFP Python SDK from Notebook to Pipeline

* https://www.kubeflow.org/docs/components/pipelines/sdk/connect-api/


```python
print(kfp.__version__)
print(kubernetes.__version__)
```

    1.8.12
    18.20.0



```python
def add(a: float, b: float) -> float:
    '''Calculates sum of two arguments'''
    print(a, '+', b, '=', a + b)
    return a + b
```

### Create component from function

https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.components.html



```python
# returns a task factory function
add_op = components.create_component_from_func(
    add,
    output_component_file='add_component.yaml',
    base_image=BASE_IMAGE,
    packages_to_install=None  
)
```

### Add pod memory and cpu restriction

https://github.com/kubeflow/pipelines/pull/5695


```python
'''
def pod_defaults_transformer(op: dsl.ContainerOp):
    op.set_memory_request('100Mi') # op.set_memory_limit('1000Mi')
    op.set_memory_limit('100Mi')
    op.set_cpu_request('100m') # 1 core, # op.set_cpu_limit('1000m')
    op.set_cpu_limit('1000m') 
    return op
'''
```




    "\ndef pod_defaults_transformer(op: dsl.ContainerOp):\n    op.set_memory_request('100Mi') # op.set_memory_limit('1000Mi')\n    op.set_memory_limit('100Mi')\n    op.set_cpu_request('100m') # 1 core, # op.set_cpu_limit('1000m')\n    op.set_cpu_limit('1000m') \n    return op\n"




```python
def pod_defaults_transformer(op: dsl.ContainerOp):
    """
    op.set_memory_limit('1000Mi') = 1GB
    op.set_cpu_limit('1000m') = 1 cpu core
    """
    return op.set_memory_request('200Mi')\
            .set_memory_limit('1000Mi')\
            .set_cpu_request('2000m')\
            .set_cpu_limit('2000m')
```


```python
@dsl.pipeline(
   name='Calculation pipeline', 
   description='A toy pipeline that performs arithmetic calculations.'
)
def calc_pipeline(
   a: float =0,
   b: float =7
):
    # Passing pipeline parameter and a constant value as operation arguments
    # first_add_task = add_op(a, 4)
    first_add_task = pod_defaults_transformer(add_op(a, 4))
    # no value taken from cache
    first_add_task.execution_options.caching_strategy.max_cache_staleness = "P0D"
    # second_add_task = add_op(first_add_task.output, b)
    second_add_task = pod_defaults_transformer(add_op(first_add_task.output, b))
    # no cache 
    second_add_task.execution_options.caching_strategy.max_cache_staleness = "P0D"
```

### (optional step) Compile the pipeline to see the settings


```python
PIPE_LINE_FILE_NAME="calc_pipeline_with_resource_limit"
kfp.compiler.Compiler().compile(calc_pipeline, f"{PIPE_LINE_FILE_NAME}.yaml")
```

# Run Pipeline with Multi-user Isolation

https://www.kubeflow.org/docs/components/pipelines/multi-user/


```python
# get the pipeline host from env set up be the notebook instance
client = kfp.Client()

# Make sure the volume is mounted /run/secrets/kubeflow/pipelines 
# client.get_experiment(experiment_name=EXPERIMENT_NAME, namespace=NAME_SPACE)
```


```python
# client.list_pipelines()
```


```python
# print(NAME_SPACE)
# client.list_experiments(namespace=NAME_SPACE)
client.set_user_namespace(NAME_SPACE)
print(client.get_user_namespace())
```

    kubeflow-kindfor



```python
exp = client.create_experiment(EXPERIMENT_NAME, description=EXPERIMENT_DESC)
```


<a href="/pipeline/#/experiments/details/c72bc5aa-4c35-4b18-aaa7-ff459d893795" target="_blank" >Experiment details</a>.



```python
# Specify pipeline argument values
arguments = {'a': '7', 'b': '8'}

# added a default pod transformer to all the pipeline ops
pipeline_config: dsl.PipelineConf = dsl.PipelineConf()

#pipeline_config.add_op_transformer(
#    pod_defaults_transformer
#)

client.create_run_from_pipeline_func(pipeline_func=calc_pipeline, arguments=arguments,
                                     experiment_name=EXPERIMENT_NAME, namespace=NAME_SPACE,
                                     pipeline_conf=pipeline_config)
# The generated links below lead to the Experiment page and the pipeline run details page, respectively
```


<a href="/pipeline/#/experiments/details/c72bc5aa-4c35-4b18-aaa7-ff459d893795" target="_blank" >Experiment details</a>.



<a href="/pipeline/#/runs/details/33f61e72-c1d2-45de-8e4a-2bbf39843af0" target="_blank" >Run details</a>.





    RunPipelineResult(run_id=33f61e72-c1d2-45de-8e4a-2bbf39843af0)




```python

```
