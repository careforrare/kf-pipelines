## About this Notebook
@author: Yingding Wang

This notebook shows the approach to read from a feast online features from a postgres db.


```python
import sys
!{sys.executable} -m pip install --user --upgrade feast feast-postgres==0.2.4
```


```python
from pprint import pprint
from feast import FeatureStore
home_dir="/home/jovyan"
fs_repo_name="fsrepo"
fs_repo_path=f"{home_dir}/{fs_repo_name}"
print(f"{fs_repo_path}")

store = FeatureStore(repo_path=fs_repo_path)
```


```python
feature_vector = store.get_online_features(
    features=[
    'proteome_sample_observations:OlinkID',
    'proteome_sample_observations:UniPort',
    'proteome_sample_observations:Assay'
    ],
    entity_rows=[{"SampleID": "COV041_20_47"}]
).to_dict()
```

```python
feature_vector = store.get_online_features(
    features=[
    'proteome_olink_observations:OlinkID'
    'proteome_olink_observations:UniPort'
    'proteome_olink_observations:Assay'
    ],
    entity_rows=[{}]
).to_dict()
```


```python
pprint(feature_vector)
```


```python
print(type(feature_vector))
```


```python
print(len(feature_vector))
```


```python

```
