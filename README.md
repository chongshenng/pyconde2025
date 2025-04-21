# Flower Tutorial for PyCon DE & PyData 2025
<a href="https://flower.ai/">
  <img src="https://flower.ai/_next/image/?url=%2F_next%2Fstatic%2Fmedia%2Fflwr-head.4d68867a.png&w=384&q=75" width="140px" alt="Flower Website"  align="right" />
</a>

This is the Flower tutorial repository for PyCon DE &amp; PyData 2025 talk "[The Future of AI is Federated](https://2025.pycon.de/talks/9Y9DM8/)". It describes the prerequisites to setup your tutorial environment and outlines the 3 parts of the tutorials.

At the end of this README, we've included a [`flwr` CLI cheatsheet](#flwr-cli-cheatsheet) to summarize the basic commands used in this tutorial.

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

[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)

## Part 2 - Flower Simulation Runtime on a Hosted Server

In Part 1, we ran a federated learning simulation locally on your system. When experimenting with your federated learning system, it is useful to be able to run the simulations on a remote machine with more resources (such as GPUs and CPUs). To do so without directly connecting to the remote machine, we can spin up a [Flower SuperLink](https://flower.ai/docs/framework/ref-api-cli.html#flower-superlink) on it and then run `flwr run` using the address of the remote machine. In this way, you can submit multiple runs to the remote machine and let the SuperLink coordinate the executions of your submitted Flower apps 🤩! 

> [!NOTE]
> This section explains how you can run a Flower app on a remote server as an _authenticated user_. To access the server and try it out, please register a Flower account by going to [flower.ai](https://flower.ai/), click on the yellow "Sign Up" button on the top right corner of the webpage, and complete the sign up process.

For this tutorial, we've setup a temporary SuperLink which you can connect to, which is at `pyconde25.flower.ai`. To use it, add a new table called `[tool.flwr.federations.pyconde25]` to your `pyproject.toml`:

```toml
[tool.flwr.federations.pyconde25]
address = "pyconde25.flower.ai"  # Sets the address of the remote SuperLink
enable-user-auth = true          # Enables user authentication
options.num-supernodes = 10
```

Next, ensure that you're logged in (so that you can run your Flower app in an authenticated user session), run:

```shell
flwr login . pyconde25
```

Click on the URI and login with your credentials that you provided during the sign up process. Then, you can run the app on the remote server by doing:

```shell
flwr run . pyconde25 --stream
```

Note that the `--stream` option is to stream the logs from the Flower app. You can safely run `CTRL+C` without interrupting the execution since it is running remotely on the server. The run status can be viewed by running `flwr ls . pyconde25 <run_id>`.

[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)

## Part 3 - Flower Deployment Runtime on a Hosted Server

> [!NOTE]
> This section introduces the relevant components to run Flower in deployment without making use of node authentication. This will be presented in the next section. Read more about the [Flower Architecture](https://flower.ai/docs/framework/explanation-flower-architecture.html) in the documentation.

In part 3, we'll move from the simulation/research approach and _deploy_ our Flower apps so that federated learning will take place on a cross-device setting. 

To deploy your Flower app, we first need to launch the two long-running components: the server, i.e. SuperLink, and clients, i.e. SuperNodes. Both SuperLink and SuperNodes can be launched in either `--isolation subprocess` mode (the default) , or the `--isolation process` mode. The `subprocess` mode allows you to run the `ServerApp` and `ClientApp`s in the same process of the SuperLink and SuperNodes, respectively. This has the benefit of a minimal deployment since all of the app dependencies can be packaged into the SuperLink and SuperNode images. For the `process` mode, the `ServerApp` and `ClientApp` will run a separate externally-managed processes. This allows, for example, to run SuperNode and `ClientApp` in separate Docker containers with different sets of dependencies installed, allowing the SuperLink and SuperNode run with the absolute minimal image requirements.

For the purposes of this tutorial, we have deployed a SuperLink for you at `91.99.49.68`.

Now, in this interactive part of the tutorial, you can participate in the _first_ PyCon DE 2025 Flower federation by spinning up a SuperNode on your local machine. To do so, from the parent directory of this repo, run:

```shell
docker run \
  -v "$(pwd)/certificates:/certificates:ro" \
  flwr/supernode:1.18.0.dev20250403 \
  --superlink="91.99.49.68:9092" \
  --root-certificates /certificates/ca.crt \
  --node-config 'num-partitions=10 partition-id=0'
```

You should be able to see the following:

```shell
INFO :      Starting Flower SuperNode
INFO :      Starting Flower ClientAppIo gRPC server on 0.0.0.0:9094
INFO :
```

## `flwr` CLI Cheatsheet

In this tutorial, we used several `flwr` CLI commands including `flwr new`, `flwr run`, `flwr ls` and `flwr log`. A cheatsheet of all the relevant commands are shown below.

> [!TIP]
> For more details on all Flower CLI commands, please refer to the [`flwr` CLI reference](https://flower.ai/docs/framework/ref-api-cli.html#basic-commands) documentation.

| Command        | Description                             | Example Usage               |
|----------------|-----------------------------------------|-----------------------------|
| `flwr new`     | Create a new Flower App from a template | `flwr new`                  |
| `flwr run`     | Run the Flower App in the CWD `<.>` on the `<federation>` federation     | `flwr run . <federation>`                |
|                | Run the Flower App and stream logs from the `ServerApp`                  | `flwr run . <federation> --stream`       |
| `flwr ls`      | List the Run statuses on the `<federation>` federation on the SuperLink (Default) | `flwr ls . <federation>`    |
|                | List the Run status of one `<run-id>` on the SuperLink                            | `flwr ls . <federation> --run-id <run-id>` |
| `flwr log`     | Stream logs from one `<run-id>` (Default) | `flwr log <run-id> . <federation>` |
|                | Print logs from one `<run-id>`            | `flwr log <run-id> . <federation> --show` |

[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)