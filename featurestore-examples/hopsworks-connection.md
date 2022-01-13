# About this notebook

@author: Yingding Wang

This notebook demonstrate an example to connect to hopsworks featurestore from a kubeflow notebook

## Prerequisites
* Make sure a API Key is created in hopsworks project and feature store, project and job are allowed.
* Read the following step carefully to get `hsfs` from pip installed into your conda env

Reference: 
* Create user API keys in Hopsworks Feature Store: https://hopsworks.readthedocs.io/en/2.0/featurestore/integrations/guides/custom.html
* Integrate Feature Store in KubeFlow Notebook: https://docs.hopsworks.ai/feature-store-api/latest/integrations/python/

## Install pip package inside conda venv

#### 1. In the console create a conda venv names hops
```console
conda create -y -n hops 
conda activate hops
conda install -y gcc=11.2.0
conda install -y pip
pip install --no-cache-dir --upgrade hsfs[hive]==2.4.7
conda deactivate
```
<!-- 
# python -m ipykernel install --user --name=hops
# ipython kernel install --user --name=hops
# python -m ipykernel install --user --name=hops
# conda env remove --name hops
-->

#### 2. Make conda env in JupyterLab available
```
conda activate hops
conda install -y ipykernel
ipython kernel install --user --name=hops
conda deactivate
```
Reference:
* https://stackoverflow.com/questions/53004311/how-to-add-conda-environment-to-jupyter-lab/53546634#53546634

**WARNING:**

In the KubeFlow Notebook, after every time Notebook Server restart, the path `/opt/conda/envs` will be cleared. You notebook venv kernel will not work after Notebook Server restart.

#### 3. Restart notebooks with new kernel hops

* You may need to stop the kernel and close the notebook and reopen it, until the new kernel with name `hops` is visible
* Select kernel `hops` and restart kernel

#### 4. (optional) List you env in conda:
```
conda env list
```

#### 5. (optional) Remove a virtual
```console
jupyter kernelspec uninstall -y <VENV_NAME>
conda env remove --name <VENV_NAME>
```
Reference:
* https://stackoverflow.com/questions/41060382/using-pip-to-install-packages-to-anaconda-environment/43729857#43729857



```python
import sys
print(f"Sys version: {sys.version}")
```

    Sys version: 3.10.1 | packaged by conda-forge | (main, Dec 22 2021, 01:39:36) [GCC 9.4.0]



```python
# This kernel runs with pip
# !{sys.executable} -m pip install hsfs[hive]==2.4.7
!{sys.executable} -m pip install python-dotenv
```

    Collecting python-dotenv
      Using cached python_dotenv-0.19.2-py2.py3-none-any.whl (17 kB)
    Installing collected packages: python-dotenv
    Successfully installed python-dotenv-0.19.2


### (Optional) Uncomment the following cell to create a key file for featurestore

Replace the following line with your real API key

Notice: This method doesn't always works, it doesn't noticed that it is a local installation and will call a aws region sometimes. Wth API_KEY_VALUE works.


```python
#%%writefile featurestore.key
#xxxxxxxxxxx
```

### (Optional) Adding Env Variables for feature store

copy the following cell template and uncomment the content and edit with your credentials


```python
#%%writefile fs.env
## environment variables for online feature_store
#ON_FS_HOST="DNS_host_name"
#ON_FS_PROJ="proj_name"
#ON_FS_PORT="443"
#ON_FS_APIKEY="You_API_KEY_VALUE"
```


```python
# Load env
import os
from dotenv import load_dotenv
load_dotenv(dotenv_path="fs.env", override=True)
```




    True




```python
#print(f"{os.environ['ON_FS_HOST']}")
#print(f"{os.environ['ON_FS_PROJ']}")
#print(f"{os.environ['ON_FS_PORT']}")
#print(f"{os.environ['ON_FS_APIKEY']}")
```


```python
import hsfs
```


```python
conn = hsfs.connection(
    host=str(os.environ['ON_FS_HOST']),    # DNS of your Feature Store instance
    port=int(os.environ['ON_FS_PORT']),    # Port to reach your Hopsworks instance, defaults to 443
    project=str(os.environ['ON_FS_PROJ']), # Name of your Hopsworks Feature Store project
    api_key_value=str(os.environ['ON_FS_APIKEY']),   # API Key Value String
    hostname_verification=False            # Disable for self-signed certificates
)
fs = conn.get_feature_store()              # Get the project's default feature store"
```

    Connected. Call `.close()` to terminate connection gracefully.



```python

```


```python
conn.close()
```

    Connection closed.



```python

```
