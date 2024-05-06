ARG TAG_OS=bookworm

ARG TAG_GOLANG=1.22.1
ARG TAG_PYTHON=3.11.8

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# stage: builder for `task` binary

FROM golang:${TAG_GOLANG}-${TAG_OS} AS go-task-builder

ENV GOBIN=/app/bin

WORKDIR /app

RUN go install github.com/go-task/task/v3/cmd/task@latest

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# stage: system layer and binaries

FROM python:${TAG_PYTHON}-${TAG_OS} as system

LABEL description="ITeen Python Image"
LABEL org.opencontainers.image.authors="Alexander Sidorov <a375296648709@gmail.com>"

ARG GID=9999
ARG UID=9999
ARG USERNAME=iteen

# This directory is used for packaging of this image.
ARG DIR_PACKAGE=/opt/iteen/

# This directory is used for the service itself.
# It will be empty in this image.
# Derived images MUST copy or mount the app code to here.
ARG DIR_SERVICE=/srv/iteen/

# This directory is used for tmpfs mounts.
ARG DIR_TMPFS=/mnt/tmpfs

# copy binaries

COPY --from=go-task-builder /app/bin/task /usr/bin/task
COPY ./bin/* /usr/local/bin/

# create user and dirs

RUN set -eux; \
    addgroup \
        --gid=${GID} \
        ${USERNAME} \
    && \
    adduser \
        --disabled-password \
        --home=${DIR_SERVICE} \
        --ingroup=${USERNAME} \
        --shell=/bin/bash \
        --uid=${UID} \
        ${USERNAME} \
    && \
    install -o ${USERNAME} -m 0755 -d ${DIR_TMPFS} \
    && \
    echo "User has been installed successfully."

# 1. Install dependencies

ARG DIR_PIP_CACHE=${DIR_PACKAGE}/.pip-cache
ARG DIR_POETRY_CACHE=${DIR_PACKAGE}/.poetry-cache
ARG PIP_VERSION=24.0
ARG POETRY_VERSION=1.8.2

WORKDIR ${DIR_PACKAGE}

# 1.1. install system packages

COPY ./dependencies/system.build.txt ./dependencies/system.prod.txt ./

RUN set -eux; \
    apt-get update --yes \
    && \
    apt-get upgrade --yes \
    && \
    apt-get install \
        --yes \
        $(cat system.build.txt) \
        $(cat system.prod.txt) \
    && \
    pip install \
        --cache-dir ${DIR_PIP_CACHE} \
        --check-build-dependencies \
        --compile \
        --no-input \
        --use-pep517 \
        pip==${PIP_VERSION} \
        poetry==${POETRY_VERSION} \
    && \
    echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
    && \
    locale-gen --purge 'en_US.UTF-8 UTF-8' \
    && \
    echo "System packages have been installed successfully."

ENV LANG en_US.utf8

# 1.2. install ODBC drivers

COPY ./dependencies/install-odbc.sh ./

RUN set -eux; \
    chmod +x install-odbc.sh \
    && \
    ./install-odbc.sh 2>&1 | tee ./install-odbc.log \
    && \
    stat /opt/mssql-tools18/bin \
    && \
    echo "ODBC drivers have been installed successfully."

# 1.3. install Python packages

ENV POETRY_CACHE_DIR=${DIR_POETRY_CACHE}
ENV POETRY_VIRTUALENVS_CREATE=false
ENV PYTHONDONTWRITEBYTECODE=
ENV PYTHONOPTIMIZE=2

COPY ./dependencies/pyproject.toml ./dependencies/poetry.lock ./


# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# stage: python packages

FROM system AS python-packages

RUN set -eux; \
    poetry install \
        --compile \
        --no-root \
        --with dev \
    && \
    echo "Python packages have been installed successfully."


# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# stage: final

FROM python-packages AS final

# 1.4. clean up

RUN set -eux; \
    poetry cache clear pypi --all \
    && \
    pip uninstall --yes poetry \
    && \
    pip cache purge \
    && \
    apt-get remove --yes $(cat system.build.txt) \
    && \
    apt-get purge -y --auto-remove \
    && \
    rm -rf \
        /etc/apt/sources.list.d/temp.list \
        /var/lib/apt/lists/* \
        ${DIR_PIP_CACHE} \
        ${DIR_POETRY_CACHE} \
    && \
    echo "System left-overs have been cleaned up."

USER ${USERNAME}

# set ENVs

ENV PYTHONDONTWRITEBYTECODE=True
ENV PYTHONOPTIMIZE=
ENV PYTHONPATH=${DIR_SERVICE}:${PYTHONPATH}
ENV PYTHONUNBUFFERED=True

WORKDIR ${DIR_SERVICE}

ENTRYPOINT [ "python" ]
