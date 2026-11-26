# Checkmk deployment security and migration

This legacy deployment is unchanged at the application image, storage, service and container configuration levels. Test in an isolated namespace/cluster before applying the updated manifests. Ingress v1 requires Kubernetes 1.19 or newer.

## Credentials (required before a fresh install)

The deployment still expects the existing Secret names and keys in namespace checkmk:
- checkmk: an Opaque Secret containing the key password.
- pull-secret: a kubernetes.io/dockerconfigjson Secret containing the key .dockerconfigjson, if registry authentication is required by the configured deployment.

Provision these through your approved secrets manager or secure deployment process before creating the workload. Do not commit real or base64-encoded credentials to Git. For an existing installation, retain existing Secret objects until replacement credentials are provisioned and verified. Do not use GitOps pruning to delete these objects during migration; transfer their ownership to your approved secret-management workflow first. A stored site password may only be used for initial setup; rotate an existing site password through the supported Checkmk procedure and verify login. Rotate any real previously committed credentials and follow your organization's history-remediation process.

## Monitoring compatibility

The ClusterRole no longer grants get/list access to Secrets. Confirm service discovery, Kubernetes checks and metrics still function in staging. Other monitoring permissions remain unchanged for compatibility; assess node proxy, wildcard metrics permissions and cluster-wide read scope separately against the deployed Checkmk version.

The NetworkPolicy now targets the deployment's app label in the correct namespace. It permits the original TCP port set from all sources and leaves egress unchanged. Check ingress controller, direct NodePort access, service discovery and monitoring traffic before rollout. Source restriction, TLS termination and NodePort removal depend on the actual network topology and are not included here.

## Validation and rollout

- Back up Checkmk configuration and persistent data, and capture the existing deployment configuration securely.
- Provision the required Secrets without exposing their values.
- Run server-side dry-run validation of the manifests against a staging Kubernetes API.
- Confirm pod startup, web login, registry access, ingress routing, persistence and all monitoring checks.
- Confirm the monitoring service account cannot read Secrets (unless another binding grants it) and confirm NetworkPolicy enforcement by the cluster CNI.
- Apply changes only after staging passes and an administrator approves the change. Revert the configuration changes if monitoring or connectivity regresses; do not restore exposed credentials.
- The legacy Checkmk image is retained to avoid untested application upgrades. Plan a supported-version migration separately.
