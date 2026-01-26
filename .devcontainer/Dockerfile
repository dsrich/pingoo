FROM node:slim as node

FROM debian:13

ENV TZ="UTC"
RUN echo "${TZ}" > /etc/timezone

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt upgrade -y
RUN apt-get install -y --no-install-recommends \
    # base system. psmisc for killall. openssh-client for git over SSH. socat for docker
    bash curl wget psmisc ca-certificates lsb-release openssh-client socat \
    # dev tools. dnsutils for dig. bsdmainutils for hexdump.
    git vim make binutils coreutils build-essential pkg-config linux-perf ripgrep \
    zip htop wrk zstd gdb dnsutils jq b3sum gnupg libimage-exiftool-perl bsdmainutils lsof xxd net-tools iproute2 \
    # Rust toolchain
    rustup libfindbin-libs-perl lld mold gcc g++ musl musl-dev musl-tools libc6-dev cmake clang libclang-dev openssl libssl-dev \
    # useful data. mailcap for mimetypes. tzdata for timezones.
    mailcap tzdata libpcre2-dev \
    # pingoo specific dependencies
    sqlite3


# setup node
COPY --from=node /usr/local/bin/node /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
  && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

RUN npm config set ignore-scripts true
RUN /usr/local/bin/npm i -g npm@latest


# Setup dev user
ARG USERNAME=dev
ARG USER_UID=10001
ARG USER_GID=$USER_UID

RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/home/${USERNAME}" \
    --shell "/bin/bash" \
    --uid "${USER_UID}" \
    "${USERNAME}"


RUN mkdir -p /etc/pingoo
RUN chown -R $USERNAME:$USERNAME /etc/pingoo/

USER $USERNAME

COPY --chown=$USERNAME bashrc /home/$USERNAME/.bashrc
COPY --chown=$USERNAME vimrc /home/$USERNAME/.vimrc

# setup node (again, as a normal user this time)
RUN npm config set ignore-scripts true
RUN npm config set prefix '~/.npm/'

# setup git
RUN git config --global push.autoSetupRemote true
RUN git config --global init.defaultBranch main

WORKDIR /

# Setup docker's socket access from docker
USER root

RUN touch /var/run/docker-host.sock \
    && ln -s /var/run/docker-host.sock /var/run/docker.sock
COPY docker-init.sh /usr/local/share/docker-init.sh
RUN chmod +x /usr/local/share/docker-init.sh

# VS Code overrides ENTRYPOINT and CMD when executing `docker run` by default.
# Setting the ENTRYPOINT to docker-init.sh will configure non-root access to
# the Docker socket if "overrideCommand": false is set in devcontainer.json.
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]


EXPOSE 8080 8081 8082 8083 8443

# Install AI tooling
# https://code.claude.com/docs/en/setup
# https://developers.openai.com/codex/cli
# https://github.com/mistralai/mistral-vibe
