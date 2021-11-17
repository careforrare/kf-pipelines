# About this Jupyter Notebook


```python
import sys
print(f"Sys version: {sys.version}")
```

# Install kfp to build a pipeline
Reference: https://www.kubeflow.org/docs/components/pipelines/sdk/build-pipeline/


```python
import sys
!{sys.executable} -m pip install kfp --upgrade
```

# Restart the kernal
After update the kfp, restart this notebook kernel

Jupyter notebook: Meun -> Kernel -> restart kernel

## Setup
Example Pipeline from

https://github.com/kubeflow/examples/tree/master/pipelines/simple-notebook-pipeline

## Getting started with Python function-based components

https://www.kubeflow.org/docs/components/pipelines/sdk/python-function-components/


```python
from platform import python_version

EXPERIMENT_NAME = 'core kf test'        # Name of the experiment in the UI
EXPERIMENT_DESC = 'testing KF platform'
BASE_IMAGE = f"library/python:{python_version()}" # Base image used for components in the pipeline
NAME_SPACE = 'kubeflow-kindfor'
```


```python
import kfp
import kfp.dsl as dsl
import kfp.compiler as compiler
import kfp.components as components
```


```python
def add(a: float, b: float) -> float:
    '''Calculates sum of two arguments'''
    print(a, '+', b, '=', a + b)
    return a + b
```

### Create component from function

https://kubeflow-pipelines.readthedocs.io/en/latest/source/kfp.components.html



```python
add_op = components.create_component_from_func(
    add,
    base_image=BASE_IMAGE,
    output_component_file='add_component.yaml' 
)
```

### Add pod memory and cpu restriction

https://github.com/kubeflow/pipelines/pull/5695


```python
# run command line to see the quota
!kubectl describe quota
```


```python
def pod_defaults(op):
    op.set_memory_request('100Mi');# op.set_memory_limit('1000Mi')
    op.set_cpu_request('100m'); # 1 core, # op.set_cpu_limit('1000m')
    return op
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
    first_add_task = pod_defaults(add_op(a, 4))
    # second_add_task = add_op(first_add_task.output, b)
    second_add_task = pod_defaults(add_op(first_add_task.output, b))
```

# Multi-user Isolation for Pipelines

https://www.kubeflow.org/docs/components/pipelines/multi-user/


```python
# get the pipeline host from env set up be the notebook instance
client = kfp.Client()
```


```python
exp = client.create_experiment(EXPERIMENT_NAME, description=EXPERIMENT_DESC, namespace=NAME_SPACE )
```


```python
# Specify pipeline argument values
arguments = {'a': '7', 'b': '8'}

client.create_run_from_pipeline_func(calc_pipeline, arguments=arguments, experiment_name=exp.name)
# The generated links below lead to the Experiment page and the pipeline run details page, respectively
```


```python

```
