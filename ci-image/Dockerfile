FROM golang:1.24-bookworm AS kaniko-builder

RUN git clone --depth 1 --branch v1.25.6 \
      https://github.com/chainguard-forks/kaniko.git /tmp/kaniko && \
    cd /tmp/kaniko && make out/executor

FROM ubuntu:24.04

# renovate: datasource=github-releases depName=google/go-containerregistry
ARG CRANE_VERSION=v0.21.1
# renovate: datasource=github-releases depName=google/osv-scanner
ARG OSV_SCANNER_VERSION=v2.3.3
# renovate: datasource=github-releases depName=kubernetes/kubernetes
ARG KUBECTL_VERSION=v1.35.2
# renovate: datasource=github-releases depName=kubernetes-sigs/kustomize extractVersion=kustomize/(?<version>.+)
ARG KUSTOMIZE_VERSION=v5.6.0
# renovate: datasource=github-releases depName=argoproj/argo-cd
ARG ARGOCD_VERSION=v3.3.2
# renovate: datasource=github-releases depName=dag-andersen/argocd-diff-preview
ARG ARGOCD_DIFF_VERSION=v0.1.26
# renovate: datasource=github-releases depName=cli/cli
ARG GH_CLI_VERSION=2.72.0

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates curl git jq make && \
    rm -rf /var/lib/apt/lists/*

# crane
RUN curl -fsSL "https://github.com/google/go-containerregistry/releases/download/${CRANE_VERSION}/go-containerregistry_Linux_x86_64.tar.gz" \
      | tar -xz -C /usr/local/bin crane && chmod +x /usr/local/bin/crane

# osv-scanner
RUN curl -fsSL -o /usr/local/bin/osv-scanner \
      "https://github.com/google/osv-scanner/releases/download/${OSV_SCANNER_VERSION}/osv-scanner_linux_amd64" && \
    chmod +x /usr/local/bin/osv-scanner

# kubectl
RUN curl -fsSL -o /usr/local/bin/kubectl \
      "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
    chmod +x /usr/local/bin/kubectl

# kustomize
RUN curl -fsSL "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F${KUSTOMIZE_VERSION}/kustomize_${KUSTOMIZE_VERSION}_linux_amd64.tar.gz" \
      | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/kustomize

# argocd CLI
RUN curl -fsSL -o /usr/local/bin/argocd \
      "https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-amd64" && \
    chmod +x /usr/local/bin/argocd

# argocd-diff-preview
RUN curl -fsSL "https://github.com/dag-andersen/argocd-diff-preview/releases/download/${ARGOCD_DIFF_VERSION}/argocd-diff-preview-Linux-x86_64.tar.gz" \
      | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/argocd-diff-preview

# gh CLI
RUN curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_CLI_VERSION}/gh_${GH_CLI_VERSION}_linux_amd64.tar.gz" \
      | tar -xz --strip-components=2 -C /usr/local/bin "gh_${GH_CLI_VERSION}_linux_amd64/bin/gh" && \
    chmod +x /usr/local/bin/gh

# Kaniko executor (pre-built from Chainguard fork)
COPY --from=kaniko-builder /tmp/kaniko/out/executor /usr/local/bin/executor
RUN chmod +x /usr/local/bin/executor
