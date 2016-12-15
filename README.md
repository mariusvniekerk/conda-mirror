# conda-mirror
[![Build Status](https://travis-ci.org/maxpoint/conda-mirror.svg?branch=master)](https://travis-ci.org/maxpoint/conda-mirror)
[![codecov](https://codecov.io/gh/maxpoint/conda-mirror/branch/master/graph/badge.svg)](https://codecov.io/gh/maxpoint/conda-mirror)


## Example Usage

WARNING: Invoking this command will pull ~10GB and take at least an hour
```bash
$ conda-mirror --upstream-channel conda-forge --target-directory local_mirror --platform linux-64
```

## More Details

### blacklist/whitelist configuration

example-conf.yaml:

```yaml
blacklist:
    - license: "*agpl*"
    - license: None
    - license: ""

whitelist:
    - name: system
```

`blacklist` removes package(s) that match the condition(s) listed from the
upstream repodata.

`whitelist` re-includes any package(s) from blacklist that match the
whitelist conditions.

blacklist and whitelist both take lists of dictionaries. The keys in the
dictionary need to be values in the repodata.json metadata. The values are
(unix) globs to match on. Go here for the full repodata of the upstream
"defaults" channel:
http://conda.anaconda.org/anaconda/linux-64/repodata.json

Here are the contents of one of the entries in repodata['packages']
```
{'botocore-1.4.10-py34_0.tar.bz2': {'arch': 'x86_64',
  'binstar': {'channel': 'main',
   'owner_id': '55fc8527d3234d09d4951c71',
   'package_id': '56b88ea1be1cc95a362b218e'},
  'build': 'py34_0',
  'build_number': 0,
  'date': '2016-04-11',
  'depends': ['docutils >=0.10',
   'jmespath >=0.7.1,<1.0.0',
   'python 3.4*',
   'python-dateutil >=2.1,<3.0.0'],
  'license': 'Apache',
  'md5': 'b35a5c1240ba672e0d9d1296141e383c',
  'name': 'botocore',
  'platform': 'linux',
  'requires': [],
  'size': 1831799,
  'version': '1.4.10'}}
```

See implementation details in the conda_mirror:match function for more 
information.

#### Common usage patterns
##### Mirror **only** one specific package
If you wanted to match exactly the botocore package listed above with your 
config, then you could use the following configuration to first blacklist 
**all** packages and then include just the botocore packages:
 
```yaml
blacklist:
    - name: "*"
whitelist:
    - name: botocore
      version: 1.4.10
      build: py34_0
```
##### Mirror everything but agpl licenses
```yaml
blacklist:
    - license: "*agpl*"
```

##### Mirror only python 3 packages
```yaml
blacklist:
    - name: "*"
whitelist:
    - build: "*py3*"
```


## CLI
```
$ conda-mirror -h
['/home/edill/miniconda/bin/conda-mirror', '-h']
usage: conda-mirror [-h] --upstream-channel UPSTREAM_CHANNEL
                    --target-directory TARGET_DIRECTORY --platform PLATFORM
                    [-v] [--config CONFIG] [--pdb]

CLI interface for conda-mirror.py

optional arguments:
  -h, --help            show this help message and exit
  --upstream-channel UPSTREAM_CHANNEL
                        The anaconda channel to mirror
  --target-directory TARGET_DIRECTORY
                        The place where packages should be mirrored to
  --platform PLATFORM   The OS platform(s) to mirror. one of: {'linux-64',
                        'linux-32','osx-64', 'win-32', 'win-64'}
  -v, --verbose         This basically turns on tqdm progress bars for
                        downloads
  --config CONFIG       Path to the yaml config file
  --pdb                 Enable PDB debugging on exception
```

## Testing

Note: Will install packages from pip

```
$ pip install -r test-requirements.txt
Requirement already satisfied: pytest in /home/edill/miniconda/lib/python3.5/site-packages (from -r test-requirements.txt (line 1))
Requirement already satisfied: coverage in /home/edill/miniconda/lib/python3.5/site-packages (from -r test-requirements.txt (line 2))
Requirement already satisfied: pytest-ordering in /home/edill/miniconda/lib/python3.5/site-packages (from -r test-requirements.txt (line 3))
Requirement already satisfied: py>=1.4.29 in /home/edill/miniconda/lib/python3.5/site-packages (from pytest->-r test-requirements.txt (line 1))

$ coverage run run_tests.py -x
sys.argv=['run_tests.py', '-x']
================================================================================== test session starts ===================================================================================
platform linux -- Python 3.5.2, pytest-3.0.4, py-1.4.31, pluggy-0.4.0 -- /home/edill/miniconda/bin/python
cachedir: .cache
rootdir: /home/edill/dev/maxpoint/conda-mirror, inifile:
plugins: xonsh-0.4.7, ordering-0.4
collected 4 items

test/test_conda_mirror.py::test_match PASSED
test/test_conda_mirror.py::test_cli[anaconda-linux-64] PASSED
test/test_conda_mirror.py::test_cli[conda-forge-linux-64] PASSED
test/test_conda_mirror.py::test_handling_bad_package PASSED

=============================================================================== 4 passed in 15.66 seconds ================================================================================

$ coverage report -m
Name                           Stmts   Miss  Cover   Missing
------------------------------------------------------------
conda_mirror/__init__.py           3      0   100%
conda_mirror/conda_mirror.py     169     15    91%   154, 160, 289-290, 333-342, 370-371, 401
------------------------------------------------------------
TOTAL                            172     15    91%
```
