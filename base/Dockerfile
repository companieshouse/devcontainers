# - Dockerfile to build the base 'tools' image, containing for example these tools:
#   - curl - jq - yq - git - wget - unzip - gpg - tmux - aws - gh
#
# - This base image is used for creating more specialised images (for Java - Golang - Node.js - Perl)
#
# - We start from ci-base-build (which, atm, contains: curl - git - wget - zip|unzip - gpg)
#

FROM 416670754337.dkr.ecr.eu-west-2.amazonaws.com/ci-base-build

# Architecture (currently just Apple Silicon)
ARG ARG_ARCH="arm64"
ARG ARCH_AARCH64="aarch64"

# pin versions
ARG JQ_VERSION="1.7.1"
ARG YQ_VERSION="4.47.1"
ARG TMUX_VERSION="3.2a"
ARG GH_VERSION="2.86.0"


RUN \
    dnf install -y \
      jq-${JQ_VERSION} \
      tmux-${TMUX_VERSION} \
      yq-${YQ_VERSION} \
    \
    # gh
    && curl -fL "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${ARG_ARCH}.rpm" -o /tmp/gh.rpm \
    && dnf install -y /tmp/gh.rpm \
    \
    # aws cli v2  (keep only the AWS CLI runtime)
    && curl -fL "https://awscli.amazonaws.com/awscli-exe-linux-${ARCH_AARCH64}.zip" -o /tmp/awscliv2.zip \
    && unzip /tmp/awscliv2.zip -d /tmp \
    && /tmp/aws/install \
         --bin-dir /usr/local/bin \
         --install-dir /usr/local/aws-cli \
         --update \
    && rm -rf /usr/local/aws-cli/v2/current/dist/aws_completer \
              /usr/local/aws-cli/v2/current/dist/doc \
    \
    # Cleanup
    && dnf clean all \
    && rm -rf /var/cache/dnf /tmp/*

