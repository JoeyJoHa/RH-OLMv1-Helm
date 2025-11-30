# OLMv1 Helm Chart

> **Disclaimer**: This repo contains AI-generated content using Cursor / Gemini AI.

A Helm chart for deploying operators using Operator Lifecycle Manager V1 (OLMv1) with proper RBAC configuration.

## Overview

This Helm chart simplifies the deployment of Kubernetes operators using OLMv1 by providing a declarative, template-based approach to managing operator installations with security-first RBAC configurations.

## Features

- **Declarative Operator Deployment**: Deploy operators using simple Helm values
- **Security-First RBAC**: Automated creation of ServiceAccounts, Roles, and RoleBindings with least-privilege principles
- **Flexible Configuration**: Support for both cluster-scoped and namespace-scoped permissions
- **Additional Resources**: Deploy custom resources alongside operators
- **Production-Ready**: Battle-tested templates with comprehensive validation

## Prerequisites

- Openshift/Kubernetes cluster with OLMv1 installed(Openshift 4.18+)
- Helm 3.x
- `oc` or `kubectl` configured to access your cluster
- Appropriate cluster permissions to create RBAC resources

## Installation

### Quick Start

```bash
# Add the Helm repository (if published)
helm repo add olmv1 <repository-url>
helm repo update

# Install with default values
helm install my-operator olmv1/olmv1-operator

# Install with custom values
helm install my-operator olmv1/olmv1-operator -f values.yaml
```

### Local Installation

```bash
# Clone the repository
git clone <repository-url>
cd OLMv1-Helm-Chart

# Install from local chart
helm install my-operator . -f values.yaml
```

## Configuration

### Basic Configuration

The chart is configured through the `values.yaml` file. Here's a minimal example:

```yaml
operator:
  name: argocd-operator
  create: true
  packageName: argocd-operator
  appVersion: "0.6.0"
  channel: stable

serviceAccount:
  create: true
  name: argocd-operator-sa
  bind: true

permissions:
  clusterRoles:
    - name: ""  # Auto-generated name
      type: "operator"
      create: true
      customRules:
        - apiGroups: [""]
          resources: ["configmaps", "secrets", "services"]
          verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```

### Configuration Options

#### Operator Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `operator.name` | Name of the operator deployment | `"my-operator"` |
| `operator.create` | Create ClusterExtension resource (false for RBAC-only) | `true` |
| `operator.packageName` | Operator package name from catalog | `"my-operator-package"` |
| `operator.appVersion` | **Operator version** to install (e.g., "3.10.13", "latest") | `"latest"` |
| `operator.channel` | Channel to use for the operator (e.g., "stable", "alpha") | `"stable"` |

**Note**: The operator will be installed in the Helm release namespace (`{{ .Release.Namespace }}`).

#### ServiceAccount Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Create a ServiceAccount | `true` |
| `serviceAccount.name` | ServiceAccount name (empty for auto-generated) | `""` |
| `serviceAccount.bind` | Bind ServiceAccount to RBAC resources | `true` |
| `serviceAccount.annotations` | ServiceAccount annotations | `{}` |
| `serviceAccount.labels` | ServiceAccount labels | `{}` |

#### Permissions Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `permissions.clusterRoles` | List of ClusterRole configurations | See values.yaml |
| `permissions.clusterRoles[].name` | ClusterRole name (empty for auto-generated) | `""` |
| `permissions.clusterRoles[].type` | Type: "operator" or "grantor" | `"operator"` |
| `permissions.clusterRoles[].create` | Create the ClusterRole | `true` |
| `permissions.clusterRoles[].customRules` | List of RBAC rules | `[]` |
| `permissions.roles` | List of namespaced Role configurations | See values.yaml |
| `permissions.roles[].name` | Role name (empty for auto-generated) | `""` |
| `permissions.roles[].type` | Type: "operator" or "grantor" | `"operator"` |
| `permissions.roles[].create` | Create the Role | `false` |
| `permissions.roles[].customRules` | List of RBAC rules | `[]` |

**Note**: Binding names are auto-generated with "-cluster-binding" or "-namespace-binding" suffixes.

#### Additional Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `additionalResources` | List of additional Kubernetes resources | `[]` |

### Example Configurations

See the `examples/` directory for complete configuration examples:

- `rbac-only-example.yaml` - Minimal RBAC configuration
- `additional-resources-example.yaml` - Operator with custom resources
- `values-quay-operator.yaml` - Complete Quay operator example

## Usage

### Deploy an Operator

1. **Prepare your values file**:

```yaml
operator:
  name: my-operator
  create: true
  packageName: my-operator-package
  appVersion: "1.0.0"
  channel: stable

serviceAccount:
  create: true
  name: my-operator-sa
  bind: true

permissions:
  clusterRoles:
    - name: my-operator-cluster-role
      type: "operator"
      create: true
      customRules:
        - apiGroups: [""]
          resources: ["pods", "services"]
          verbs: ["get", "list", "watch"]
```

2. **Install the chart**:

```bash
helm install my-operator . -f my-values.yaml
```

3. **Verify the installation**:

```bash
# Check the release
helm list

# Check the operator status
kubectl get clusterextension my-operator

# Check the operator pods
kubectl get pods -n my-namespace
```

### Upgrade an Operator

To upgrade the **operator version**:

```bash
# Update your values file with new operator version
helm upgrade my-operator . -f my-values.yaml

# Or upgrade the operator version inline
helm upgrade my-operator . --set operator.appVersion=1.1.0
```

### Uninstall an Operator

```bash
# Uninstall the Helm release
helm uninstall my-operator

```

## Chart Structure

```tree
OLMv1-Helm-Chart/
├── Chart.yaml                    # Chart metadata
├── values.yaml                   # Default configuration values
├── templates/
│   ├── _helpers.tpl             # Template helpers
│   ├── NOTES.txt                # Post-installation notes
│   ├── serviceaccount.yaml      # ServiceAccount template
│   ├── clusterrole.yaml         # ClusterRole template
│   ├── role.yaml                # Role template
│   ├── clusterextension.yaml    # ClusterExtension template
│   └── additional-resources.yaml # Additional resources template
└── README.md                     # This file
```

## Best Practices

### Security

1. **Least Privilege**: Only grant the minimum required permissions in your RBAC rules
2. **Review RBAC**: Carefully define RBAC rules in `values.yaml` for your operator's needs
3. **ServiceAccount per Operator**: Create dedicated ServiceAccounts for each operator

### Configuration Management

1. **Version Control**: Store your values files in version control
2. **Environment-Specific Values**: Use separate values files for dev/staging/prod
3. **Secrets Management**: Use external secret management (e.g., Sealed Secrets, External Secrets Operator)
4. **Validation**: Test deployments in non-production environments first

### Maintenance

1. **Regular Updates**: Keep operator versions up to date
2. **Monitor Resources**: Watch for deprecated API versions
3. **Backup Configurations**: Maintain backups of your values files
4. **Documentation**: Document custom configurations and decisions

## Troubleshooting

### Common Issues

#### ClusterExtension Not Installing

```bash
# Check the ClusterExtension status
kubectl describe clusterextension my-operator

# Check operator-controller logs
kubectl logs -n olmv1-system -l app=operator-controller
```

#### RBAC Permission Errors

```bash
# Verify ServiceAccount exists
kubectl get serviceaccount my-operator-sa -n my-namespace

# Check Role/ClusterRole bindings
kubectl get rolebinding,clusterrolebinding | grep my-operator

# Review RBAC rules
kubectl describe clusterrole my-operator-cluster-role
```

#### Catalog Not Found

```bash
# List available catalogs
kubectl get clustercatalog

# Check catalog status
kubectl describe clustercatalog operatorhubio-catalog
```

### Debug Mode

Enable debug output in Helm:

```bash
helm install my-operator . -f values.yaml --debug --dry-run
```

## Related Projects

- **[OLMv1 Case Study](https://github.com/yourusername/OLMv1-CaseStudy)**: Examples and documentation for OLMv1 deployments
- **[OLMv1 RBAC Manager](https://github.com/yourusername/OLMv1-RBAC-Manager)**: Tool for extracting and managing operator RBAC permissions

## Support

For issues and questions:

- Open an issue in this repository
- Check the [OLMv1 documentation upstream](https://github.com/operator-framework/operator-controller)
- Check the [OLMv1 documentation downstream](https://github.com/openshift/operator-framework-operator-controller)
- Review the examples in the [Case Study repository](https://github.com/yourusername/OLMv1-CaseStudy)
