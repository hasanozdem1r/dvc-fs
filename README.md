# DVC filesystem abstraction layer

**Note:**
This is an old archive of the original repository, before the code was restructured.

**[→ Please click here to navigate to the current maintained version ←](https://github.com/styczynski/dvc-fs)**


# DVC filesystem abstraction layer (0.8.2)

[![PyPI version](https://badge.fury.io/py/dvc-fs.svg)](https://badge.fury.io/py/dvc-fs) [![Build and test](https://github.com/covid-genomics/dvc-fs/actions/workflows/build_and_test.yml/badge.svg)](https://github.com/covid-genomics/dvc-fs/actions/workflows/build_and_test.yml) [![Lint code](https://github.com/covid-genomics/dvc-fs/actions/workflows/lint.yml/badge.svg)](https://github.com/covid-genomics/dvc-fs/actions/workflows/lint.yml)

This package provides high-level API work easy writing/reading/listing files inside the DVC.
It can be used for automation systems integrated with data pipelines.

dvc-fs provides basic compatibility (Still work in progress) with [PyFilesystem2 API](https://github.com/PyFilesystem/pyfilesystem2).

## :floppy_disk: Installation

To install this package please do:
```bash
  $ python3 -m pip install "dvc-fs==0.8.2"
```
Or with Poetry:
```bash
  $ poetry install dvc-fs
```

### :question: Usage

**Using via PyFielsystem2:**

The dvc-fs package is integrated with [PyFilesystem](https://github.com/PyFilesystem/pyfilesystem2), so you can do:

```python
from fs import open_fs
fs1 = open_fs("dvc://github.com/covid-genomics/data-artifacts") # Clone by https
fs2 = open_fs("dvc://ssh@github.com/covid-genomics/data-artifacts") # Clone by ssh
fs3 = open_fs("dvc://<PAT>@github.com/covid-genomics/data-artifacts") # Clone by https with personal access token
 # You can also use normal HTTPS and create env variable GIT_TOKEN
 # In that case Personal Access Token will be injected in the clone url
```

And now the usage is as follows:
```python
from fs import open_fs
with open_fs("dvc://github.com/covid-genomics/data-artifacts") as fs:
    fs.writetext("fs_test/fasta2.txt", "TEST")
```

**Explicitly creating DVCFS:**

This method allows you to explicitly create DVCFS class in your applciation:

```python
from dvc_fs.fs import DVCFS
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    for path in fs.walk.files():
        # Print all paths in repo
        print(path)
```

# README FOR OUTDATED VERSION
## Basic features

**List all files with walk():**

```python
from dvc_fs.fs import DVCFS
with DVCFS(
    "https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git"
) as fs:
    for path in fs.walk.files():
        print(path)
```

**Check if the file exists:**

```python
from dvc_fs.fs import DVCFS
from os import environ
with DVCFS(
    dvc_repo=f"https://{environ['GIT_TOKEN']}@github.com/covid-genomics/data-artifacts.git"
) as fs:
    for path in ["cirk//BVic HIs_SH2021.xlsx"]:
         print(fs.exists(path=path))
```

**Removing files:**

```python
from dvc_fs.fs import DVCFS
with DVCFS(
    "https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/data-artifacts.git"
) as fs:
    fs.writetext("data/to_remove.txt", "TEST STRING ABCDEF 123456")
    fs.remove("data/to_remove.txt")
```

## Writing to the repository from various sources

Read and write contents:
```python
from dvc_fs.fs import DVCFS
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    contents = fs.readtext('data/1.txt')
    print(f"THIS IS CONTENTS: {contents}")
    fs.writetext("test.txt", contents+"!")
```

You can also directly use DVC high-level api via the Client:
```python
from dvc_fs.client import Client, DVCPathUpload
# Git repo with DVC configured
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    fs.bulk_update([
        # Upload local file ~/local_file_path.txt to DVC repo under path data/1.txt
        DVCPathUpload("data/1.txt", "~/local_file_path.txt"),
    ])
```

The upload operator supports various types of data inputs that you can feed into it.

**Uploading a string as a file:**
```python
from dvc_fs import DVCStringUpload, DVCPathUpload
from datetime import datetime
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    fs.bulk_update([
        DVCStringUpload("data/1.txt", f"This will be saved into DVC. Current time: {datetime.now()}"),
    ])
```

**Uploading local file using its path:**
```python
from dvc_fs import DVCPathUpload
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    fs.bulk_update([
        DVCPathUpload("data/1.txt", "~/local_file_path.txt"),
    ])
```

**Upload content generated by a python function:**
```python
from dvc_fs import DVCCallbackUpload
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    fs.bulk_update([
        DVCCallbackUpload("data/1.txt", lambda: "Test data"),
    ])
```

We can use `download` operation similarily to the `upload`. The syntax is the same:
```python
from dvc_fs import DVCCallbackDownload
# Download DVC file data/1.txt and print it on the screen
with DVCFS("https://<GITHUB_PERSONAL_TOKEN>@github.com/covid-genomics/dvc_repo.git") as fs:
    fs.bulk_update([
        DVCCallbackDownload("data/1.txt", lambda content: print(content)),
    ])
```

### Versioning

To bump project version before release please use the following command (for developers):
```bash
    $ poetry run bump2version minor
```
