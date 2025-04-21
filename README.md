# Flower Tutorial for PyCon DE & PyData 2025

This is the Flower tutorial repository for PyCon DE &amp; PyData 2025 talk "The Future of AI is Federated". It describes the prerequisites to setup your tutorial environment and outlines the 3 parts of the tutorials.

Let's get started 🚀!

## Prerequisites

The easiest way to start using this repository is to use GitHub Codespaces. The _only_ requirement is that you need to have an active GitHub account. Click on the badge below to launch your codespace with all of the code contents in this repository.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=pyconde2025&repo=966052297)

The two alternatives to using Codespaces are:
1. Clone this repository and run the Dev Container from your VS Code.
2. Clone this repository and install the latest version of Flower in a new Python environment with `pip install -U "flwr[simulation]"`.

If you choose the manual option to setup your tutorial environment, here are the prerequisites:
- Use macOS or Ubuntu
- Have a Python environment (minimum is Python 3.9, but Python 3.10, 3.11, or 3.12 is recommended)
- Have `flwr` installed:
    ```shell
    pip install -U flwr
    ```
- Have [Docker installed](https://docs.docker.com/engine/install/) on your system
- Have an IDE, e.g. VS Code

Additionally, install the [VS Code Containers Extension](vscode:extension/ms-vscode-remote.remote-containers).


## Part 1 - Flower Quickstart with PyTorch

Let's begin by creating a Flower app. This can be done easily using `flwr new` and then choosing one of the available templates. Let's use the `PyTorch` template.

```shell
flwr new awesomeapp
# Then follow the prompt
```

The above command would create the following directory and content:

```shell
awesomeapp
├── awesomeapp
│   ├── __init__.py
│   ├── client_app.py   # Defines your ClientApp
│   ├── server_app.py   # Defines your ServerApp
│   └── task.py         # Defines your model, training, and data loading
├── pyproject.toml      # Project metadata like dependencies and configs
└── README.md
```

Assuming you have already installed the dependencies for your app, you can run the app by doing:

```shell
cd path/to/app_dir # the directory where the pyproject.toml is
flwr run .
```

### Overriding the `run-config`

The Run config sets hyperparameters for your app at runtime. These are defined in the `[tool.flwr.app.config]` section of your app's `pyproject.toml`, which you can extend. The run config can be overridden directly from the CLI:

```shell
flwr run . --run-config="num-server-rounds=5 fraction-fit=0.333"
```

### Configuring your simulation

> [!TIP]
> This section provides a quick overview of how to modify the default simulation settings. Learn more about Flower's Simulation Runtime in [the documentation](https://flower.ai/docs/framework/how-to-run-simulations.html).

The templates available through `flwr new` create a relatively small simulation with just 10 nodes. This is defined in the `pyproject.toml` and should look as follows:
```toml
[tool.flwr.federations.local-simulation]
options.num-supernodes = 10
```

## Part 2 - Flower Simulation on a Hosted Server

In Part 1, we ran a federated learning simulation locally on your system. When experimenting with your federated learning system, it is useful to be able to run the simulations on a remote machine with more resources (such as GPUs and CPUs). To do so without directly connecting to the remote machine, we can spin up a [Flower SuperLink](https://flower.ai/docs/framework/ref-api-cli.html#flower-superlink) on it and then run `flwr run` using the address of the remote machine. In this way, you can submit multiple runs to the remote machine and let the SuperLink coordinate the executions of your submitted Flower apps. 

> [!NOTE]
> This section explains how you can run a Flower app on a remote server as an _authenticated user_. To access the server and try it out, please register a Flower account by going to [flower.ai](https://flower.ai/), click on the yellow "Sign Up" button on the top right corner of the webpage, and complete the sign up process.

For this tutorial, we've setup a temporary SuperLink which you can connect to, which is at `xyz.flower.ai`. To use it, add a new table called `[tool.flwr.federations.pyconde]` to your `pyproject.toml`:

```toml
[tool.flwr.federations.pyconde]
address = "xyz.flower.ai"
enable-user-auth = true
options.num-supernodes = 10
```

Next, ensure that you're logged in (so that you can run your Flower app in an authenticated user session), run:

```shell
flwr login . pyconde
```

Click on the URI and login with your credentials that you provided during the sign up process. Then, you can run the app on the remote server by doing:

```shell
flwr run . pyconde --stream
```

Note that the `--stream` option is to stream the logs from the Flower app. You can safely run `CTRL+C` without interrupting the execution since it is running remotely on the server. The run status can be viewed by running `flwr ls . pyconde <run_id>`.
