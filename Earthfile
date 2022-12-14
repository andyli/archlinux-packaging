VERSION 0.6
FROM archlinux:base-devel
ARG DEVCONTAINER_IMAGE_NAME_DEFAULT=ghcr.io/andyli/archlinux_packaging_devcontainer

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG TARGETARCH

devcontainer-base:
    RUN pacman -Sy --noconfirm \
        curl \
        wget \
        ca-certificates \
        python \
        gzip \
        tar \
        git \
        svn \
        jq \
        vim \
        sudo \
        socat \
        docker \
        docker-compose \
        direnv \
        bash-completion

    # create user
    RUN groupadd --gid $USER_GID $USERNAME \
        && useradd -s /bin/bash --uid $USER_UID --gid $USERNAME -m $USERNAME \
        && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
        && chmod 0440 /etc/sudoers.d/$USERNAME \
        && usermod -G docker -a $USERNAME

    # Setting the ENTRYPOINT to docker-init.sh will configure non-root access 
    # to the Docker socket. The script will also execute CMD as needed.
    COPY .devcontainer/docker-init.sh /usr/local/share/
    ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
    CMD [ "sleep", "infinity" ]

    RUN mkdir -m 777 "/workspace"
    WORKDIR /workspace

# Usage:
# COPY +earthly/earthly /usr/local/bin/
# RUN earthly bootstrap --no-buildkit --with-autocomplete
earthly:
    FROM +devcontainer-base
    RUN curl -fsSL https://github.com/earthly/earthly/releases/download/v0.6.22/earthly-linux-${TARGETARCH} -o /usr/local/bin/earthly \
        && chmod +x /usr/local/bin/earthly
    SAVE ARTIFACT /usr/local/bin/earthly

devcontainer:
    FROM +devcontainer-base

    # Install earthly
    COPY +earthly/earthly /usr/local/bin/
    RUN earthly bootstrap --no-buildkit --with-autocomplete

    USER $USERNAME

    # Config direnv
    COPY --chown=$USER_UID:$USER_GID .devcontainer/direnv.toml /home/$USERNAME/.config/direnv/config.toml
    RUN echo 'eval "$(direnv hook bash)"' >> ~/.bashrc

    USER root

    ARG GIT_SHA
    ENV GIT_SHA="$GIT_SHA"
    ARG IMAGE_NAME="$DEVCONTAINER_IMAGE_NAME_DEFAULT"
    ARG IMAGE_TAG="master"
    ARG IMAGE_CACHE="$IMAGE_NAME:$IMAGE_TAG"
    SAVE IMAGE --cache-from="$IMAGE_CACHE" --push "$IMAGE_NAME:$IMAGE_TAG"

ci-images:
    ARG --required GIT_REF_NAME
    ARG --required GIT_SHA
    BUILD +devcontainer \ 
        --IMAGE_CACHE="$DEVCONTAINER_IMAGE_NAME_DEFAULT:$GIT_REF_NAME" \
        --IMAGE_TAG="$GIT_REF_NAME" \
        --IMAGE_TAG="$GIT_SHA" \
        --GIT_SHA="$GIT_SHA"
