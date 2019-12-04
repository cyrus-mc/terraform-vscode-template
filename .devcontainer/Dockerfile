FROM golang:1.13.4-buster

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Or your actual UID, GID on Linux if not the default 1000
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt, install packages and tools
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git procps lsb-release unzip \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# install terraform
ENV TERRAFORM_VERSION=0.12.17
RUN curl https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip -o /tmp/terraform.zip && \
      unzip /tmp/terraform.zip -d /usr/local/bin && \
      chmod +x /usr/local/bin/terraform && \
      rm -f /tmp/terraform.zip

# install terraform language server
ENV TERRAFORM_LANGUAGE_SERVER_VERSION=0.0.9
RUN curl -L https://github.com/juliosueiras/terraform-lsp/releases/download/v${TERRAFORM_LANGUAGE_SERVER_VERSION}/terraform-lsp_${TERRAFORM_LANGUAGE_SERVER_VERSION}_linux_amd64.tar.gz -o /tmp/terraform-lsp.tar.gz && \
         tar -C /usr/local/bin/ -xvzf /tmp/terraform-lsp.tar.gz terraform-lsp && \
         rm -f /tmp/terraform-lsp.tar.gz

COPY providers.tf /usr/local/bin/providers.tf
RUN cd /usr/local/bin && \
    terraform init && \
    mv /usr/local/bin/.terraform/plugins/**/terraform-provider* /usr/local/bin

USER $USERNAME
