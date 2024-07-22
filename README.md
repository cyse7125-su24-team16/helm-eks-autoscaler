Here's an updated sample `README.md` file that reflects the configuration details from your configuration file

```markdown
# Helm EKS Autoscaler

This Helm chart deploys an autoscaler on an Amazon EKS cluster.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+
- An Amazon EKS cluster

## Installation

1. Add the Helm repository (if you have a custom repository):

   ```sh
   helm repo add <repo-name> <repo-url>
   helm repo update
   ```

2. Install the chart with the release name `my-autoscaler`:

   ```sh
   helm install my-autoscaler <repo-name>/helm-eks-autoscaler
   ```

   Alternatively, you can clone this repository and install the chart from the local directory:

   ```sh
   git clone https://github.com/your-repo/helm-eks-autoscaler.git
   cd helm-eks-autoscaler
   helm install my-autoscaler .
   ```

## Uninstallation

To uninstall the `my-autoscaler` release:

```sh
helm uninstall my-autoscaler
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the Helm chart and their default values.

| Parameter                                 | Description                                                 | Default                                  |
| ----------------------------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| `replicas`                                | Number of autoscaler replicas                               | `1`                                      |
| `image.repository`                        | Autoscaler image repository                                 | `your-repository-here`                   |
| `image.tag`                               | Autoscaler image tag                                        | `latest`                                 |
| `image.pullPolicy`                        | Image pull policy                                           | `IfNotPresent`                           |
| `resources.limits.cpu`                    | CPU limit for the autoscaler                                | `500m`                                   |
| `resources.limits.memory`                 | Memory limit for the autoscaler                             | `512Mi`                                  |
| `resources.requests.cpu`                  | CPU request for the autoscaler                              | `100m`                                   |
| `resources.requests.memory`               | Memory request for the autoscaler                           | `256Mi`                                  |
| `deployment.awsRegion`                    | AWS region for the deployment                               | `your-region-here`                       |
| `deployment.AWS_ACCESS_KEY_ID`            | AWS access key ID                                           | `your-access-key-id-here`                |
| `deployment.AWS_SECRET_ACCESS_KEY`        | AWS secret access key                                       | `your-secret-access-key-here`            |
| `rbac.serviceAccountName`                 | Service account name for RBAC                               | `your-service-account-name-here`         |
| `rbac.clusterRoleName`                    | Cluster role name for RBAC                                  | `your-cluster-role-name-here`            |
| `rbac.roleName`                           | Role name for RBAC                                          | `your-role-name-here`                    |
| `rbac.roleBindingName`                    | Role binding name for RBAC                                  | `your-role-binding-name-here`            |
| `rbac.clusterRoleBindingName`             | Cluster role binding name for RBAC                          | `your-cluster-role-binding-name-here`    |
| `autoscalerNamespace`                     | Namespace for the autoscaler                                | `your-namespace-here`                    |
| `annotations."eks.amazonaws.com/role-arn"`| ARN of the IAM role for the autoscaler                      | `arn:aws:iam::123456789012:role/ExampleRole` |
| `autoscaler.cloudProvider`                | Cloud provider for the autoscaler                           | `aws`                                    |
| `autoscaler.expander`                     | Expander for the autoscaler                                 | `least-waste`                            |
| `autoscaler.nodeGroupAutoDiscovery`       | Node group auto discovery settings                          | `[]`                                     |
| `autoscaler.balanceSimilarNodeGroups`     | Balance similar node groups                                 | `true`                                   |
| `autoscaler.skipNodesWithLocalStorage`    | Skip nodes with local storage                               | `true`                                   |
| `autoscaler.skipNodesWithSystemPods`      | Skip nodes with system pods                                 | `true`                                   |
| `secrets.myregistrykey.name`              | Name of the Docker registry secret                          | `your-secret-name-here`                  |
| `secrets.myregistrykey.dockerconfigjson`  | Docker config JSON for the registry secret                  | `your-docker-config-json-here`           |

You can override these parameters in your own `values.yaml` file and install the chart:

```sh
helm install my-autoscaler -f custom-values.yaml .
```

## Maintainers

- [Vaishnavi Mantri](https://github.com/VaishnaviNEU)
- [Anusha Senthilnathan](https://github.com/Anu-3980)
