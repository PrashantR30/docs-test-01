# Cloud Provider Credentials Management in CAPI

Cloud provider credentials in Cluster API (CAPI) environments are managed through objects in the management cluster.
`Credential`, `ClusterIdentity`, and `Secret` (related to `ClusterIdentity`) objects handle credential storage and management, while a dedicated `ConfigMap` object is used as a template to render configuration into child clusters.

## Configuration Patterns

The configuration follows two patterns:

1. **ClusterIdentity Pattern**
   - Uses a `ClusterIdentity` resource that defines provider identity configuration
   - References a `Secret` with credentials
   - Used by `Azure` and `vSphere` in-tree providers

2. **Source Secret Pattern**
   - Uses only a `Secret` without `ClusterIdentity`
   - `Secret` contains all cloud configuration data
   - Used by `OpenStack` in-tree provider

In both cases `ConfigMap` with template code is used to render configuration into child clusters.

### Credential Resource

The `Credential` resource provides an abstraction layer by either:
- Referencing a `ClusterIdentity` through `identityRef`
- Directly referencing a `Secret`, depending on the pattern used

### Template ConfigMap

- Marked with `projectsveltos.io/template: "true"` annotation
- Contains Go template code for generating child cluster resources via the Sveltos templating system
- Template processing accesses cluster objects through:
  - Built-in Sveltos variables (`Cluster`, `InfrastructureProvider`)
  - `getResource` function for additionally exposed objects (`InfrastructureProviderIdentity`, `InfrastructureProviderIdentitySecret`)

## Templating System

The templating system leverages:
- [Golang templating](https://pkg.go.dev/text/template)
- [Sprig functions](https://masterminds.github.io/sprig)
- [Sveltos resource manipulation functions](https://projectsveltos.github.io/sveltos/template/intro_template/#:~:text=Resource%20Manipulation-,functions,-Sveltos%20provides%20a)

### Examples

Provider-specific examples are available in `*.credentials.yaml` files [here](https://github.com/k0rdent/kcm/tree/main/config/dev).

Let's take `Azure` provider as an example [azure-credentials.yaml](https://github.com/k0rdent/kcm/blob/v0.1.0/config/dev/azure-credentials.yaml)

 - [`ClusterIdentity`](https://github.com/k0rdent/kcm/blob/v0.1.0/config/dev/azure-credentials.yaml#L2-L17)
 - [`Secret`](https://github.com/k0rdent/kcm/blob/v0.1.0/config/dev/azure-credentials.yaml#L19-L28) (related to `ClusterIdentity`)
 - [`Credential`](https://github.com/k0rdent/kcm/blob/v0.1.0/config/dev/azure-credentials.yaml#L30-L41)
 - [`ConfigMap`](https://github.com/k0rdent/kcm/blob/v0.1.0/config/dev/azure-credentials.yaml#L43-L97)

## Provider Registration

Providers are registered through YAML configuration files mounted into a [predefined path](https://github.com/k0rdent/kcm/blob/v0.1.0/templates/provider/kcm/templates/deployment.yaml#L42-L43) in the manager container at startup [using `ConfigMap`](https://github.com/k0rdent/kcm/blob/v0.1.0/templates/provider/kcm/templates/providers.yaml).

### Examples

Provider configuration examples can be found [here](https://github.com/k0rdent/kcm/tree/v0.1.0/providers)

Let's take `Azure` provider as an example [azure.yml](https://github.com/k0rdent/kcm/blob/v0.1.0/providers/azure.yml), as seen, the definition is straightforward.
