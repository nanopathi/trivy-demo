## Repository to demo Trivy

This repository provides several examples, showcasing Trivy. Trivy is an open source security scanner.

### Resources
- GitHub repository: https://github.com/aquasecurity/trivy
- Documentation: https://aquasecurity.github.io/trivy/latest/
- Slack Channel: https://slack.aquasec.com

### Presentations
- Container Security on AKS with AquaSecurity OSS: https://youtu.be/a6iUUWqJDg0?t=297

## Trivy CLI

Installation options: https://aquasecurity.github.io/trivy/latest/getting-started/installation/

Below are some of the commands that can be used through the Trivy CLI.

### GIT Repository Scans
[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/vulnerability/scanning/git-repository/)

You can scan any remote 
```
trivy repo <repo-name>
trivy repo https://github.com/Aristat/golang-example-app
```

Pass a --branch argument with a valid branch name on the remote repository provided:
```
trivy repo --branch <branch-name> <repo-name>
trivy repo --branch gin-example https://github.com/Aristat/golang-example-app
```

Pass a --commit argument with a valid commit hash on the remote repository provided:
```
trivy repo --commit <commit-hash> <repo-name>
trivy repo --commit f7be6c387f0e9f2e05e4a3e631b0186014b463c9 https://github.com/Aristat/golang-example-app
```

Pass a --tag argument with a valid tag on the remote repository provided:
```
trivy repo --tag <tag-name> <repo-name>
trivy repo --tag v0.0.6 https://github.com/Aristat/golang-example-app
```

### Vulnerability Scans

[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/vulnerability/scanning/)

Scan a container image for vulnerabilities:
```
trivy i ubuntu:20.04
```

By default Trivy will use the Docker Hub, however, you can also specify any other registry:
```
trivy i public.ecr.aws/docker/library/ubuntu:20.04
```

Scan a container image for vulnerabilities but ignore all vulnerabilities that do not have a fix available:
```
trivy i --ignore-unfixed ubuntu:20.04
```

Scan a container image for vulnerabilities but filter for HIGH and CRITICAL vulnerabilities
```
trivy image --severity HIGH,CRITICAL --vuln-type os postgres:10.6
```

Scan any GH repository for vulnerabilities:
```
trivy repo --vuln-type library https://github.com/raesene/sycamore
```

Scan any filesystem for vulnerabilities:
```
git clone https://github.com/raesene/sycamore
trivy fs ./sycamore/
```

### Misconfiguration Scans

[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/misconfiguration/scanning/)

Scan all of your infrastructure configuration for vulnerabilities:
```
ls bad_iac
```

Scan your Dockerfile for vulnerabilities and misconfigurations:
```
trivy config trivy-demo/bad_iac/docker/
```

Scan your Kubernetes manifests for vulnerabilities and misconfigurations:
```
trivy config trivy-demo/bad_iac/kubernetes
```

Scan your Terraform for vulnerabilities and misconfigurations:
```
trivy config trivy-demo/bad_iac/terraform
```

### Custom Policies

[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/misconfiguration/custom/)

Trivy makes it possible to scan custom policies defined in Rego.

Note that if Rego is not your cup of tea and you are focusing on Terraform scans, you can specify custom policies in JSON and YAML format in tfsec.

The following file provides a custom polocy that compares a Kubernetes deployment and a Kubernetes service. It then scans them to see whether they have the same selectors applied:
```
cat custom-policies/combine-yaml.rego
```

The following command will run the scan:
```
trivy conf --severity CRITICAL --policy ./custom-policies/combine-yaml.rego --namespaces user ./manifests
```

### Trivy Cloud | Trivy AWS
[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/cloud/aws/scanning/)

See all the options to scan your AWS services by running the following command:
```
trivy aws --help 
```

Perform an account wide scan:
```
trivy aws  
```

The cache is naturally saved for 24h. This makes scanning your resources again or looking in detail at specific services much quicker:
```
trivy aws --update-cache 
```

Alternatively, you can also specify a max-cache age:
```
trivy aws --max-cache-age 12h 
```

By default, Trivy will either connect with your configured default region or by the region in your ENV variable. However, you can also scan any other region with the 
```
trivy aws --region eu-central-1
```

Alternatively to scnaning your entire cluster or once you have scanned your entire cluster, you can also look at specific services:
```
trivy aws --region eu-central-1 --service ec2 
```

Next, you can look at a specific service by specifying the arn:
```
trivy aws --region eu-central-1 --service ec2 --arn arn:aws:ec2:eu-central-1:XXXXXXXXXXXX:vpc/vpc-00ce30b51ebebb314 
```

OR by specifying the `--format` should be json:
```
trivy aws --region eu-central-1 --format json --service ec2 
```

Jless is a great tool to view the output better:
```
trivy aws --region eu-central-1 --format json --service ec2 | jless 
```

You can also scan myltiple different services at once:
```
trivy aws --region us-east-1 --service s3 --service ec2
```

### Scan your connected Kubernetes cluster

[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/kubernetes/cli/scanning/)

The Trivy Kubernetes command scans any connected Kubernetes cluster for vulnerabilities, misconfigurations, exposed secrets and more.

To scan your entire cluster and receive a summary report use the following command:

```
trivy k8s --report summary cluster
```

To scan a specific namespace in your cluster and receive a summary report use the following command:
```
trivy k8s --n kube-system --report summary cluster
```

To receive a detailed report, you can use the `--report=all` flag. However, we would advice to only do that on specific namespaces or resources since you will be provided with a lot of detailed information:
```
trivy k8s --n kube-system --report all cluster
```

Similar to vulnerabilities, we can also filter in-cluster for specific vulnerabilty types:
```
trivy k8s --severity=CRITICAL --report all cluster
```

With the trivy K8s command, you can also scan specific workloads that are running within your cluster:
```
trivy k8s -–n default --report summary cluster deployments/react-application
```
or
```
trivy k8s –-n app --report summary cluster deployments/react-application
```

## Trivy Operator

[**Documentation**](https://aquasecurity.github.io/trivy-operator/latest)

While you would use the Trivy CLI on your local machine or from within a CI/CD pipeline, the Trivy operator is installed inside your Kubernetes cluster. From there, it performs continuous scanning of your Kuberentes resources.
Have a look at the documentation for more information:

Installation options: https://aquasecurity.github.io/trivy-operator/v0.0.8/

### Scan your connected Kubernetes cluster

Once the operator is installed, you will see it running in the Trivy System namespace:
```
kubectl get all -n trivy-system
```

Now, we can install a deployment. If you do not have a deployment within your cluster, you can use our example application:
```
kubectl apply -f manifests 
```

Trivy will then automatically scan new deployments:
```
kubectl get Vulnerabilityreports
```

To get detailed information on the vulnerabilities, describe the Vulnerabilityreports:
```
kubectl describe Vulnerabilityreports replicaset-react-application-79694589b9-react-application
```

## Trivy Config

[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/references/customization/config-file/)

The Trivy Config allows us to define the configurations for our security scans in a YAML manifest. An example is provided in this repository within [./trivy-config.yaml](./trivy-config.yaml)

You can then use the config manifest in your security scans such as your image vulnerability scans:

```
trivy image --severity HIGH node:14
```

## Trivy SBOM & Attestation

[**Documentation**](https://aquasecurity.github.io/trivy/latest/docs/sbom/)

For more information watch the following tutorials:

1. [Generate SBOMs with Trivy & Scan SBOMs for vulnerabilities](https://youtu.be/Kibk6qq7ZCs)
2. [Creating an SBOM Attestation with Trivy and Cosign from Sigstore](https://youtu.be/nF15vzo5Gts)

Trivy can generate SBOMs in the following three formats SPDX, SPDX-json, and CycloneDX.

Command structure:
```
trivy image --format <spdx,spdx-json,cyclonedx> -o <sbom.spdx,sbom.spdx.json,sbom.json> <IMAGE> 
```

Example:
```
trivy image --format spdx -o sbom.spdx anaisurlichs/cns-website:0.0.6 
```

An attestation can be generated with Cosign and in-toto. For this, first generate a key-pair with Cosign:
```
cosign generate-key-pair
```

Creater the attestation fo the SBOM with your private-key with the following structure:
```
cosign attest --key /path/to/cosign.key --type spdx --predicate sbom.spdx <IMAGE>
```

Example:
```
cosign attest --key ./cosign.key --type spdx --predicate sbom.spdx anaisurlichs/cns-website:0.0.6 
```

Verify the attestation with your public key:
```
cosign attest --key /path/to/cosign.pub --type spdx --predicate sbom.spdx <IMAGE> 
```

Following the example again:
```
cosign attest --key /path/to/cosign.pub --type spdx --predicate sbom.spdx <IMAGE> 
```
