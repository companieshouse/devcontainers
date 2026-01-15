# - Dockerfile to build the base 'tools' image, containing for example these tools:
#   - curl - jq - git - wget - unzip - gnupg - tmux - yq - aws - gh
#
# - This base image is used for creating more specialised images (for Java - Golang - Node.js - Perl)
#
# - We start from Debian slim (~ 108 MB) which uses glibc
#
# - Alpine would be much smaller (~ 7 MB), but it uses musl, not glibc, and runs into issues later with
#   glibc-dependent tools (ex Node native modules, some CPAN installs, Java, ...)
#
# - impacts on size:
#     TOOL             ~ SIZE      INCLUDES/PULLS
#   -------------------------------------------------
#   - AWS CLI v2       264  MB     Embedded Python (Botocore / SSL certs / Dozens of Python modules)
#   - Git              124  MB     Perl / openssl / libexpat
#   - GitHub CLI (gh)   53  MB     Go runtime / libstdc++ / libc++ / Git integration
#   - yq                25  MB
#   - ca-certificates   14  MB
#   - gnupg             15  MB
#   - curl               8  MB
#   - tmux               3  MB
#   - wget               1  MB
#   - jq                 1  MB
#   - unzip              1  MB
#
# - The major offender, for the size, is aws cli v2 (~ 50% of the final image size):
#    - it is not a single static binary: /usr/local/bin/aws is a symlink into a self-contained runtime dir,
#      (where aws bundles its own Python runtime).
#    - we can just slim it down removing non-essential content: documentation - examples - completion scripts - installer metadata


FROM debian:bookworm-slim

# Architecture (currently just Apple Silicon)
ARG ARCH="arm64"
ARG ARCH_AARCH64="aarch64"

# Debian snapshot (APT reproducibility) - latest at the time of writing
ARG DEBIAN_SNAPSHOT="20260101T022824Z"
ARG DEBIAN_SNAPSHOT_URL="http://snapshot.debian.org/archive/debian"
ARG DEBIAN_SNAPSHOT_SECURITY_URL="${DEBIAN_SNAPSHOT_URL}-security"

# Apt pkg versions
# (discovered via: apt-cache madison  curl wget unzip jq gnupg tmux git)
ARG CURL_APT_VERSION="7.88.1-10+deb12u14"
ARG WGET_APT_VERSION="1.21.3-1+deb12u1"
ARG UNZIP_APT_VERSION="6.0-28"
ARG JQ_APT_VERSION="1.6-2.1+deb12u1"
ARG GNUPG_APT_VERSION="2.2.40-1.1+deb12u2"
ARG TMUX_APT_VERSION="3.3a-3"
ARG GIT_APT_VERSION="1:2.39.5-0+deb12u3"

# Curl-installed tools version - latest at the time of writing
ARG YQ_VERSION="v4.50.1"
ARG GH_VERSION="2.85.0"

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=/usr/local/bin:$PATH


RUN \
    \
    # Configure snapshot-based APT
    printf "deb %s/%s bookworm main\ndeb %s/%s bookworm-security main\n" \
            "${DEBIAN_SNAPSHOT_URL}"          "${DEBIAN_SNAPSHOT}" \
            "${DEBIAN_SNAPSHOT_SECURITY_URL}" "${DEBIAN_SNAPSHOT}" \
    > /etc/apt/sources.list \
    \
    # install pinned versions
    && apt-get -o Acquire::Check-Valid-Until=false update \
    && apt-get install -y --no-install-recommends \
       ca-certificates \
       curl=${CURL_APT_VERSION} \
       wget=${WGET_APT_VERSION} \
       unzip=${UNZIP_APT_VERSION} \
       jq=${JQ_APT_VERSION} \
       gnupg=${GNUPG_APT_VERSION} \
       tmux=${TMUX_APT_VERSION} \
       git=${GIT_APT_VERSION} \
       # tar already available in the debian image
    \
    # yq
    && curl -fL https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${ARCH} -o /usr/local/bin/yq \
    && chmod +x /usr/local/bin/yq \
    \
    # gh
    && curl -fL https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${ARCH}.tar.gz \
    | tar -xz -C /tmp \
    && mv /tmp/gh_${GH_VERSION}_linux_${ARCH}/bin/gh /usr/local/bin/gh \
    && chmod +x /usr/local/bin/gh \
    \
    # aws cli v2  (keep only the AWS CLI runtime)
    && curl -L "https://awscli.amazonaws.com/awscli-exe-linux-${ARCH_AARCH64}.zip" -o /tmp/awscliv2.zip \
    && unzip /tmp/awscliv2.zip -d /tmp \
    && /tmp/aws/install \
        --install-dir /opt/aws-cli \
        --bin-dir /usr/local/bin \
    \
    # ---- remove non-essential content after install
    && rm -rf \
        /opt/aws-cli/v2/current/examples \
        /opt/aws-cli/v2/current/dist/awscli/examples \
        /opt/aws-cli/v2/current/dist/awscli/data/*.rst \
        /opt/aws-cli/v2/current/dist/awscli/data/*.txt \
        /opt/aws-cli/v2/current/dist/awscli/data/ac.index \
    \
    # Cleanup
    && rm -rf /tmp/* /var/lib/apt/lists/*
