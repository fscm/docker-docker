# global args
ARG __BUILD_DIR__="/build"
ARG __DATA_DIR__="/data"



FROM fscm/centos:stream as build

ARG __BUILD_DIR__
ARG __DATA_DIR__
ARG DOCKER_CHANNEL="stable"
ARG DOCKER_VERSION="20.10.8"
ARG __USER__="root"
ARG __WORK_DIR__="/work"

ENV \
  LANG="C.UTF-8" \
  LC_ALL="C.UTF-8"

USER ${__USER__}

#COPY --chown="${__USER__}":${__USER__} "LICENSE" "files/" "${__WORK_DIR__}/"
COPY "LICENSE" "files/" "${__WORK_DIR__}"/
COPY --from=fscm/centos:stream "/" "${__BUILD_DIR__}"/

WORKDIR "${__WORK_DIR__}"

SHELL ["/bin/bash", "-c"]

RUN \
# build env
  echo '--> setting build env' && \
  set +h && \
# build structure
  echo '--> creating build structure' && \
  for folder in /usr/local/bin /usr/bin /licenses; do install --directory --owner=${__USER__} --group=${__USER__} --mode=0755 "${__BUILD_DIR__}${folder}"; done && \
  for folder in /tmp "${__DATA_DIR__}"; do install --directory --owner=${__USER__} --group=${__USER__} --mode=1777 "${__BUILD_DIR__}${folder}"; done && \
# dependencies
  echo '--> instaling dependencies' && \
  dnf --assumeyes --quiet --setopt=install_weak_deps='no' install \
    binutils \
    ca-certificates \
    curl \
    findutils \
    gzip \
    tar && \
# copy tests
  echo '--> copying test files' && \
  install --owner="${__USER__}" --group="${__USER__}" --mode=0755 --target-directory="${__BUILD_DIR__}/usr/bin" "${__WORK_DIR__}/tests"/* && \
# copy scripts
  echo '--> copying scripts' && \
  install --owner="${__USER__}" --group="${__USER__}" --mode=0755 --target-directory="${__BUILD_DIR__}/usr/bin" "${__WORK_DIR__}/scripts"/* && \
# docker dependencies
  echo '--> installing docker dependencies' && \
  cp "/etc/resolv.conf" "${__BUILD_DIR__}/etc/" && \
  chroot "${__BUILD_DIR__}" dnf --assumeyes --quiet --disablerepo='*' --enablerepo='baseos' --setopt=tsflags='nodocs' --setopt=install_weak_deps='no' install \
    ca-certificates \
    device-mapper-libs \
    e2fsprogs \
    e2fsprogs-libs \
    gzip \
    iproute \
    iptables \
    kmod \
    libcgroup \
    libseccomp \
    openssl \
    openssl-libs \
    tar \
    xfsprogs \
    xz && \
  chroot "${__BUILD_DIR__}" dnf --quiet clean all && \
# docker
  echo '--> installing docker' && \
  curl --silent --location --retry 3 "https://download.docker.com/linux/static/${DOCKER_CHANNEL}/$(arch)/docker-${DOCKER_VERSION}.tgz" \
    | tar --extract --gunzip --no-same-owner --strip-components=1 --directory="${__BUILD_DIR__}/usr/local/bin" && \
  install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}/licenses/docker" && \
  curl --silent --location --retry 3 "https://raw.githubusercontent.com/docker/engine/master/LICENSE" \
    --output "${__BUILD_DIR__}/licenses/docker/LICENSE" && \
# docker-compose
  echo '--> installing docker-compose' && \
  DOCKER_COMPOSE_VERSION="$(curl --silent --location --retry 3 'https://api.github.com/repos/docker/compose/releases/latest' | sed -n -e '/tag_name/ s/.*"\(.*\)",/\1/p')" && \
  curl --silent --location --retry 3 "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-Linux-x86_64" \
    --output "${__BUILD_DIR__}/usr/local/bin/docker-compose" && \
  chmod 0755 "${__BUILD_DIR__}/usr/local/bin/docker-compose" && \
  install --directory --owner="${__USER__}" --group="${__USER__}" --mode=0755 "${__BUILD_DIR__}/licenses/docker-compose" && \
  curl --silent --location --retry 3 "https://raw.githubusercontent.com/docker/compose/master/LICENSE" \
    --output "${__BUILD_DIR__}/licenses/docker-compose/LICENSE" && \
# stripping
  echo '--> stripping libraries and binaries' && \
  for folder in lib lib64; do \
    find "${__BUILD_DIR__}/usr/${folder}" -type f -name '*.a' -exec strip --strip-debug {} ';' ; \
    find "${__BUILD_DIR__}/usr/${folder}" -type f -name '*.so*' -exec strip --strip-unneeded {} ';' ; \
  done && \
  for folder in bin sbin; do \
    find "${__BUILD_DIR__}/usr/local/${folder}" -type f -not -links +1 -exec strip --strip-all {} ';' ; \
  done && \
# system settings / cleanup
  echo '--> system settings / cleaning up' && \
  rm -f > "${__BUILD_DIR__}/etc/resolv.conf" && \
  echo 'container' > "${__BUILD_DIR__}/etc/dnf/vars/infra" && \
  echo '%_install_langs C.utf8' > "${__BUILD_DIR__}/etc/rpm/macros.image-language-conf" && \
  :> "${__BUILD_DIR__}/etc/machine-id" && \
  find "${__BUILD_DIR__}/lib/systemd/system" -path '*/sysinit.target.wants/*' \( -type l -o -type f \) -not -name 'systemd-tmpfiles-setup.service' -delete && \
  find "${__BUILD_DIR__}/etc" -path '*/systemd/system/*.wants/*' \( -type l -o -type f \) -delete && \
  rm -f "${__BUILD_DIR__}/lib/systemd/system/anaconda.target.wants"/* && \
  rm -f "${__BUILD_DIR__}/lib/systemd/system/basic.target.wants"/* && \
  rm -f "${__BUILD_DIR__}/lib/systemd/system/local-fs.target.wants"/* && \
  rm -f "${__BUILD_DIR__}/lib/systemd/system/multi-user.target.wants"/* && \
  rm -f "${__BUILD_DIR__}/lib/systemd/system/sockets.target.wants"/*udev* && \
  rm -f "${__BUILD_DIR__}/lib/systemd/system/sockets.target.wants"/*initctl* && \
  if [ -x "${__BUILD_DIR__}/usr/bin/systemctl" ]; then chroot "${__BUILD_DIR__}/" systemctl mask systemd-logind.service getty.target console-getty.service sys-fs-fuse-connections.mount systemd-remount-fs.service dev-hugepages.mount; fi && \
  rm -rf "${__BUILD_DIR__}/usr/share/info"/* && \
  rm -rf "${__BUILD_DIR__}/usr/share/man"/* && \
  rm -rf "${__BUILD_DIR__}/var/cache/yum"/* && \
  rm -rf "${__BUILD_DIR__}/var/lib/dnf"/history.* && \
  rm -rf "${__BUILD_DIR__}/var/log"/* && \
  rm -rf "${__BUILD_DIR__}/sbin/sln" && \
  rm -rf "${__BUILD_DIR__}/dev"/.??* && \
  rm -rf "${__BUILD_DIR__}/home"/.??* && \
  rm -rf "${__BUILD_DIR__}/root"/.??* && \
  rm -rf "${__BUILD_DIR__}/tmp"/.??* && \
  rm -rf "${__BUILD_DIR__}/boot" && \
  rm -rf "${__BUILD_DIR__}/proc" && \
  rm -rf "${__BUILD_DIR__}/sys" && \
  find "${__BUILD_DIR__}/usr/share/i18n/locales" -mindepth 1 -maxdepth 1 -not -name 'en_US' -exec rm -r {} + && \
  find "${__BUILD_DIR__}/usr/lib64/gconv" -mindepth 1 -maxdepth 1 -type f -not -name 'UTF*' -delete && \
  find "${__BUILD_DIR__}/usr/share/locale" -mindepth 1 -maxdepth 1 -type d -not \( -name 'en' -o -name 'en_US' \) -exec rm -r {} + && \
  find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -not -type d -not \( -name 'COPYING*' -o -name 'GPL' -o -name '*LICENSE*' \) -delete && \
  find "${__BUILD_DIR__}/usr/share/doc" -mindepth 1 -type d -empty -delete && \
  find "${__BUILD_DIR__}/var/cache" -type f -delete && \
  find "${__BUILD_DIR__}/etc/yum.repos.d" \( -iname '*debuginfo*' -o -iname '*media*' \) -type f -delete && \
# licenses
  echo '--> project licenses' && \
  install --owner=${__USER__} --group=${__USER__} --mode=0644 --target-directory="${__BUILD_DIR__}/licenses" "${__WORK_DIR__}/LICENSE" && \
# done
  echo '--> all done!'



FROM scratch

ARG __BUILD_DIR__
ARG __DATA_DIR__

LABEL \
  maintainer="Frederico Martins <https://hub.docker.com/u/fscm/>" \
  vendor="fscm" \
  cmd="docker container run --detach --privileged --publish 2375:2375/tcp fscm/docker:centos start" \
  params="--volume ./:${__DATA_DIR__}:rw"

ENV \
  LANG="C.UTF-8" \
  LC_ALL="C.UTF-8" \
  DATA_DIR="${__DATA_DIR__}" \
  DOCKER_TLS_CERTDIR="${__DATA_DIR__}/certs"

EXPOSE \
  2375/tcp \
  2376/tcp

COPY --from=build "${__BUILD_DIR__}" "/"

VOLUME ["${__DATA_DIR__}"]

WORKDIR "${__DATA_DIR__}"

ENTRYPOINT ["/usr/bin/entrypoint"]

CMD ["help"]
