ARG IMAGE_REGISTRY="416670754337.dkr.ecr.eu-west-2.amazonaws.com"
ARG BASE_VERSION="latest"
ARG BUILD_IMAGE=""            # ci-corretto-build-21 | ci-node-build-24 | ci-golang-build-1.25 | ...
ARG BUILD_VERSION="latest"

FROM ${IMAGE_REGISTRY}/devcontainers-base:${BASE_VERSION} AS tools

FROM ${IMAGE_REGISTRY}/${BUILD_IMAGE}:${BUILD_VERSION}

# Binaries
COPY --from=tools /usr/bin        /usr/bin
# Non-RPM installers (aws, gh, ...)
COPY --from=tools /usr/local      /usr/local

# Runtime libs
COPY --from=tools /lib64          /lib64
COPY --from=tools /usr/lib64      /usr/lib64

# helpers sometimes used at runtime
COPY --from=tools /usr/libexec    /usr/libexec
COPY --from=tools /usr/share      /usr/share
