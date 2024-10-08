# Build arguments
ARG SOURCE_CODE=.
ARG CI_CONTAINER_VERSION="unknown"
ARG GOLANG_VERSION=1.21

# Use ubi8/go-toolset as base image
FROM registry.access.redhat.com/ubi8/go-toolset:1.21 as builder
ARG TARGETOS
ARG TARGETARCH

ARG SOURCE_CODE

COPY go.mod go.mod
COPY go.sum go.sum

RUN go mod download

# Copy the go source
COPY main.go main.go
COPY api/ api/

WORKDIR /workspace

# Copy the Go Modules manifests
COPY notebook-controller /workspace/notebook-controller
COPY common /workspace/common

# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer
RUN cd /workspace/notebook-controller && go mod download

WORKDIR /workspace/notebook-controller

# Build
RUN CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -tags strictfipsruntime  -a -mod=mod -o manager main.go

# Use ubi8/ubi-minimal as base image
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest

## Build args to be used at this step
ARG CI_CONTAINER_VERSION

## Install additional packages
#RUN microdnf install -y shadow-utils &&\
 #   microdnf clean all

## Create a non-root user with UID 1001
RUN useradd --uid 1001 --create-home --user-group --system rhods

WORKDIR /home/rhods

COPY --from=builder /workspace/notebook-controller/manager .
COPY --from=builder /workspace/notebook-controller/third_party/license.txt third_party/license.txt
# COPY --from=builder /go/pkg/mod/github.com/hashicorp third_party/hashicorp
ENTRYPOINT ["/manager"]

LABEL com.redhat.component="odh-kf-notebook-controller-container" \
      name="managed-open-data-hub/odh-kf-notebook-controller-rhel8" \
      version="${CI_CONTAINER_VERSION}" \
      git.url="${CI_KUBEFLOW_UPSTREAM_URL}" \
      git.commit="${CI_KUBEFLOW_UPSTREAM_COMMIT}" \
      summary="odh-kf-notebook-controller" \
      io.openshift.expose-services="" \
      io.k8s.display-name="odh-kf-notebook-controller" \
      maintainer="['managed-open-data-hub@redhat.com']" \
      description="odh-kf-notebook-controller" \
      com.redhat.license_terms="https://www.redhat.com/licenses/Red_Hat_Standard_EULA_20191108.pdf"

