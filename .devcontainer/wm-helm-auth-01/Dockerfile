ARG __from_image=ubuntu:24.04

FROM alpine:3 as oc-builder

ARG __go_ver=1.23.4
ARG __go_father_dir=/usr/local
ARG __oc_source_release_tag=release-4.20

# trunk-ignore(hadolint/DL3018)
RUN apk --no-cache update \
    && apk --no-cache upgrade \
    && apk --no-cache add \
      curl \
      git \
      go \
      gpgme \
      heimdal \
      heimdal-dev \
      linux-headers \
      make \
    && curl https://go.dev/dl/go${__go_ver}.linux-amd64.tar.gz -L -o /tmp/go.tgz \
    && tar -C "${__go_father_dir}" -zxf /tmp/go.tgz \
    && rm -f /tmp/go.tgz

ENV PATH ${__go_father_dir}/go/bin:$PATH

RUN git clone -b ${__oc_source_release_tag} --depth 1 https://github.com/openshift/oc.git /tmp/oc
WORKDIR /tmp/oc
RUN make

# trunk-ignore(checkov/CKV_DOCKER_7,hadolint/DL3006)
FROM ${__from_image}

# trunk-ignore(hadolint/DL4006,hadolint/DL3008)
RUN apt-get -qy update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -qy install --no-install-recommends \
        ca-certificates \
        curl \
        gcc \
        git \
        gpg \
        less \
        lsb-release \
        make \
        ssh \
    && curl https://get.trunk.io -fsSL | bash \
    && rm -rf /var/lib/apt/lists/*

ARG __gid=756
ARG __gname=wmhelm
ARG __uid=756
ARG __uname=wmhelm
ARG __uhome=/home/${__uname}
ARG __workspace_folder=/workspace/project
ARG __musl_version=1.2.5

RUN groupadd -g ${__gid} ${__gname} \
    && useradd -l -rm -d  ${__uhome} -s /bin/bash -g ${__gid} -u ${__uid} ${__uname} \
    && mkdir -p ${__workspace_folder} \
    && chown ${__uname}:${__gname} ${__workspace_folder} \
    && chown -R ${__uname}:${__gname} /usr/local/bin

# Our build of oc requires musl to work
# TODO: investigate if it can be taken from the previous stage instead of being built from source here
# trunk-ignore(hadolint/DL3003)
RUN curl https://musl.libc.org/releases/musl-${__musl_version}.tar.gz -o /tmp/musl.tgz \
    && mkdir -p /tmp/musl-builder \
    && tar -C /tmp/musl-builder -xvf /tmp/musl.tgz \
    && cd /tmp/musl-builder/musl-${__musl_version} \
    && ./configure && make && make install \
    && ln -s /usr/lib64/libc.so.6 /usr/lib64/libc.musl-x86_64.so.1 \
    && rm -rf /tmp/musl*

COPY --chown=${__uid}:${__gid} --from=oc-builder /tmp/oc/oc /usr/local/bin/oc

# We also need kubectl
ARG __kube_version=v1.32

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/${__kube_version}/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg \
    && chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${__kube_version}/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list \
    && chmod 644 /etc/apt/sources.list.d/kubernetes.list \
    && apt-get -qy update \
    && apt-get -qy install --no-install-recommends kubectl=1.* \
    && rm -rf /var/lib/apt/lists/*

USER ${__uname}

HEALTHCHECK --interval=60s --timeout=60s --start-period=25s --retries=3  CMD [ exit 0 ]
