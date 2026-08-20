FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    bash \
    coreutils \
    sudo \
    curl \
    wget \
    git \
    vim \
    nano \
    less \
    grep \
    sed \
    gawk \
    tar \
    gzip \
    unzip \
    procps \
    net-tools \
    iputils-ping \
    ca-certificates \
    locales \
    tzdata \
    && rm -rf /var/lib/apt/lists/*

# Generate a standard locale so things like sorting/text output behave normally
RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

WORKDIR /src

CMD ["bash"]
