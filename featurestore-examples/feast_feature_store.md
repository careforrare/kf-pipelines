## About this notebook
@author: Yingding Wang

This notebook shows the approach to save and load a feaure data set with google feast connected to an online postgres db.
* feast: https://feast.dev/
* feast-postgres: https://github.com/nossrannug/feast-postgres
* postgres on dockerhub: https://hub.docker.com/_/postgres



```python
import sys, os
```


```python
# %%capture 
# discard the pip output

# Fixed feast and feast-postgres version, since this works and feel free to change it for the future release
!{sys.executable} -m pip install feast==0.17.0 feast-postgres==0.2.4 python-dotenv pandas
```

## (optional) create an .env file 

Uncomment the following cell to create an .env file, change the param values to meet your settings before running the cell

Note:
Uncomment the block comment and the `%%writefile .env` should be the first in the line.


```python
'''
%%writefile .env
# environment variables for online feature_store
ON_FS_HOST="POSTGRES_HOST_NAME"
ON_FS_DB="featurestore"
ON_FS_PORT="5432"
ON_FS_USER="postgres_name"
ON_FS_PW="postgres_pw"
'''
```


```python
from dotenv import load_dotenv
# load all values paar from ./.env into environment variables
load_dotenv(override=True)

'''
print(f"\
{os.environ['ON_FS_HOST']}\n\
{os.environ['ON_FS_DB']}\n\
{os.environ['ON_FS_PORT']}\n\
{os.environ['ON_FS_USER']}\n\
{os.environ['ON_FS_PW']}\n\
")
'''
```

## Init a feature store local repo


```python
home_dir="/home/jovyan"
feature_repo_name="fsrepo"
feature_repo_path=f"{home_dir}/{feature_repo_name}"
print(f"{feature_repo_path}")

config_file=f"{feature_repo_path}/feature_store.yaml"
# print(f"{config_file}")
```

    /home/jovyan/fsrepo



```python
# use bash cd to change the directory, since feast -c doesn't work in version 0.15.1
if os.path.exists(feature_repo_path) and os.path.isdir(feature_repo_path):
    print(f"feast repository already exists: {feature_repo_path}")
else:
    !cd $home_dir && feast init $feature_repo_name

# print(f"{os.getcwd()}")
```

    
    Creating a new Feast repository in [1m[32m/home/jovyan/fsrepo[0m.
    


## Update the feature store configuration file
* create custom "%%writetemplate" magic command: https://stackoverflow.com/questions/26385041/is-it-possible-to-write-the-value-of-a-variable-in-a-writefile-magic-command-i/63784887#63784887
* use filename variable in "%%writefile" magic command: https://github.com/ipython/ipython/issues/6701#issue-45873574


```python
from IPython.core.magic import register_line_cell_magic

# create a custom template magic command %%writetemplate
# https://github.com/ipython/ipython/issues/6701#issuecomment-382640776
@register_line_cell_magic
def writetemplate(line, cell):
    with open(line, 'w') as f:
        f.write(cell.format(**globals()))
        
        
# all need to assign global variables from environment variables
ON_FS_HOST=os.environ['ON_FS_HOST']
ON_FS_PORT=os.environ['ON_FS_PORT']
ON_FS_DB=os.environ['ON_FS_DB']
ON_FS_USER=os.environ['ON_FS_USER']
ON_FS_PW=os.environ['ON_FS_PW']
```


```python
%%writetemplate $config_file
project: feature_repo
registry: data/registry.db
provider: local
online_store:
    type: feast_postgres.PostgreSQLOnlineStore # MUST be this value
    host: {ON_FS_HOST}
    port: {ON_FS_PORT}       # Optional, default is 5432
    database: {ON_FS_DB}     # postgres is the default postgres db
    db_schema: feature_store # Optional, default is None, feature_store schema will be created and please enable the view of this new schema in your DB GUI Tools       
    user: {ON_FS_USER}
    password: {ON_FS_PW}

#registry:
#    registry_store_type: feast_postgres.PostgreSQLRegistryStore
#    path: feast_registry    # This will become the table name for the registry
#    host: {ON_FS_HOST}
#    port: {ON_FS_PORT}      # Optional, default is 5432
#    database: {ON_FS_DB}
#    db_schema: registry_feature_store
#    user: {ON_FS_USER}
#    password: {ON_FS_PW} 
```

Examples of change the working directory in Jupyter Notebook, which is not needed in this example:

* get current work directory: `CUR_DIR=os.getcwd()`
* Default work directory: `WORK_DIR="/home/jovyan/"`
* change the current work directory: `os.chdir(feature_repo)`


```python
# apply the features, and the notebook cell work directory remain unchanged
!cd $feature_repo_path && feast apply
```

    Created entity [1m[32mdriver_id[0m
    Created feature view [1m[32mdriver_hourly_stats[0m



```python
!cd $feature_repo_path && feast materialize-incremental $(date -u +"%Y-%m-%dT%H:%M:%S")
```

    Materializing [1m[32m1[0m feature views to [1m[32m2022-01-11 11:58:56+00:00[0m into the [1m[32mfeast_postgres.PostgreSQLOnlineStore[0m online store.
    
    [1m[32mdriver_hourly_stats[0m from [1m[32m2022-01-10 11:58:56+00:00[0m to [1m[32m2022-01-11 11:58:56+00:00[0m:
    15it [00:00, 848.45it/s]                                                                            


## deploy feature to online feature store

https://www.mikulskibartosz.name/adding-datasets-to-feast-feature-store/


```python
proteome_olink_data_path="/home/jovyan/data/Proteome_Olink_data.csv"
```


```python
import pandas
from datetime import datetime, timezone
df = pandas.read_csv(proteome_olink_data_path)
# df.reset_index(level=0, inplace=True) # turn  the index into a columne

# datetime(year, month, day, hour, min, sec).timestamp() returns utc timestamp in secs as float, cast it to int()
# ts = datetime(2021, 11, 22, 20, 0, 0).replace(tzinfo=timezone.utc).timestamp()
# ts_rounded = int(ts)

# must be a datetime and can not be int timestamp
df['observation_dt'] = datetime(2021, 11, 22, 20, 30, 0).replace(tzinfo=timezone.utc)
df.head(1000)
```


```python
df.describe()
```


```python
# save the dataframe to local parquet file
print(f"{feature_repo_path}")
df.to_parquet(f"{feature_repo_path}/data/proteome_olink.parquet")
```

    /home/jovyan/fsrepo


## Define new features in Feast repository

The code flow does three things:

* It defines the feature source location. In this case, a path to the local file system. Note that the FileSource also requires the column containing the event timestamp.
* The Entity object describes which column contains the entity identifier. In our example, the value is useless and has no business meaning, but we still need it.
* Finally, we define the FeatureView, which combines the available column names (and types) with the entity identifier and the data location. We have only historical data in our example, so I set the online parameter to False.

Note:\
Since Feast 0.11, we can skip the features parameter in FeatureView, and the library will infer the column names and types from the data.

Reference:
* Feature View feast 0.17: https://docs.feast.dev/getting-started/concepts/feature-view


```python
%%writefile $feature_repo_path/proteome_sample.py
# this file defines the proteome_oline_data.csv
from datetime import timedelta
from google.protobuf.duration_pb2 import Duration
from feast import Entity, Feature, FeatureView, ValueType, FileSource
#from feast.data_source import FileSource

proteome_sample_observations = FileSource(
    # NOTICE: need to change this path to fit the feast repository print(f"{feature_repo_path}") 
    path="/home/jovyan/fsrepo/data/proteome_olink.parquet",
    event_timestamp_column="observation_dt",
)

proteome_sample = Entity(name="SampleID", value_type=ValueType.STRING, description="Sample Identifier",)

proteome_sample_observations_view = FeatureView(
    name="proteome_sample_observations",
    entities=["SampleID"],
    ttl=timedelta(days=-1),
    features=[
         Feature(name="OlinkID", dtype=ValueType.STRING),
         Feature(name="UniPort", dtype=ValueType.STRING),
         Feature(name="Assay", dtype=ValueType.STRING),
#        Feature(name="UniPort", dtype=ValueType.STRING),
#        Feature(name="sepal_width", dtype=ValueType.FLOAT),
#        Feature(name="petal_length", dtype=ValueType.INT64),
#        Feature(name="petal_width", dtype=ValueType.INT64),
#        Feature(name="species", dtype=ValueType.STRING),
    ],
    online=True,
#    input=proteome_sample_observations,
    batch_source=proteome_sample_observations,
    tags={},
)
```


```python
# reload feature repository
import feast_postgres
!cd $feature_repo_path && feast apply
```

    Unchanged entity [1m[94mSampleID[0m
    Unchanged entity [1m[94mdriver_id[0m
    Unchanged feature view [1m[94mdriver_hourly_stats[0m
    Unchanged feature view [1m[94mproteome_sample_observations[0m
    Deploying infrastructure for [1m[32mdriver_hourly_stats[0m
    Deploying infrastructure for [1m[32mproteome_sample_observations[0m



```python
!cd $feature_repo_path && feast version
```

    Feast SDK Version: "feast 0.17.0"


## Populate data to online store
https://aws.amazon.com/blogs/opensource/getting-started-with-feast-an-open-source-feature-store-running-on-aws-managed-services/


```python
# populate feature value to online store, incremental
!cd $feature_repo_path && feast materialize-incremental $(date -u +"%Y-%m-%dT%H:%M:%S")
```

    Materializing [1m[32m2[0m feature views to [1m[32m2022-01-11 12:01:15+00:00[0m into the [1m[32mfeast_postgres.PostgreSQLOnlineStore[0m online store.
    
    [1m[32mdriver_hourly_stats[0m from [1m[32m2022-01-11 11:58:56+00:00[0m to [1m[32m2022-01-11 12:01:15+00:00[0m:
    0it [00:00, ?it/s]
    [1m[32mproteome_sample_observations[0m from [1m[32m2022-01-12 12:01:16+00:00[0m to [1m[32m2022-01-11 12:01:15+00:00[0m:
    Traceback (most recent call last):
      File "/opt/conda/bin/feast", line 8, in <module>
        sys.exit(cli())
      File "/opt/conda/lib/python3.8/site-packages/click/core.py", line 829, in __call__
        return self.main(*args, **kwargs)
      File "/opt/conda/lib/python3.8/site-packages/click/core.py", line 782, in main
        rv = self.invoke(ctx)
      File "/opt/conda/lib/python3.8/site-packages/click/core.py", line 1259, in invoke
        return _process_result(sub_ctx.command.invoke(sub_ctx))
      File "/opt/conda/lib/python3.8/site-packages/click/core.py", line 1066, in invoke
        return ctx.invoke(self.callback, **ctx.params)
      File "/opt/conda/lib/python3.8/site-packages/click/core.py", line 610, in invoke
        return callback(*args, **kwargs)
      File "/opt/conda/lib/python3.8/site-packages/click/decorators.py", line 21, in new_func
        return f(get_current_context(), *args, **kwargs)
      File "/opt/conda/lib/python3.8/site-packages/feast/cli.py", line 467, in materialize_incremental_command
        store.materialize_incremental(
      File "/opt/conda/lib/python3.8/site-packages/feast/usage.py", line 269, in wrapper
        return func(*args, **kwargs)
      File "/opt/conda/lib/python3.8/site-packages/feast/feature_store.py", line 877, in materialize_incremental
        provider.materialize_single_feature_view(
      File "/opt/conda/lib/python3.8/site-packages/feast/infra/passthrough_provider.py", line 142, in materialize_single_feature_view
        table = offline_job.to_arrow()
      File "/opt/conda/lib/python3.8/site-packages/feast/infra/offline_stores/offline_store.py", line 67, in to_arrow
        return self._to_arrow_internal()
      File "/opt/conda/lib/python3.8/site-packages/feast/usage.py", line 280, in wrapper
        raise exc.with_traceback(traceback)
      File "/opt/conda/lib/python3.8/site-packages/feast/usage.py", line 269, in wrapper
        return func(*args, **kwargs)
      File "/opt/conda/lib/python3.8/site-packages/feast/infra/offline_stores/file.py", line 66, in _to_arrow_internal
        df = self.evaluation_function()
      File "/opt/conda/lib/python3.8/site-packages/feast/infra/offline_stores/file.py", line 334, in evaluate_offline_job
        return last_values_df[columns_to_extract]
      File "/opt/conda/lib/python3.8/site-packages/pandas/core/frame.py", line 3030, in __getitem__
        indexer = self.loc._get_listlike_indexer(key, axis=1, raise_missing=True)[1]
      File "/opt/conda/lib/python3.8/site-packages/pandas/core/indexing.py", line 1266, in _get_listlike_indexer
        self._validate_read_indexer(keyarr, indexer, axis, raise_missing=raise_missing)
      File "/opt/conda/lib/python3.8/site-packages/pandas/core/indexing.py", line 1316, in _validate_read_indexer
        raise KeyError(f"{not_found} not in index")
    KeyError: "['UniPort'] not in index"



```python
# optional: call feature materialize from a past UTC datetime to ingest all the data
!cd $feature_repo_path && feast materialize "2010-10-10T00:00:00" $(date -u +"%Y-%m-%dT%H:%M:%S")
```

## Advanced feature store with TTL

**What does TTL mean?**\
In the example below, we retrieve the value from the feature store. We must specify the event_timestamp. The ttl describes the maximal time difference between the actual event timestamp and the timestamp we want to get. Of course, it is a difference “in the past.” We can never retrieve events “in the future.”


```python

```
