ARG LLVM_VERSION=18

FROM alpine:latest AS downloader

RUN apk --update add --no-cache --virtual .build-deps ca-certificates gnupg curl \
 && curl -sL https://apt.llvm.org/llvm-snapshot.gpg.key > /tmp/llvm-snapshot.gpg.key \
 && curl -sL "https://keybase.io/crystal/pgp_keys.asc" > /tmp/pgp_keys.asc

FROM poad/docker-zsh:latest AS base

LABEL maintainer="Kenji Saito<ken-yo@mbr.nifty.com>"

ARG LLVM_VERSION

WORKDIR /home/zsh

COPY --chown=zsh:zsh shard.yml /home/zsh/shard.yml 

COPY --from=downloader /tmp/llvm-snapshot.gpg.key /tmp/llvm-snapshot.gpg.key
COPY --from=downloader /tmp/pgp_keys.asc /tmp/pgp_keys.asc

USER root

RUN apt-get update -qq \
 && apt-get install --no-install-recommends -qqy ca-certificates gnupg2 binutils apt-utils software-properties-common \
 && cat /tmp/llvm-snapshot.gpg.key | apt-key add - \
 && cat /tmp/pgp_keys.asc | apt-key add - \
 && add-apt-repository "deb http://ppa.launchpad.net/git-core/ppa/ubuntu bionic main" -y \
 && echo "deb http://apt.llvm.org/buster/ llvm-toolchain-buster-${LLVM_VERSION} main" >> /etc/apt/sources.list.d/llvm-toolchain.list \
 && echo "deb https://dist.crystal-lang.org/apt crystal main" >> /etc/apt/sources.list.d/crystal.list \
 && apt-get update -qq \
 && apt-get install -qqy --no-install-recommends libssl-dev libxml2-dev libyaml-dev libgmp-dev libreadline-dev libz-dev git crystal \
 && shards install \
 && apt-get autoremove --purge -qqy \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/log/apt/* /var/log/alternatives.log /var/log/dpkg.log /var/log/faillog /var/log/lastlog

HEALTHCHECK CMD [ "crystal", "-version" ]

USER zsh

EXPOSE 8080

CMD [ "crystal", "play", "-b", "0.0.0.0" ]
