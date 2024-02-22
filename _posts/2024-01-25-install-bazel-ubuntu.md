---
layout: post
title: Instaling bazel on linux
date: 2024-01-25
description: Setting up your bazel configurations on linux
tags: bazel linux
categories: deployment
giscus_comments: true
related_posts: false

authors:
  - name: Zuhaib Sohail
    url: "https://github.com/ZuhaibSohail163"
    affiliations:
      name: Allied Consultants

---

### Required Dependencies

To install bazel for python project you need to install the following.

 - Bazelisk
 - poetry
 - pyenv

### Pyenv

First of all `What is pyenv?`
Basically, Pyenv is a tool to simplify installation and version changing in python. It helps developers quickly install or change the python version without needing to change the whole system.

Let's install pyenv so that every dependency or library installed will be exclusive to this project

Open terminal and execute the following commands

```apt update -y```

Now let's install pyenv dependencies

```
apt install -y make build-essential libssl-dev zlib1g-dev \
> libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev\
> libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl\
> git
```

After that, we will install the latest version on Pyenv. So, we will need to clone from the Pyenv repository.

```
pyenv install 3.10
pyenv global 3.10

```

And for the last step, we must add pyenv to our path so that the pyenv command is recognized globally.

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc

```

Finally, to start using pyenv, restart the shell and execute command:

```
pyenv activate

```

If you want to update pyenv you can use command

```
pyenv update

```

### Installing python

After configuring pyenv we need to install python.

First check if python is already installed

```
python3
```

if it shows error, run following commands

```
sudo apt update
```

```
sudo apt install python3
```

Now verify installation

```
python3 --version
```

### Installing Bazel
Bazelisk is a wrapper for Bazel written in Go. It automatically picks a good version of Bazel given your current working directory, downloads it from the official server (if required) and then transparently passes through all command-line arguments to the real Bazel binary. You can call it just like you would call Bazel.

### Supported Versions
Supported Ubuntu Linux platforms on the time of writing this blog are:

 - 22.04 (LTS)
 - 20.04 (LTS)
 - 18.04 (LTS)

On Linux: You can download Bazelisk binary on our [Releases]:(https://github.com/bazelbuild/bazelisk/releases) page and add it to your PATH manually, which also works on macOS and Windows.

To verify install, run the following command

```
bazel --version
```

### Installing poetry

Now let's install poetry

#### Introduction

Poetry is a tool for dependency management and packaging in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you. Poetry offers a lockfile to ensure repeatable installs, and can build your project for distribution


To install poetry you need to install **pip** first

```
sudo apt install python3-pip
```

To verify install, run the following command

```
pip3 --version
```

Now to install poetry run the following command

```
pip install poetry
```

To verify install, run the following command

```
poetry --version
```
### How do I configure LSP / DSP or other IDE tools ?
Bazel runs in a sandbox, however we are using [poetry][poetry] to manage dependencies. These dependencies are located in the third_party/ folder. As long as you set your IDE python interpreter to be the poetry env python, things should function normally in your IDE.

Here is one way to do this:

```
$ cd third_party

# Get the python executable form the virtual env used by poetry, this is the python interpreter to use

poetry env info --executable

# An easy way to get the IDE to use the right environment is getting the poetry shell, which will set the right python
# interpreter and then cd to the project root and fire up your editor

cd third_party

poetry shell

cd ..

code
```

### Setup Development Environment

Create the environment

Using terminal navigate to: third_party folder

Run these commands

```
poetry install

poetry shell

cd ..
```

After setting up environment you can perform some functions using following commands

```
# Run the fast_api locally (should start a service on 0.0.0.0:8080)
# Open api documentation available at "/docs"

bazel run //apps/api

# Run the starlette_api locally in debug mode (debugpy will wait for a debugger client to connect)

bazel run //apps/api --define DEBUG=1

# Clean everything

bazel clean --expunge

# Run all unit tests

bazel test $(bazel query 'attr(name, "module_tests", //...)') --test_output=streamed --test_arg="--disable-warnings" --nocache_test_results

# Run format and linting (before staging)

isort $(git ls-files "*.py" --modified) && black -l 120 $(git ls-files "*.py" --modified)

# Add files from git.patch

git app filename.patch

# Code structure in tree shape

```
.
├── BUILD.bazel
├── README.md
├── Vonoy.postman_collection.json
├── WORKSPACE.bazel
├── apps
│   └── api
│       ├── BUILD.bazel
│       ├── service.py
│       └── test_service.http
├── lib
│   ├── forecast_model
│   │   ├── api
│   │   │   ├── BUILD.bazel
│   │   │   ├── __init__.py
│   │   │   ├── pyproject.toml
│   │   │   └── routes.py
│   │   ├── models
│   │   │   ├── BUILD.bazel
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── __init__.cpython-39.pyc
│   │   │   │   └── core.cpython-39.pyc
│   │   │   ├── core.py
│   │   │   ├── pyproject.toml
│   │   │   └── tests
│   │   │       ├── conftest.py
│   │   │       └── test_forecast_input.py
│   │   ├── services
│   │   │   ├── BUILD.bazel
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── __init__.cpython-39.pyc
│   │   │   │   └── services.cpython-39.pyc
│   │   │   ├── models
│   │   │   │   ├── __init__.py
│   │   │   │   ├── __pycache__
│   │   │   │   │   ├── __init__.cpython-39.pyc
│   │   │   │   │   └── forecast_model.cpython-39.pyc
│   │   │   │   └── forecast_model.py
│   │   │   ├── pyproject.toml
│   │   │   ├── services.py
│   │   │   └── tests
│   │   │       ├── conftest.py
│   │   │       └── test_get_forecast_prediction.py
│   │   └── testsupport
│   │       ├── BUILD.bazel
│   │       ├── __init__.py
│   │       ├── forecast_sample_data.py
│   │       └── pyproject.toml
│   └── vonoy
│       └── tools
│           └── pytest
│               ├── BUILD.bazel
│               ├── defs.bzl
│               └── pytest_wrapper.py
├── pyproject.toml
├── third_party
│   ├── BUILD.bazel
│   ├── poetry.lock
│   └── pyproject.toml
└── tree_2.1.1-2_amd64.deb

```

```