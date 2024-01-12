# argocd-helm-quay

## Installation
- Quay
- Openshift GitOps

## Demo

### Upload helm chart to quay
- Create user redhatredhat in quay
- Create shop organization
- (this is not necessary) podman login https://example-registry-quay-quay-enterprise.apps.cluster-zv2sq.zv2sq.sandbox1370.opentlc.com -u redhatredhat -p redhatredhat
- cd argocd-helm-quay/helm/quarkus-helm-base/chart
- export HELM_EXPERIMENTAL_OCI=1
- helm push quarkus-base-0.1.0.tgz oci://example-registry-quay-quay-enterprise.apps.cluster-zv2sq.zv2sq.sandbox1370.opentlc.com/shop
- cd ../../quarkus-helm-networking/chart/
- helm push quarkus-base-networking-0.1.0.tgz oci://example-registry-quay-quay-enterprise.apps.cluster-zv2sq.zv2sq.sandbox1370.opentlc.com/shop

Useful Documentation: https://www.redhat.com/en/blog/quay-oci-artifact-support-for-helm-charts



### Configure quay as Argo CD repo

It seems that organization have to be added /shop

´´´
apiVersion: v1
kind: Secret
metadata:
  name: quay-private-repo
  namespace: openshift-gitops
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: helm
  url: example-registry-quay-quay-enterprise.apps.cluster-zv2sq.zv2sq.sandbox1370.opentlc.com/shop
  password: redhatredhat
  username: redhatredhat
  enableOCI: "true"
  name: quay
´´´

### Deploy chart from quay

- create test proyect

´´´
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
     argocd.argoproj.io/managed-by: openshift-gitops
spec:
  finalizers:
  - kubernetes
´´´

- Create argocd application
´´´
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: test-quarkus
  namespace: openshift-gitops
spec:
  destination:
    namespace: test
    server: 'https://kubernetes.default.svc'
  source:
    repoURL: example-registry-quay-quay-enterprise.apps.cluster-zv2sq.zv2sq.sandbox1370.opentlc.com/shop
    chart: quarkus-base
    targetRevision: 0.1.0
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
´´´

### Deploy chart with a dependency from quay


- Create argocd application
´´´
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: shop
  namespace: openshift-gitops
spec:
  destination:
    namespace: test
    server: 'https://kubernetes.default.svc'
  source:
    path: helm/quarkus-helm-umbrella/chart
    repoURL:  https://github.com/davidseve/argocd-helm-quay.git
    targetRevision: HEAD
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
´´´


