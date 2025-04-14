FROM ubuntu:24.04

# https://code.visualstudio.com/docs/remote/containers-advanced#_creating-a-nonroot-user
ARG USERNAME=flwr-vscode

# Create the user
RUN useradd -m -s /bin/bash $USERNAME \
    && apt-get update \
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get install -y sudo bash \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# A persistent volume will be configured in devcontainer.json so we don't loose the commandhistory
# after rebuilding the container
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
    && mkdir /commandhistory \
    && touch /commandhistory/.bash_history \
    && chown -R $USERNAME /commandhistory \
    && echo $SNIPPET >> "/home/$USERNAME/.bashrc"

# Install system dependencies
RUN apt-get update
RUN apt-get install -y curl wget gnupg python3 python-is-python3 python3-pip git \
    build-essential tmux vim python3-venv

# Create and activate a virtual environment
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN python -m pip install -U pip
# RUN pip install flwr\[simulation\]==1.17.0
RUN pip install "flwr[simulation]@git+https://github.com/adap/flower.git@add-secure-conn-with-ca-cert"

USER $USERNAME
ENV PATH="/home/$USERNAME/.local/bin:${PATH}"
