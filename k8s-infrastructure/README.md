# Creation of required Kubernetes infrastrucutre

## kubernetes-external-secrets
CDK by default stores the secrets for things like our RDS database in AWS Secrets Manager. We are going to leverage an operator for AWS Secrets Manager (https://github.com/godaddy/kubernetes-external-secrets) to be able to get those secrets in to Kubernetes to leverage within our `k8s-app-resrouces` like Ghost.

This operator is installed via a Helm chart so we need to set up Helm on the cluster and then we use that to install the operator.

### Prerequisites
Kubectl w/IAM authenticator plugin, Helm CLI, jq, fluxctl

To install on Amazon Linux run `./install-prereqs.sh`. Further instructions coming for other OSes soon.

### Instructions
1. Run `install-helm.sh` to install Helm's backend (Tiller) to the cluster
1. Run `install-external-secrets.sh` to install kubernetes-external-secrets (https://github.com/godaddy/kubernetes-external-secrets) to map the secret(s) for our RDS in Secrets Manager to Kubernetes secrets
1. Run `update-ghost-external-secret.sh` to fill in the name of the secret in Secrets Manager into `ghost-externalsecret.yaml`
1. Run `kubectl apply -f ghost-externalsecret.yaml`

## Flux
Flux is a GitOps tool that runs in the cluster and watches a Git repository for changes. On changes it will work out if there are any deltas and, if there are, deploy them to the cluster.

To install Flux:
1. Run `sudo curl --location -o /usr/local/bin/fluxctl "https://github.com/fluxcd/flux/releases/download/1.15.0/fluxctl_linux_amd64"`
1. Run `sudo chmod +x /usr/local/bin/fluxctl`
1. Run `./install-flux.sh`

## ExternalDNS
The Service or Ingress will create a Load Balancer with a long random name in the AWS DNS namespace. If you'd prefer to have a CNAME with an intelligable name to that to use instead you can leverage External DNS (https://github.com/kubernetes-sigs/external-dns).

To install ExternalDNS:
1. Run `kubectl apply -f exernaldns.yaml`