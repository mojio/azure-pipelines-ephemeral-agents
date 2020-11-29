FROM ubuntu:16.04

# To make it easier for build and release pipelines to run apt-get,
# configure apt to not require confirmation (assume the -y argument by default)
ENV DEBIAN_FRONTEND=noninteractive
RUN echo "APT::Get::Assume-Yes \"true\";" > /etc/apt/apt.conf.d/90assumeyes

RUN apt-get update \
&& apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        jq \
        git \
        iputils-ping \
        libcurl3 \
        libicu55 \
        libunwind8 \
        netcat \
        wget \ 
        apt-transport-https \ 
        gnupg \ 
        openssh-client

# this is needed because otherwise it can't find dotnetcore 3.1 later on.  taken from:
#  https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-ubuntu-1604#troubleshoot-the-package-manager
RUN wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o microsoft.asc.gpg
RUN mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
RUN wget -q https://packages.microsoft.com/config/ubuntu/16.04/prod.list
RUN mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
RUN chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
RUN chown root:root /etc/apt/sources.list.d/microsoft-prod.list

# as per https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-ubuntu-1604#install-the-net-core-runtime,
#  after installing apt-transport-https we need to run update again before installing dotnetcore runtime.....i don't really know why
RUN apt-get update
RUN apt-get install -y --no-install-recommends dotnet-sdk-3.1

# Install AZ CLI
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash
RUN echo "AZURE_EXTENSION_DIR=/usr/local/lib/azureExtensionDir" | tee -a /etc/environment \
    && mkdir -p /usr/local/lib/azureExtensionDir

# Install powershell core, and azure module
RUN apt-get install -y powershell
RUN /usr/bin/pwsh -Command "Install-Module PowerShellGet -Force"
RUN /usr/bin/pwsh -Command "Install-Module -Name Az -AllowClobber -Force" # -Scope CurrentUser
# RUN /usr/bin/pwsh -Command "Install-Module -Name Az.Accounts -AllowClobber -Force" # -Scope CurrentUser

WORKDIR /azp

RUN mkdir ./patches
COPY ./patches/AgentService.js ./patches/
COPY ./start.sh .
COPY ./start-once.sh .
RUN chmod +x start.sh
RUN chmod +x start-once.sh

CMD ["./start-once.sh"]
