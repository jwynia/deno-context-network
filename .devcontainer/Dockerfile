FROM mcr.microsoft.com/devcontainers/base:debian
ENV DENO_INSTALL=/deno
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends curl ca-certificates unzip \
    && mkdir -p /deno \
    && curl -fsSL https://deno.land/install.sh | sh \
    && chown -R vscode /deno
ENV PATH=${DENO_INSTALL}/bin:${PATH} \
    DENO_DIR=${DENO_INSTALL}/.cache/deno