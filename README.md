# Flower Tutorial for PyCon DE & PyData 2025
<a href="https://flower.ai/">
  <img src="https://flower.ai/_next/image/?url=%2F_next%2Fstatic%2Fmedia%2Fflwr-head.4d68867a.png&w=384&q=75" width="140px" alt="Flower Website"  align="right" />
</a>

This is the Flower tutorial repository for PyCon DE &amp; PyData 2025 talk "[The Future of AI is Federated](https://2025.pycon.de/talks/9Y9DM8/)". It describes the prerequisites to setup your tutorial environment and outlines the 3 parts of the tutorials. It also includes a bonus part 4 that walks you through setting up a local deployment with both node and secure TLS connection:

1. [Create a Flower App and run it using the Simulation Runtime](#part-1---flower-quickstart-with-pytorch)
2. [Run a Flower App on a remote SuperLink](#part-2---flower-simulation-runtime-on-a-remote-superlink)
3. [Deploy and run a Flower App using the Deployment Runtime and Docker](#part-3---flower-deployment-runtime-on-a-remote-superlink)
4. [(Bonus) Deploy SuperNodes and a SuperLink with node and TLS authentication](#part-4---flower-deployment-runtime-with-tls-and-node-authentication)

At the end of this README, we've included a [`flwr` CLI cheatsheet](#flwr-cli-cheatsheet) to summarize the basic commands used in this tutorial.

Let's get started 🚀!

## Prerequisites

The easiest way to start using this repository is to use GitHub Codespaces. The _only_ requirement is that you need to have an active GitHub account. Click on the badge below to launch your codespace with all of the code contents in this repository.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=pyconde2025&repo=966052297)

Additionally, you should have [Docker installed](https://docs.docker.com/engine/install/) on your system.

The two alternatives to using Codespaces are:
1. Clone this repository and run the Dev Container from your VS Code.
2. Clone this repository and install the latest version of Flower in a new Python environment with `pip install -U "flwr[simulation]"`.

If you choose the manual option to setup your tutorial environment, here are the prerequisites:
- Use macOS or Ubuntu
- Have a Python environment (minimum is Python 3.9, but Python 3.10, 3.11, or 3.12 is recommended)
- Have `flwr` installed:
    ```shell
    pip install -U "flwr[simulation]"
    ```
- Have an IDE, e.g. VS Code, and install the [VS Code Containers Extension](vscode:extension/ms-vscode-remote.remote-containers).


## Part 1 - Flower Quickstart with PyTorch

🎯 **Feature Highlights:**
- Create a new Flower app from templates using `flwr new`
- Start a Flower app using `flwr run`
- Understand basic federated learning workflow with Flower
- Customize the hyperparameters of your workflow

Let's begin by creating a Flower app. This can be done easily using `flwr new` and then choosing one of the available templates. Let's use the `NumPy` template.

```shell
flwr new awesomeapp
# Then follow the prompt
```

The above command would create the following directory and content:

```shell
awesomeapp
├── README.md
├── awesomeapp
│   ├── __init__.py
│   ├── client_app.py   # Defines your ClientApp
│   ├── server_app.py   # Defines your ServerApp
│   └── task.py         # Defines your model, training, and data loading
└── pyproject.toml      # Project metadata like dependencies and configs
```

Assuming you have already installed the dependencies for your app, you can run the app by doing:

```shell
cd path/to/app_dir # the directory where the pyproject.toml is
flwr run .
```

> [!TIP]
> This section uses one of the pre-built templates available in the Flower platform. Learn more about other quickstart tutorials in [quickstart documentation](https://flower.ai/docs/framework/tutorial-quickstart.html).

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

You can make your simulation larger (as large as you want!) by increasing the number of supernodes. Additionally, you can control how many compute and memory resources these get assigned. Let's do this by defining a new federation that we'll name `"simulation-xl"` (note that you can choose any other name):

```toml
[tool.flwr.federations.simulation-xl]
options.num-supernodes = 200
options.backend.client-resources.num-cpus = 1 # each ClientApp assumes to use 1 CPU
```

Then, to run the app on this new federation you execute (the second argument to [flwr run](https://flower.ai/docs/framework/ref-api-cli.html#flwr-run) indicates the federation to use):

```shell
flwr run . simulation-xl
```

[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)

## Part 2 - Flower Simulation Runtime on a Remote SuperLink

🔐 **Feature Highlights:**
- Login to the SuperLink using `flwr login`
- Run a federated learning simulation remotely

In Part 1, we ran a federated learning simulation locally on your system. When experimenting with your federated learning system, it is useful to be able to run the simulations on a remote machine with more resources (such as GPUs and CPUs). To do so without directly connecting to the remote machine, we can spin up a [Flower SuperLink](https://flower.ai/docs/framework/ref-api-cli.html#flower-superlink) on it and then run `flwr run` using the address of the remote machine. In this way, you can submit multiple runs to the remote machine and let the SuperLink coordinate the executions of your submitted Flower apps 🤩! 

> [!NOTE]
> This section explains how you can run a Flower app on a remote server as an _authenticated user_. To access the server and try it out, please register a Flower account by going to [flower.ai](https://flower.ai/), click on the yellow "Sign Up" button on the top right corner of the webpage, and complete the sign up process.

For this tutorial, we've setup a temporary SuperLink which you can connect to, which is at `pyconde25.flower.ai`. You can also try to create and run other templates from `flwr new`. The list of supported templates that have been preinstalled in this SuperLink is:  PyTorch, TensorFlow, sklearn, JAX, and NumPy. To use the remote SuperLink, add a new federation table called `[tool.flwr.federations.pyconde25]` to your `pyproject.toml`:

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

## Part 3 - Flower Deployment Runtime on a Remote SuperLink

🐳 **Feature Highlights**
- Deploy SuperNode on your device using Docker and connect it to a remote SuperLink
- Enable secure TLS connection between SuperNodes and SuperLink

> [!NOTE]
> This section introduces the relevant components to run Flower in deployment without making use of node authentication. This will be presented in the next section. Read more about the [Flower Architecture](https://flower.ai/docs/framework/explanation-flower-architecture.html) in the documentation.

In part 3, we'll move from the simulation/research approach and _deploy_ our Flower apps so that federated learning will take place on a cross-device setting. 

To deploy your Flower app, we first need to launch the two long-running components: the server, i.e. SuperLink, and clients, i.e. SuperNodes. Both SuperLink and SuperNodes can be launched in either `--isolation subprocess` mode (the default) , or the `--isolation process` mode. The `subprocess` mode allows you to run the `ServerApp` and `ClientApp`s in the same process of the SuperLink and SuperNodes, respectively. This has the benefit of a minimal deployment since all of the app dependencies can be packaged into the SuperLink and SuperNode images. For the `process` mode, the `ServerApp` and `ClientApp` will run a separate externally-managed processes. This allows, for example, to run SuperNode and `ClientApp` in separate Docker containers with different sets of dependencies installed, allowing the SuperLink and SuperNode run with the absolute minimal image requirements.

For the purposes of this tutorial, we have deployed another SuperLink for you at `91.99.49.68`. We have also enabled secure TLS connection using self-signed certificates, which we have already generated for you.

> [!CAUTION]
> Using self-signed certificates is for testing purposes only and not recommended for production.

Now, in this interactive part of the tutorial, you can participate in the _first_ PyCon DE 2025 Flower federation by spinning up a SuperNode on your local machine. To do so, from the parent directory of this repo, run:

```shell
docker run \
  --volume "$(pwd)/certificates:/certificates:ro" \
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

> [!TIP]
> In this section, we used the official Flower Docker images to deploy the SuperLink and the SuperNodes. Check out Flower's [Docker Hub repository](https://hub.docker.com/u/flwr) to learn about the available base and Python images. Learn more about deploying Flower with Docker in our [documentation](https://flower.ai/docs/framework/docker/index.html).


[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)

## Part 4 - Flower Deployment Runtime with TLS and Node Authentication

🔐 **Feature Highlights**
- Enable node and secure TLS connection between SuperNodes and the SuperLink
- Start SuperNodes and SuperLink via CLI

> [!NOTE]
> Part 4 will be the stretch section of the PyCon DE tutorial. Feel free to follow the tutorial in your own free time as you will be deploying the SuperLink and SuperNodes in your local machine.

In this section, we'll enable secure TLS connection and `SuperNode` authentication in the deployment mode. The TLS connection will be enabled between the `SuperLink` and `SuperNode`s, as well as between the Flower CLI and the `SuperLink`. For authenticated `SuperNode`s, each identity of the `SuperNode` is verified when connecting to the `SuperLink`. 

> [!NOTE]
> For more details, refer to the documentation on [enabling TLS connections](https://flower.ai/docs/framework/how-to-enable-tls-connections.html) and authenticating [`SuperNode`s](https://flower.ai/docs/framework/how-to-authenticate-supernodes.html).

### Generate public and private keys

In this repo, we provide a utility script called [`generate.sh`](generate.sh) and a configuration file [`certificate.conf`](certificate.conf). The script by default generates self-signed certificates for creating a secure TLS connection and three private and public key pairs for one server and two clients. The script also generates a CSV file that includes each of the generated (client) public keys. The script uses `certificate.conf`, which is a configuration file typically used by OpenSSL to generate a Certificate Signing Request (CSR) or self-signed certificates.

> [!CAUTION]
> Using self-signed certificates is for testing purposes only and not recommended for production.

First, copy `generate.sh` and `certificate.conf` to your Flower App. Then, run the script: 

```shell
cp generate.sh certificate.conf path/to/app_dir
./generate.sh
```

> [!NOTE]
> You can generate more keys by specifying the number of client credentials that you wish to generate, as follows: `./generate.sh {your_number_of_clients}`

After running the script, the following new folders and files will be generated:

```shell
awesomeapp
├── README.md
├── certificate.conf
├── certificates    # Folder containing certificates for TLS connection
│   ├── ca.crt      # *Certificate Authority (CA) certificate
│   ├── ca.key      # Private key for CA
│   ├── ca.srl      # Serial number file for CA
│   ├── server.csr  # Server certificate signing request
│   ├── server.key  # *Server private key
│   └── server.pem  # *Server certificate
├── generate.sh
├── keys                          # Folder containing keys for authenticating SuperNodes
│   ├── client_credentials_1      # Private key for client 1
│   ├── client_credentials_1.pub  # Public key for client 1
│   ├── client_credentials_2      # Private key for client 2
│   ├── client_credentials_2.pub  # Public key for client 2
│   ├── client_public_keys.csv    # *Public keys for both clients
│   ├── server_credentials        # *Private server credentials
│   └── server_credentials.pub    # *Public server credentials
├── awesomeapp
│   ├── __init__.py
│   ├── client_app.py
│   ├── server_app.py
│   └── task.py
└── pyproject.toml

```

The files that are preceded by asterisks `*` will be used in our deployment.

###  Launch `SuperLink` and `SuperNode`s with certificates and keys

> [!NOTE]
> From this point onwards, ensure that your working directory where you execute all Flower commands is in `/path/to/app_dir`. This is because the paths to the certificates and keys are relative to execution directory. Optionally, modify the paths below to absolute paths. 

Launch a local instance of your `SuperLink` with additional commands.

```shell
flower-superlink \
    --ssl-ca-certfile certificates/ca.crt \
    --ssl-certfile certificates/server.pem \
    --ssl-keyfile certificates/server.key \
    --auth-list-public-keys keys/client_public_keys.csv \
    --auth-superlink-private-key keys/server_credentials \
    --auth-superlink-public-key keys/server_credentials.pub
```

The first three flags defines the three certificates paths: CA certificate (`--ssl-ca-certfile`), server certificate (`--ssl-certfile`) and server private key (`--ssl-keyfile`), respectively. The following three flags defines the path to a CSV file storing all known node public keys (`--auth-list-public-keys`), and the paths to the server’s private (`--auth-superlink-private-key`) and public keys (`--auth-superlink-public-key`).

Next, we restart the `SuperNode`s with a secure TLS connection and authentication. Run the following command to start the first `SuperNode`:

```shell
flower-supernode \
    --superlink="127.0.0.1:9092" \
    --root-certificates certificates/ca.crt \
    --auth-supernode-private-key keys/client_credentials_1 \
    --auth-supernode-public-key keys/client_credentials_1.pub \
    --clientappio-api-address="0.0.0.0:9094" \
    --node-config 'num-partitions=10 partition-id=0'
```

Then, the next command to start the second `SuperNode`:

```shell
flower-supernode \
    --superlink="127.0.0.1:9092" \
    --root-certificates certificates/ca.crt \
    --auth-supernode-private-key keys/client_credentials_2 \
    --auth-supernode-public-key keys/client_credentials_2.pub \
    --clientappio-api-address="0.0.0.0:9095" \
    --node-config 'num-partitions=10 partition-id=1'
```

Now, we need to modify our `pyproject.toml` so that our Flower CLI will connect in a secure way to our `SuperLink`. In the `pyproject.toml`, make the following changes:

```toml
[tool.flwr.federations.pyconde]
address = "127.0.0.1:9093"                # Point to the local SuperLink address
root-certificates = "certificates/ca.crt" # Points to the path of the CA certificate. Must be relative to `pyproject.toml`.
```

Finally, we can launch the Run in the same way as above, but now with TLS and client authentication like this:

```shell
flwr run . pyconde --stream
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

## References

Here are some useful references to expand on the networking and architecture topics covered in this tutorial:
- Flower federated learning architecture ([link](https://flower.ai/docs/framework/explanation-flower-architecture.html))
- Flower network communication ([link](https://flower.ai/docs/framework/ref-flower-network-communication.html))
- Quickstart Docker guides ([link](https://flower.ai/docs/framework/docker/index.html))
- Flower official Docker images ([link](https://hub.docker.com/u/flwr))

And here are some links to the Flower quickstarts and tutorials.
- Flower tutorial ([link](https://flower.ai/docs/framework/tutorial-series-what-is-federated-learning.html))
- Flower quickstarts ([link](https://flower.ai/docs/framework/tutorial-quickstart.html))
- Federated AI with Embedded Devices using Flower ([link](https://flower.ai/docs/examples/embedded-devices.html))

[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)

## Community

- [Join us on Slack](https://flower.ai/join-slack)
- [Join us on Flower Discuss](https://discuss.flower.ai)


[🔼 Back to top](#flower-tutorial-for-pycon-de--pydata-2025)