## About this notebook
@author: Yingding Wang

This notebook shows the approach to use scp client in a Jupyter Notebook to copy data folder from an remote server.



```python
import sys
!{sys.executable} -m pip install paramiko scp python-dotenv
```

## (optional) create an .env file 

Uncomment the following cell to create an .env file, change the user and password for your ssh connection before running the cell


```python
#%%writefile .env
## environment variables for ssh
#SSH_HOST="remote_host_name"
#SSH_USER="user_name"
#SSH_PW="pass_word"
```


```python
from paramiko import SSHClient, AutoAddPolicy
from scp import SCPClient
from dotenv import load_dotenv
import os

load_dotenv()
```


```python
# debug .env file with
#print(os.environ["SSH_HOST"])
#print(os.environ["SSH_USER"])
#print(os.environ["SSH_PW"])
```

### Add HostKey without varification in a private network
* issue with unkown host key: https://stackoverflow.com/questions/53635843/paramiko-ssh-failing-with-server-not-found-in-known-hosts-when-run-on-we/53645945#53645945
* Example of copy file recursive: https://stackoverflow.com/questions/52954532/how-to-copy-a-complete-directory-recursively-to-a-remote-server-in-python-param
* Example of file copy: https://pypi.org/project/scp/
* SCP get code: https://github.com/jbardin/scp.py/blob/master/scp.py#L148


```python
# data folder on an remote scp host
SOURCE_DIR="/home/bioinformatic/data"
# save the source_dir directory to the target_dir (local parent directory)
TARGET_DIR="/home/jovyan"

with SSHClient() as ssh:
    ssh.set_missing_host_key_policy(AutoAddPolicy())
    ssh.connect(os.environ["SSH_HOST"], port=22, username=os.environ["SSH_USER"], password=os.environ["SSH_PW"])
    
    with SCPClient(ssh.get_transport()) as scp:
        scp.get(SOURCE_DIR, local_path=TARGET_DIR, recursive=True, preserve_times=True)
```


```python

```
