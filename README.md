# Ansible Role: Velero

[![CI](https://github.com/astigmata/ansible-role-velero/actions/workflows/ci.yml/badge.svg)](https://github.com/astigmata/ansible-role-velero/actions/workflows/ci.yml)

Deploys Velero backup and disaster recovery solution on Kubernetes.

## Description

This role installs Velero with AWS S3-compatible storage backend. It supports any S3-compatible storage provider including AWS S3, MinIO, Ceph, and others.

The role can install Velero using either:
- **Velero CLI** (if available on the system)
- **Kubernetes manifests** (if CLI is not available)

## Requirements

- Kubernetes cluster accessible via `kubectl`
- Ansible collection: `kubernetes.core`
- S3-compatible storage endpoint (AWS S3, MinIO, etc.)

Install the required collection:

```bash
ansible-galaxy collection install kubernetes.core
```

## Role Variables

### Required Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `s3_endpoint` | `""` | S3 endpoint URL (e.g., `https://s3.amazonaws.com` or `http://minio:9000`) |
| `s3_access_key` | `""` | S3 access key ID |
| `s3_secret_key` | `""` | S3 secret access key |
| `s3_bucket` | `velero` | S3 bucket name for backups |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `velero_namespace` | `velero` | Kubernetes namespace for Velero |
| `velero_version` | `1.17.2` | Velero version to install |
| `velero_aws_plugin_version` | `1.13.2` | AWS plugin version |
| `velero_image` | `velero/velero:v{{ velero_version }}` | Velero image |
| `velero_aws_plugin_image` | `velero/velero-plugin-for-aws:v{{ velero_aws_plugin_version }}` | AWS plugin image |
| `s3_region` | `us-east-1` | S3 region |
| `s3_force_path_style` | `true` | Use path-style URLs (required for MinIO) |
| `velero_backup_ttl` | `720h` | Default backup TTL (30 days) |
| `velero_use_volume_snapshots` | `false` | Enable volume snapshots |

## Dependencies

None. This role is designed to work independently.

If using the `minio` role from this collection, S3 variables are automatically set.

## Example Playbook

### With AWS S3

```yaml
- hosts: localhost
  roles:
    - role: velero
      vars:
        s3_endpoint: "https://s3.amazonaws.com"
        s3_access_key: "AKIAIOSFODNN7EXAMPLE"
        s3_secret_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
        s3_bucket: "my-velero-backups"
        s3_region: "eu-west-1"
        s3_force_path_style: false
```

### With MinIO (manual configuration)

```yaml
- hosts: localhost
  roles:
    - role: velero
      vars:
        s3_endpoint: "http://minio.minio.svc:9000"
        s3_access_key: "minio"
        s3_secret_key: "minio123"
        s3_bucket: "velero"
        s3_region: "minio"
        s3_force_path_style: true
```

### With MinIO role (automatic)

```yaml
- hosts: localhost
  roles:
    - role: minio   # Sets s3_* variables automatically
    - role: velero  # Uses s3_* variables from minio role
```

### Using command line

```bash
# With external S3
ansible-playbook playbook.yml --tags velero \
  -e s3_endpoint=https://s3.amazonaws.com \
  -e s3_access_key=AKIAXXXX \
  -e s3_secret_key=secret \
  -e s3_bucket=my-bucket \
  -e s3_region=eu-west-1

# After minio role has run
ansible-playbook playbook.yml --tags minio,velero
```

## Velero CLI Commands

After deployment, use the Velero CLI to manage backups:

```bash
# Check backup location status
velero backup-location get

# Create a backup
velero backup create my-backup --include-namespaces app-namespace

# List backups
velero backup get

# Restore from backup
velero restore create --from-backup my-backup

# Schedule automatic backups
velero schedule create daily --schedule="0 2 * * *" --include-namespaces app-namespace
```

## Troubleshooting

### Backup location not available

Check the Velero logs:

```bash
kubectl logs -n velero -l app.kubernetes.io/name=velero
```

Verify S3 connectivity:

```bash
# Check BackupStorageLocation status
kubectl get backupstoragelocation -n velero -o yaml
```

### Invalid credentials

Verify the credentials secret:

```bash
kubectl get secret cloud-credentials -n velero -o yaml
```

## License

MIT

## Author

Your Name
