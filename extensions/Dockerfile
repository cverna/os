## Downloads the extensions given the extensions.yaml
FROM registry.ci.openshift.org/rhcos-devel/rhel-coreos:latest as os
# Expects os to be cloned and this build run from the top level dir like:
# podman build -f extensions/Dockerfile .
# also expects submodules to be initialized
RUN mkdir /os
WORKDIR /os
ADD . .
ARG COSA
RUN if [[ -z "$COSA" ]] ; then ci/get-ocp-repo.sh ; fi
RUN rpm-ostree compose extensions --rootfs=/ --output-dir=/usr/share/rpm-ostree/extensions/ {manifest,extensions}.yaml

## Creates the repo metadata for the extensions & builds the go binary
FROM quay.io/centos/centos:stream8 as builder
COPY --from=os /usr/share/rpm-ostree/extensions/ /usr/share/rpm-ostree/extensions/
RUN dnf install -y createrepo_c golang
ADD extensions/repo-server/main.go .
RUN mkdir /build
RUN go build -o /build/webserver main.go
RUN createrepo_c /usr/share/rpm-ostree/extensions/

## Final container that has the extensions and webserver
FROM registry.access.redhat.com/ubi8/ubi:latest
COPY --from=builder /build/webserver /usr/bin/webserver
COPY --from=builder /usr/share/rpm-ostree/extensions/ /usr/share/rpm-ostree/extensions/
CMD ["./usr/bin/webserver"]
EXPOSE 9091/tcp
