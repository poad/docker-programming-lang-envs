ARG LLVM_VERSION=13
ARG NIM_VERSION="1.4.8"

FROM alpine:latest AS downloader

ARG NIM_VERSION

WORKDIR /tmp

RUN apk --update add --no-cache --virtual .build-deps ca-certificates gnupg curl \
 && curl -sL https://apt.llvm.org/llvm-snapshot.gpg.key > llvm-snapshot.gpg.key \
 && curl -sL "https://nim-lang.org/download/nim-${NIM_VERSION}.tar.xz" > nim-${NIM_VERSION}.tar.xz \
 && tar xf nim-${NIM_VERSION}.tar.xz

RUN ls

FROM poad/docker-zsh:buster

LABEL maintainer="Kenji Saito<ken-yo@mbr.nifty.com>"

ARG LLVM_VERSION
ARG NIM_VERSION

WORKDIR /home/zsh

COPY --from=downloader /tmp/llvm-snapshot.gpg.key /tmp/llvm-snapshot.gpg.key
COPY --chown=zsh:zsh --from=downloader /tmp/nim-${NIM_VERSION} /tmp/nim-${NIM_VERSION}

USER root

RUN apt-get update -qq \
 && apt-get install --no-install-recommends -qqy ca-certificates gnupg2 binutils apt-utils libssl-dev librocksdb-dev git \
 && cat /tmp/llvm-snapshot.gpg.key | apt-key add - \
 && echo "deb http://apt.llvm.org/buster/ llvm-toolchain-buster-${LLVM_VERSION} main" >> /etc/apt/sources.list.d/llvm-toolchain.list \
 && apt-get update -qq \
 && apt-get install -qqy --no-install-recommends llvm-${LLVM_VERSION} clang-${LLVM_VERSION} lld-${LLVM_VERSION} git make \
 && apt-get autoremove --purge -qqy \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* /var/log/apt/* /var/log/alternatives.log /var/log/dpkg.log /var/log/faillog /var/log/lastlog

ENV CC=clang-${LLVM_VERSION} \
    CXX=clang++-${LLVM_VERSION}

RUN ln -s /usr/bin/clang-${LLVM_VERSION} /usr/bin/gcc

HEALTHCHECK CMD [ "nimble", "-version" ]

USER zsh

WORKDIR /tmp/nim-${NIM_VERSION}

RUN chmod 744 build.sh \
 && ./build.sh \
 && bin/nim c koch \
 && ./koch tools

USER root

RUN ./install.sh /usr/bin

USER zsh

ENV PATH=/usr/bin/nim/bin:${PATH}
