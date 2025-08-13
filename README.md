# DNS Record Service Template

A Backstage Software Template for managing DNS records using Crossplane.

## Overview

This template creates the necessary Crossplane resources to manage DNS records in your infrastructure. It includes:

- **XRD (Composite Resource Definition)**: Defines the DNSRecord API
- **Composition**: Implements the DNS record management logic
- **Example Claim**: Shows how to create a DNS record

## Features

- 🌐 **Multiple Record Types**: Support for A, AAAA, CNAME, and TXT records
- 🔧 **Simple Configuration**: Easy-to-use parameters for DNS management
- 📊 **FQDN Tracking**: Fully qualified domain name available in status
- ⚡ **Quick Provisioning**: Instant DNS record creation

## Prerequisites

### Crossplane Installation

Ensure Crossplane is installed in your cluster:

```bash
kubectl create namespace crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

### Required Functions

Install the necessary Crossplane composition functions:

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-go-templating
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-go-templating:v0.10.0
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-auto-ready
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-auto-ready:v0.2.1
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-environment-configs
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-environment-configs:v0.1.0
EOF
```

### Environment Configuration

Create an environment configuration for DNS settings:

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: EnvironmentConfig
metadata:
  name: dnsrecord
data:
  zone: example.com  # Your DNS zone
```

## Usage

### Through Backstage

1. Navigate to the Software Catalog
2. Click "Create Component"
3. Select "DNS Record"
4. Fill in the required parameters:
   - **Resource Name**: Unique identifier for this DNS record
   - **Namespace**: Kubernetes namespace (default: `default`)
   - **Record Type**: A, AAAA, CNAME, or TXT
   - **DNS Name**: The hostname (without domain)
   - **Record Value**: IP address, hostname, or text value
   - **Owner**: Team or user who owns this resource

### Manual Deployment

1. Apply the XRD:
```bash
kubectl apply -f content/definition.yaml
```

2. Apply the Composition:
```bash
kubectl apply -f content/composition.yaml
```

3. Create a claim:
```bash
kubectl apply -f content/example.yaml
```

## Parameters

| Parameter | Description | Type | Default | Required |
|-----------|-------------|------|---------|----------|
| `name` | Resource name | string | - | Yes |
| `namespace` | Kubernetes namespace | string | `default` | No |
| `recordType` | DNS record type | string | `A` | No |
| `dnsName` | DNS hostname | string | - | Yes |
| `recordValue` | Record value | string | - | Yes |
| `owner` | Resource owner | string | `group:platform` | No |

## Record Types

### A Record (IPv4)
Maps a hostname to an IPv4 address.
```yaml
spec:
  type: A
  name: app
  value: "192.168.1.100"
```

### AAAA Record (IPv6)
Maps a hostname to an IPv6 address.
```yaml
spec:
  type: AAAA
  name: app
  value: "2001:db8::1"
```

### CNAME Record
Creates an alias to another hostname.
```yaml
spec:
  type: CNAME
  name: www
  value: "app.example.com"
```

### TXT Record
Stores text data, often used for verification or SPF records.
```yaml
spec:
  type: TXT
  name: _verification
  value: "v=spf1 include:_spf.example.com ~all"
```

## Architecture

The template creates a composite resource that:

1. **ConfigMap**: Stores DNS record configuration
2. **DNS Provider Integration**: Would connect to actual DNS provider (Route53, CloudDNS, etc.)
3. **Status Updates**: Returns FQDN in resource status

## Getting the FQDN

After provisioning, retrieve the fully qualified domain name:

```bash
kubectl get dnsrecord <record-name> -n <namespace> -o jsonpath='{.status.fqdn}'
```

## Development

### Testing Locally

1. Clone this repository
2. Install the template in Backstage:
```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/open-service-portal/service-dnsrecord-template/blob/main/template.yaml
```

### Customization

To customize DNS record management:

1. Edit `content/definition.yaml` to add parameters
2. Modify `content/composition.yaml` to integrate with your DNS provider
3. Update `template.yaml` to expose new parameters in Backstage

## Production Considerations

For production use, you'll need to:

1. **Install a DNS Provider**: Such as provider-aws, provider-azure, or provider-gcp
2. **Configure Provider Credentials**: Set up authentication for your DNS provider
3. **Update Composition**: Replace the mock implementation with actual DNS provider resources

Example with AWS Route53:
```yaml
- step: create-route53-record
  functionRef:
    name: function-go-templating
  input:
    apiVersion: gotemplating.fn.crossplane.io/v1beta1
    kind: GoTemplate
    source: Inline
    inline:
      template: |
        apiVersion: route53.aws.upbound.io/v1beta1
        kind: Record
        spec:
          forProvider:
            region: us-east-1
            type: {{ .observed.composite.resource.spec.type }}
            name: {{ .observed.composite.resource.spec.name }}
            records:
              - {{ .observed.composite.resource.spec.value }}
            ttl: 300
            zoneIdRef:
              name: my-zone
```

## Troubleshooting

### Common Issues

**DNS record not created**
- Check if environment configuration is properly set
- Verify DNS zone is configured correctly
- Check composition logs: `kubectl describe composition dnsrecord`

**FQDN not appearing in status**
- Ensure the environment config contains the `zone` field
- Check if the composition pipeline completed successfully

**Permission errors**
- Verify Crossplane has necessary permissions for ConfigMap creation
- Check RBAC settings for the namespace

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT License - see LICENSE file for details

## Support

For issues and questions:
- GitHub Issues: [open-service-portal/service-dnsrecord-template](https://github.com/open-service-portal/service-dnsrecord-template/issues)
- Discussions: [open-service-portal/discussions](https://github.com/orgs/open-service-portal/discussions)