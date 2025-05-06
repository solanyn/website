# Deploying the Kubeflow platform

### Preamble

I recently started contributing to the Kubeflow open-source project and have been following some recent GitHub discussions, watched working group (WG) call recordings (they run past midnight in my timezone :( ) and community calls. There are a lot of contributors from organisations who deploy Kubeflow internally and as vendors. One of the more popular requests from the community is to build Helm charts for deploying the Kubeflow platform.

### What on earth is Kubeflow?

The Kubeflow platform is a collection of machine learning tools designed to be deployed on Kubernetes. Kubeflow components can be deployed on A community maintained set of Kubernetes manifests exists ([kubeflow/manifests](https://github.com/kubeflow/manifests)) and provides a base for deploying the Kubeflow platform on Kubernetes. The repository holds a collection of Kustomization manifests upstreamed from each Kubeflow component. It also implements an [Istio](https://istio.io) service mesh which secures traffic between components along with other security measures like pod security standards, network policies and authorization tokens. It can also be configured to user external OAuth2 provider. The scope of designing Kubeflow helm charts is enormous and no easy effort, which makes sense why the Kubeflow organisation chooses not to maintain a central distribution of Kubeflow and leaves this to vendors. Like many open-source projects, contributors are either funded (working time) by organisations or independently volunteer their own time. Kubeflow is already extremely complex to maintain to be security compliant for enterprises and has its own challenges designing and coordinating AI/ML tooling on Kubernetes.

I recently went through deploying a customised Kubeflow. I operate a [bare-metal Kubernetes cluster](https://github.com/solanyn/home-ops) using GitOps using [Flux](https://fluxcd.io). I usually reach for Helm charts and occassionally deploy operators and/or Custom Resource Definitions (CRDs) from manifests. I recently moved away from [Ingress Nginx Controller](https://kubernetes.github.io/ingress-nginx/) to the new [Gateway API](https://gateway-api.sigs.k8s.io) using the [Cilium Gateway implementation](https://docs.cilium.io/en/stable/network/servicemesh/gateway-api/gateway-api/) which posed some challenges with Istio.

Deploying the Kubeflow platform has a few external dependencies:

- S3 compatible storage
- MySQL database
- Istio
- Cert Manager

I already deploy Cert Manager for certificates for my split-brain DNS setup to publicly expose services via Cloudflare DNS and Cloudflare tunnels and internal (only on my network) using External DNS and the external-dns-unifi-webhook and a MinIO deployment with an NFS backend to my NAS. I settled on MariaDB operator for its convenient CRDs to create databases, users and grants and scheduled S3 backup and restore procedures.

### Kubeflow and Fluxtomizations

I largely used the [deployKF](https://github.com/deployKF/deployKF) repository as inspiration which a more configurable set up using gomplate, ArgoCD and raw bash scripts to configure settings. It's also a couple of revisions old (my guess is the maintainer started vendoring the distribution for later Kubeflow versions). It uses the same mechanism Helm values are templated into charts to create customised deployments AND kustomizations. I ended up opting for configurable options through Kubernetes Secret and ConfigMap objects. One of the biggest challenges was identifying all the points where MySQL and MinIO credentials were needed and using Kustomize patches to use a centralised secret. I should mention that secrets can be populated manually, using GitOps with [sops](https://github.com/getsops/sops) encryption (natively supported in Flux) or use [External Secrets Operator](https://external-secrets.io/latest/) to populate secrets from an external secret store. I personal use 1Password to populate my secrets which allows me to conveniently autofill credentials.

Roughly speaking I went through the following steps:

1. Write a GitHub actions workflow to commit the latest non release candidate tag of the manifests to my repository
2. Add components to generate namesapces with the appropriate labels e.g., for istio injection
3. Add OAuth2 providers from my own [Pocket ID](https://pocket-id.org/) deployment and GitHub to the `dex` and `oauth2-proxy` deployments using ExternalSecret and ConfigMap definitions
4. Add Kustomize patches to patch `dex` and `oauth2-proxy` deployments to mount the configuration I defined
5. Add an ExternalSecret manifest with configurations for ALL Kubeflow components
6. Create Database objects for Kubeflow components that use MySQL/MySQL compatible databases
7. Add Kustomize patches to patch Kubeflow components (mainly Katib, Pipelines and Model Registry) to connect to my MariaDB cluster and the appropriate tables
8. Add Kustomize patches to patch Kubeflow Pipelines to connect to my MinIO deployment
9. Add any additional Istio RequestAuthentication manifests to enable authorization from GitHub and Pocket ID

### Conclusion

Of course, things could break with the next release of manifests and I may have to go through the whole process again. The key things that I think will make this easily configurable are using Kustomize patches to update deployments. Being a contributor helps since I would be well-aware ahead of time if there are any significant manifest changes. There are plans to include some new features in like supporting Gateway API and using Istio Ambient mode which could pose some new challenges. Until then, thanks for reading!
