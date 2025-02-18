# The build stage
# ---------------
FROM python:3.9-bullseye as build-stage

# VULN_SCAN_TIME=2022-05-30_05:27:26

WORKDIR /build-stage

# Build wheels for packages that requires gcc and other build dependencies and
# lack wheels either for amd64 or aarch64. If you find a dependency here no
# longer listed in requirements.txt, remove it from here as well.
#
COPY requirements.txt requirements.txt
RUN pip install build \
 && pip wheel \
        $(cat requirements.txt | grep "pycryptodomex==") \
        $(cat requirements.txt | grep "pycurl==") \
        $(cat requirements.txt | grep "ruamel-yaml-clib==")


# The final stage
# ---------------
FROM python:3.9-slim-bullseye

ARG NB_USER=jovyan
ARG NB_UID=1000
ARG HOME=/home/jovyan
ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8

RUN adduser --disabled-password \
        --gecos "Default user" \
        --uid ${NB_UID} \
        --home ${HOME} \
        --force-badname \
        ${NB_USER}

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        # misc network utilities
        curl \
        dnsutils \
        git \
        # misc other utilities
        less \
        vim \
        # requirement for pycurl
        libcurl4 \
        # requirement for using a local sqlite database
        sqlite3 \
 && rm -rf /var/lib/apt/lists/*

RUN if [ "$(uname -m)" = x86_64 ]; then ARCH=amd64; fi; \
    if [ "$(uname -m)" = aarch64 ]; then ARCH=arm64; fi; \
    curl -sSLo /tini "https://github.com/krallin/tini/releases/download/v0.19.0/tini-$ARCH" \
 && chmod +x /tini

COPY --from=build-stage /build-stage/*.whl /tmp/pre-built-wheels/
COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir \
        /tmp/pre-built-wheels/*.whl \
        -r /tmp/requirements.txt

WORKDIR /srv/jupyterhub
RUN chown ${NB_USER}:${NB_USER} /srv/jupyterhub
USER ${NB_USER}

EXPOSE 8081
ENTRYPOINT ["/tini", "--"]
CMD ["jupyterhub", "--config", "/usr/local/etc/jupyterhub/jupyterhub_config.py"]
