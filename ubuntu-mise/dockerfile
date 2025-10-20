ARG UBUNTU_VERSION
FROM ubuntu:${UBUNTU_VERSION}

USER root
RUN useradd -rm -d /home/ubuntu -s /bin/bash -U -G sudo ubuntu || true
RUN apt update && apt install -y curl git
RUN echo "StrictHostKeyChecking no" | tee -a /etc/ssh/ssh_config

USER ubuntu

RUN curl -fsSL https://mise.jdx.dev/install.sh | sh
RUN echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
RUN echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.profile
