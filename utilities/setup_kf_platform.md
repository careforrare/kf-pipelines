# About this notebook

@author: Yingding Wang

This notebook shall be run to set the default kubeflow notebook


```python
import sys
print(f"Sys version: {sys.version}")
```

    Sys version: 3.8.10 | packaged by conda-forge | (default, May 11 2021, 07:01:05) 
    [GCC 9.3.0]


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
!{sys.executable} -m pip install --upgrade --user jupyterlab==3.2.5
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

    Name: jupyterlab
    Version: 3.2.5
    Summary: JupyterLab computational environment
    Home-page: https://jupyter.org
    Author: Jupyter Development Team
    Author-email: jupyter@googlegroups.com
    License: UNKNOWN
    Location: /home/jovyan/.local/lib/python3.8/site-packages
    Requires: jupyter-core, tornado, jupyterlab-server, packaging, jinja2, nbclassic, jupyter-server, ipython
    Required-by: 


# Install packages in Jupter notebook

https://jakevdp.github.io/blog/2017/12/05/installing-python-packages-from-jupyter/


```python
# Install a pip package in the current Jupyter Kernel
# import sys
!{sys.executable} -m pip install --user numpy tensorflow panda
```

    Requirement already satisfied: numpy in /opt/conda/lib/python3.8/site-packages (1.21.2)
    Collecting tensorflow
      Using cached tensorflow-2.7.0-cp38-cp38-manylinux2010_x86_64.whl (489.6 MB)
    Collecting panda
      Using cached panda-0.3.1-py3-none-any.whl
    Collecting libclang>=9.0.1
      Using cached libclang-12.0.0-py2.py3-none-manylinux1_x86_64.whl (13.4 MB)
    Collecting tensorflow-io-gcs-filesystem>=0.21.0
      Downloading tensorflow_io_gcs_filesystem-0.23.0-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (2.1 MB)
    [K     |████████████████████████████████| 2.1 MB 5.0 MB/s eta 0:00:01
    [?25hRequirement already satisfied: termcolor>=1.1.0 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (1.1.0)
    Collecting keras<2.8,>=2.7.0rc0
      Using cached keras-2.7.0-py2.py3-none-any.whl (1.3 MB)
    Requirement already satisfied: six>=1.12.0 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (1.16.0)
    Collecting grpcio<2.0,>=1.24.3
      Using cached grpcio-1.42.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (4.0 MB)
    Collecting tensorboard~=2.6
      Using cached tensorboard-2.7.0-py3-none-any.whl (5.8 MB)
    Collecting keras-preprocessing>=1.1.1
      Using cached Keras_Preprocessing-1.1.2-py2.py3-none-any.whl (42 kB)
    Requirement already satisfied: protobuf>=3.9.2 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (3.17.3)
    Collecting astunparse>=1.6.0
      Using cached astunparse-1.6.3-py2.py3-none-any.whl (12 kB)
    Collecting opt-einsum>=2.3.2
      Using cached opt_einsum-3.3.0-py3-none-any.whl (65 kB)
    Requirement already satisfied: typing-extensions>=3.6.6 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (3.10.0.2)
    Collecting gast<0.5.0,>=0.2.1
      Using cached gast-0.4.0-py3-none-any.whl (9.8 kB)
    Collecting flatbuffers<3.0,>=1.12
      Using cached flatbuffers-2.0-py2.py3-none-any.whl (26 kB)
    Collecting tensorflow-estimator<2.8,~=2.7.0rc0
      Using cached tensorflow_estimator-2.7.0-py2.py3-none-any.whl (463 kB)
    Requirement already satisfied: h5py>=2.9.0 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (3.2.1)
    Requirement already satisfied: wrapt>=1.11.0 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (1.13.1)
    Requirement already satisfied: absl-py>=0.4.0 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (0.11.0)
    Requirement already satisfied: wheel<1.0,>=0.32.0 in /opt/conda/lib/python3.8/site-packages (from tensorflow) (0.36.2)
    Collecting google-pasta>=0.1.1
      Using cached google_pasta-0.2.0-py3-none-any.whl (57 kB)
    Collecting markdown>=2.6.8
      Using cached Markdown-3.3.6-py3-none-any.whl (97 kB)
    Collecting werkzeug>=0.11.15
      Using cached Werkzeug-2.0.2-py3-none-any.whl (288 kB)
    Collecting tensorboard-data-server<0.7.0,>=0.6.0
      Using cached tensorboard_data_server-0.6.1-py3-none-manylinux2010_x86_64.whl (4.9 MB)
    Requirement already satisfied: setuptools>=41.0.0 in /opt/conda/lib/python3.8/site-packages (from tensorboard~=2.6->tensorflow) (49.6.0.post20210108)
    Collecting tensorboard-plugin-wit>=1.6.0
      Using cached tensorboard_plugin_wit-1.8.0-py3-none-any.whl (781 kB)
    Requirement already satisfied: requests<3,>=2.21.0 in /opt/conda/lib/python3.8/site-packages (from tensorboard~=2.6->tensorflow) (2.25.1)
    Requirement already satisfied: google-auth<3,>=1.6.3 in /opt/conda/lib/python3.8/site-packages (from tensorboard~=2.6->tensorflow) (1.35.0)
    Collecting google-auth-oauthlib<0.5,>=0.4.1
      Using cached google_auth_oauthlib-0.4.6-py2.py3-none-any.whl (18 kB)
    Requirement already satisfied: rsa<5,>=3.1.4 in /opt/conda/lib/python3.8/site-packages (from google-auth<3,>=1.6.3->tensorboard~=2.6->tensorflow) (4.7.2)
    Requirement already satisfied: cachetools<5.0,>=2.0.0 in /opt/conda/lib/python3.8/site-packages (from google-auth<3,>=1.6.3->tensorboard~=2.6->tensorflow) (4.2.4)
    Requirement already satisfied: pyasn1-modules>=0.2.1 in /opt/conda/lib/python3.8/site-packages (from google-auth<3,>=1.6.3->tensorboard~=2.6->tensorflow) (0.2.8)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in /opt/conda/lib/python3.8/site-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard~=2.6->tensorflow) (1.3.0)
    Collecting importlib-metadata>=4.4
      Using cached importlib_metadata-4.8.2-py3-none-any.whl (17 kB)
    Collecting zipp>=0.5
      Using cached zipp-3.6.0-py3-none-any.whl (5.3 kB)
    Requirement already satisfied: pyasn1<0.5.0,>=0.4.6 in /opt/conda/lib/python3.8/site-packages (from pyasn1-modules>=0.2.1->google-auth<3,>=1.6.3->tensorboard~=2.6->tensorflow) (0.4.8)
    Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/lib/python3.8/site-packages (from requests<3,>=2.21.0->tensorboard~=2.6->tensorflow) (2021.10.8)
    Requirement already satisfied: chardet<5,>=3.0.2 in /opt/conda/lib/python3.8/site-packages (from requests<3,>=2.21.0->tensorboard~=2.6->tensorflow) (4.0.0)
    Requirement already satisfied: idna<3,>=2.5 in /opt/conda/lib/python3.8/site-packages (from requests<3,>=2.21.0->tensorboard~=2.6->tensorflow) (2.10)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /opt/conda/lib/python3.8/site-packages (from requests<3,>=2.21.0->tensorboard~=2.6->tensorflow) (1.26.5)
    Requirement already satisfied: oauthlib>=3.0.0 in /opt/conda/lib/python3.8/site-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard~=2.6->tensorflow) (3.1.1)
    Installing collected packages: zipp, importlib-metadata, werkzeug, tensorboard-plugin-wit, tensorboard-data-server, markdown, grpcio, google-auth-oauthlib, tensorflow-io-gcs-filesystem, tensorflow-estimator, tensorboard, opt-einsum, libclang, keras-preprocessing, keras, google-pasta, gast, flatbuffers, astunparse, tensorflow, panda
    [33m  WARNING: The script markdown_py is installed in '/home/jovyan/.local/bin' which is not on PATH.
      Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.[0m
    [33m  WARNING: The script google-oauthlib-tool is installed in '/home/jovyan/.local/bin' which is not on PATH.
      Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.[0m
    [33m  WARNING: The script tensorboard is installed in '/home/jovyan/.local/bin' which is not on PATH.
      Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.[0m
    [33m  WARNING: The scripts estimator_ckpt_converter, import_pb_to_tensorboard, saved_model_cli, tensorboard, tf_upgrade_v2, tflite_convert, toco and toco_from_protos are installed in '/home/jovyan/.local/bin' which is not on PATH.
      Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.[0m
    Successfully installed astunparse-1.6.3 flatbuffers-2.0 gast-0.4.0 google-auth-oauthlib-0.4.6 google-pasta-0.2.0 grpcio-1.42.0 importlib-metadata-4.8.2 keras-2.7.0 keras-preprocessing-1.1.2 libclang-12.0.0 markdown-3.3.6 opt-einsum-3.3.0 panda-0.3.1 tensorboard-2.7.0 tensorboard-data-server-0.6.1 tensorboard-plugin-wit-1.8.0 tensorflow-2.7.0 tensorflow-estimator-2.7.0 tensorflow-io-gcs-filesystem-0.23.0 werkzeug-2.0.2 zipp-3.6.0


# Running TensorFlow code


```python
import tensorflow as tf
print("TensorFlow version:", tf.__version__)
```

    TensorFlow version: 2.7.0



```python


```
