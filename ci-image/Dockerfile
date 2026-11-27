FROM --platform=$BUILDPLATFORM golang:1.27.3-bookworm@sha256:972b4110ba0f47539a975c31e52c64765e1ff19f2d74469ae4a595c6676e80a5 AS gitleaks
# renovate: datasource=github-releases depName=gitleaks/gitleaks
ARG GITLEAKS_VERSION=v8.30.1
ARG TARGETOS TARGETARCH
RUN git clone --depth 1 --branch "${GITLEAKS_VERSION}" https://github.com/gitleaks/gitleaks.git /src
WORKDIR /src
RUN go get github.com/klauspost/compress@v1.20.2 golang.org/x/crypto@v0.58.0 golang.org/x/text@v0.43.0 github.com/andybalholm/brotli@v1.2.8 && \
    CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -trimpath \
      -ldflags "-s -w -X github.com/zricethezav/gitleaks/v8/version.Version=${GITLEAKS_VERSION}" \
      -o /out/gitleaks .

FROM ubuntu:26.04@sha256:d66b366dac61b575bee2aef07e9c26e8070096fbe84a3d4d3eb0da069b5d76db

ARG TARGETARCH

# renovate: datasource=github-releases depName=google/go-containerregistry
ARG CRANE_VERSION=v0.22.1
# renovate: datasource=github-releases depName=google/osv-scanner
ARG OSV_SCANNER_VERSION=v2.7.1
# renovate: datasource=github-releases depName=kubernetes/kubernetes
ARG KUBECTL_VERSION=v1.37.2
# renovate: datasource=github-releases depName=kubernetes-sigs/kustomize extractVersion=kustomize/(?<version>.+)
ARG KUSTOMIZE_VERSION=v5.8.3
# renovate: datasource=github-releases depName=argoproj/argo-cd
ARG ARGOCD_VERSION=v3.6.2
# renovate: datasource=github-releases depName=dag-andersen/argocd-diff-preview
ARG ARGOCD_DIFF_VERSION=v0.2.18
# renovate: datasource=github-releases depName=cli/cli
ARG GH_CLI_VERSION=2.105.0
# renovate: datasource=github-releases depName=mikefarah/yq
ARG YQ_VERSION=v4.54.3
# renovate: datasource=github-tags depName=docker/cli extractVersion=^v(?<version>.+)$
ARG DOCKER_VERSION=29.10.1
# renovate: datasource=github-releases depName=docker/buildx
ARG BUILDX_VERSION=v0.39.1
# renovate: datasource=github-releases depName=sigstore/cosign
ARG COSIGN_VERSION=v3.1.4
# renovate: datasource=github-releases depName=aquasecurity/trivy
ARG TRIVY_VERSION=v0.76.2

RUN apt-get update && apt-get upgrade -y && apt-get install -y --no-install-recommends \
      ca-certificates curl git jq make python3 && \
    rm -rf /var/lib/apt/lists/*

# crane (release uses x86_64 / arm64)
RUN CRANE_ARCH=$([ "$TARGETARCH" = "amd64" ] && echo "x86_64" || echo "$TARGETARCH") && \
    curl -fsSL "https://github.com/google/go-containerregistry/releases/download/${CRANE_VERSION}/go-containerregistry_Linux_${CRANE_ARCH}.tar.gz" \
      | tar -xz -C /usr/local/bin crane && chmod +x /usr/local/bin/crane

# osv-scanner
RUN curl -fsSL -o /usr/local/bin/osv-scanner \
      "https://github.com/google/osv-scanner/releases/download/${OSV_SCANNER_VERSION}/osv-scanner_linux_${TARGETARCH}" && \
    chmod +x /usr/local/bin/osv-scanner

# kubectl
RUN curl -fsSL -o /usr/local/bin/kubectl \
      "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" && \
    chmod +x /usr/local/bin/kubectl

# kustomize
RUN curl -fsSL "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F${KUSTOMIZE_VERSION}/kustomize_${KUSTOMIZE_VERSION}_linux_${TARGETARCH}.tar.gz" \
      | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/kustomize

# argocd CLI
RUN curl -fsSL -o /usr/local/bin/argocd \
      "https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-${TARGETARCH}" && \
    chmod +x /usr/local/bin/argocd

# argocd-diff-preview (release uses x86_64 / aarch64)
RUN DIFF_ARCH=$([ "$TARGETARCH" = "amd64" ] && echo "x86_64" || echo "aarch64") && \
    curl -fsSL "https://github.com/dag-andersen/argocd-diff-preview/releases/download/${ARGOCD_DIFF_VERSION}/argocd-diff-preview-Linux-${DIFF_ARCH}.tar.gz" \
      | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/argocd-diff-preview

# gh CLI
RUN curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_CLI_VERSION}/gh_${GH_CLI_VERSION}_linux_${TARGETARCH}.tar.gz" \
      | tar -xz --strip-components=2 -C /usr/local/bin "gh_${GH_CLI_VERSION}_linux_${TARGETARCH}/bin/gh" && \
    chmod +x /usr/local/bin/gh

# yq
RUN curl -fsSL -o /usr/local/bin/yq \
      "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${TARGETARCH}" && \
    chmod +x /usr/local/bin/yq

COPY --from=gitleaks /out/gitleaks /usr/local/bin/gitleaks

# docker CLI (client only) + buildx plugin — remote-buildkit image builds in CI
RUN DOCKER_ARCH=$([ "$TARGETARCH" = "amd64" ] && echo "x86_64" || echo "aarch64") && \
    curl -fsSL "https://download.docker.com/linux/static/stable/${DOCKER_ARCH}/docker-${DOCKER_VERSION}.tgz" \
      | tar -xz -C /usr/local/bin --strip-components=1 docker/docker && \
    chmod +x /usr/local/bin/docker && \
    mkdir -p /usr/local/lib/docker/cli-plugins && \
    curl -fsSL -o /usr/local/lib/docker/cli-plugins/docker-buildx \
      "https://github.com/docker/buildx/releases/download/${BUILDX_VERSION}/buildx-${BUILDX_VERSION}.linux-${TARGETARCH}" && \
    chmod +x /usr/local/lib/docker/cli-plugins/docker-buildx

# cosign (image signing in CI)
RUN curl -fsSL -o /usr/local/bin/cosign \
      "https://github.com/sigstore/cosign/releases/download/${COSIGN_VERSION}/cosign-linux-${TARGETARCH}" && \
    chmod +x /usr/local/bin/cosign

# trivy
RUN TRIVY_ARCH=$([ "$TARGETARCH" = "amd64" ] && echo "64bit" || echo "ARM64") && \
    TRIVY_STRIPPED=$(echo "$TRIVY_VERSION" | sed 's/^v//') && \
    curl -fsSL "https://github.com/aquasecurity/trivy/releases/download/${TRIVY_VERSION}/trivy_${TRIVY_STRIPPED}_Linux-${TRIVY_ARCH}.tar.gz" \
      | tar -xz -C /usr/local/bin trivy && chmod +x /usr/local/bin/trivy

# Non-root user for hardened ARC workflow pods
RUN groupadd -g 1001 runner && \
    useradd -u 1001 -g 1001 -m -s /bin/bash runner
USER 1001
