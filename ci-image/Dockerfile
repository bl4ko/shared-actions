FROM ubuntu:24.04@sha256:0d39fcc8335d6d74d5502f6df2d30119ff4790ebbb60b364818d5112d9e3e932

ARG TARGETARCH

# renovate: datasource=github-releases depName=google/go-containerregistry
ARG CRANE_VERSION=v0.21.3
# renovate: datasource=github-releases depName=google/osv-scanner
ARG OSV_SCANNER_VERSION=v2.3.3
# renovate: datasource=github-releases depName=kubernetes/kubernetes
ARG KUBECTL_VERSION=v1.35.2
# renovate: datasource=github-releases depName=kubernetes-sigs/kustomize extractVersion=kustomize/(?<version>.+)
ARG KUSTOMIZE_VERSION=v5.8.1
# renovate: datasource=github-releases depName=argoproj/argo-cd
ARG ARGOCD_VERSION=v3.3.4
# renovate: datasource=github-releases depName=dag-andersen/argocd-diff-preview
ARG ARGOCD_DIFF_VERSION=v0.2.2
# renovate: datasource=github-releases depName=cli/cli
ARG GH_CLI_VERSION=2.88.1
# renovate: datasource=github-releases depName=mikefarah/yq
ARG YQ_VERSION=v4.52.4
# renovate: datasource=github-releases depName=gitleaks/gitleaks
ARG GITLEAKS_VERSION=v8.30.0

RUN apt-get update && apt-get install -y --no-install-recommends \
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

# gitleaks (release uses x64 / arm64)
RUN GITLEAKS_ARCH=$([ "$TARGETARCH" = "amd64" ] && echo "x64" || echo "$TARGETARCH") && \
    GITLEAKS_STRIPPED=$(echo "$GITLEAKS_VERSION" | sed 's/^v//') && \
    curl -fsSL "https://github.com/gitleaks/gitleaks/releases/download/${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_STRIPPED}_linux_${GITLEAKS_ARCH}.tar.gz" \
      | tar -xz -C /usr/local/bin gitleaks && chmod +x /usr/local/bin/gitleaks

# Non-root user for hardened ARC workflow pods
RUN groupadd -g 1001 runner && \
    useradd -u 1001 -g 1001 -m -s /bin/bash runner
USER 1001
