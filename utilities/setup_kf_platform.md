# About this notebook

@author: Yingding Wang

This notebook shall be run to set the default kubeflow notebook


```python
import sys
print(f"Sys version: {sys.version}")
```

## Useful JupyterLab Basic

Before start, you may consider to update the jupyterlab with the command

<code>python
!{sys.executable} -m pip install --upgrade --user jupyterlab    
</code>  

1. Autocomplete syntax with "Tab"
2. View Doc String with "Shift + Tab"
3. mark the code snippet -> select with right mouse -> Show Contextual Help (see the function code)


```python
# upgrade jupyter lab
!{sys.executable} -m pip install --upgrade --user jupyterlab==3.2.7
```

## Restart this notebook server
* Stop kernel
* close Notebook
* close JupyterLab Tab
* Stop the JupyterLab Server in KubeFlow Notebook
* Start the JupyterLab Server in KubeFlow Notebook


```python
# Know you shall see the Jupyter Lab is updated
!{sys.executable} -m pip show jupyterlab # 3.0.16
```

# Install packages in Jupter notebook

https://jakevdp.github.io/blog/2017/12/05/installing-python-packages-from-jupyter/


```python
# Install a pip package in the current Jupyter Kernel
# import sys
!{sys.executable} -m pip install --user numpy tensorflow panda
```

# Running TensorFlow code


```python
import tensorflow as tf
print("TensorFlow version:", tf.__version__)
```


```python


```
