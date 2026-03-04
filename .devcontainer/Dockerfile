FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install basic tools + sudo + jq
RUN apt-get update && apt-get install -y \
    sudo \
    jq \
    curl \
    unzip \
    git \
    wget \
    ca-certificates \
    gnupg \
    vim \
    less \
    && rm -rf /var/lib/apt/lists/*

# Install Terraform
RUN wget -O /tmp/terraform.zip https://releases.hashicorp.com/terraform/1.13.5/terraform_1.13.5_linux_amd64.zip \
    && unzip /tmp/terraform.zip -d /usr/local/bin \
    && rm /tmp/terraform.zip

# Install AWS CLI v2
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip" \
    && unzip /tmp/awscliv2.zip -d /tmp \
    && /tmp/aws/install \
    && rm -rf /tmp/aws /tmp/awscliv2.zip

WORKDIR /workspaces