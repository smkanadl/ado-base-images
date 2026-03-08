# escape=`

FROM ubuntu:18.04

LABEL org.opencontainers.image.source="https://github.com/smkanadl/ado-base-images"

# https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux#nonglibc-based-containers
# Use official alpine node images as code ref. Cannot use as base as they have an entrypoint
# https://github.com/nodejs/docker-node/blob/main/20/bullseye-slim/Dockerfile

LABEL "com.azure.dev.pipelines.agent.handler.node.path"="/usr/local/bin/node"

# Build node from source
RUN groupadd --gid 1000 node `
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node

# Install gcc 12 to compile a proper Python-3 and then NodeJS 24
# https://launchpad.net/~savoury1/+archive/ubuntu/gcc-defaults-12
RUN apt-get update && apt install software-properties-common -y `
	&& add-apt-repository ppa:savoury1/gcc-defaults-12 `
	&& apt-get update `
	&& apt-get upgrade -y && apt-get dist-upgrade -y

# Install python3.9 which is needed to build node24 from source
# https://gist.github.com/basaks/652eea861a143a9b3d11805c96273488
RUN apt update && DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends build-essential checkinstall libreadline-gplv2-dev `
	libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev `
	libbz2-dev openssl libffi-dev curl`
	# Fix curl cert issues https://askubuntu.com/questions/1390288/curl-77-error-setting-certificate-verify-locations-ubuntu-20-04-3-lts
	&& rm -f /etc/ssl/certs/ca-bundle.crt `
	&& apt install ca-certificates `
	&& update-ca-certificates `
	# Build python
	&& mkdir -p $HOME/opt `
	&& cd $HOME/opt `
	&& curl -O https://www.python.org/ftp/python/3.9.2/Python-3.9.2.tgz `
	&& tar -xzf Python-3.9.2.tgz `
	&& cd Python-3.9.2/ `
    && ./configure --enable-shared --enable-optimizations --prefix=/usr/local LDFLAGS="-Wl,--rpath=/usr/local/lib" `
    && make altinstall `
    && cd .. `
    && rm -rf Python-3.9.2 `
    && rm Python-3.9.2.tgz `
	# smoke test
	&& python3.9 -V

ENV NODE_VERSION=24.14.0

RUN ARCH= OPENSSL_ARCH= && dpkgArch="$(dpkg --print-architecture)" `
    && case "${dpkgArch##*-}" in `
      amd64) ARCH='x64' OPENSSL_ARCH='linux-x86_64';; `
      ppc64el) ARCH='ppc64le' OPENSSL_ARCH='linux-ppc64le';; `
      s390x) ARCH='s390x' OPENSSL_ARCH='linux*-s390x';; `
      arm64) ARCH='arm64' OPENSSL_ARCH='linux-aarch64';; `
      armhf) ARCH='armv7l' OPENSSL_ARCH='linux-armv4';; `
      i386) ARCH='x86' OPENSSL_ARCH='linux-elf';; `
      *) echo "unsupported architecture"; exit 1 ;; `
    esac `
    && set -ex `
    # libatomic1 for arm
    && apt-get update && apt-get install -y ca-certificates curl wget gnupg dirmngr xz-utils libatomic1 --no-install-recommends `
    # use pre-existing gpg directory, see https://github.com/nodejs/docker-node/pull/1895#issuecomment-1550389150
    && export GNUPGHOME="$(mktemp -d)" `
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && for key in `
      5BE8A3F6C8A5C01D106C0AD820B1A390B168D356 `
      DD792F5973C6DE52C432CBDAC77ABFA00DDBF2B7 `
      CC68F5A3106FF448322E48ED27F5E38D5B0A215F `
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 `
      890C08DB8579162FEE0DF9DB8BEAB4DFCF555EF4 `
      C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C `
      108F52B48DB57BB0CC439B2997B01419BD92F80A `
      A363A499291CBBC940DD62E41F10027AF002F8B0 `
    ; do `
      gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$key" || `
      gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" ; `
    done `
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" `
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" `
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc `
    && gpgconf --kill all `
    && rm -rf "$GNUPGHOME" `
    && grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - `
    && tar -xJf "node-v$NODE_VERSION.tar.xz" `
    && cd "node-v$NODE_VERSION" `
    && ./configure `
    && make -j$(nproc) `
    && make install `
    && cd .. `
    && rm -rf "node-v$NODE_VERSION" `
    && rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt `
	# Remove unused OpenSSL headers to save ~34MB. See this NodeJS issue: https://github.com/nodejs/node/issues/46451
    && find /usr/local/include/node/openssl/archs -mindepth 1 -maxdepth 1 ! -name "$OPENSSL_ARCH" -exec rm -rf {} \; `
    && apt-mark auto '.*' > /dev/null `
    # After some testing, just do a plain autoremove. Keep the ppa:savoury1/gcc-defaults-12 so the 
	&& apt-get autoremove -y `
    # smoke tests
    && node --version `
    && npm --version

# Install core requirements (after node install to ensure those packages are not purged)
RUN apt-get update && `
	apt-get install -y `
    git `
    bash `
    sudo `
    && rm -rf /var/lib/apt/lists/*
