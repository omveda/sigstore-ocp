
# trusted-artifact-signer

A Helm chart for deploying Sigstore scaffold chart that is opinionated for OpenShift

![Version: 0.1.20](https://img.shields.io/badge/Version-0.1.20-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

## Overview

This wrapper chart builds on top of the [Scaffold](https://github.com/sigstore/helm-charts/tree/main/charts/scaffold)
chart from the Sigstore project to both simplify and satisfy the requirements for deployment within an OpenShift

If you have already read this document and want a quick no-fail path to installing a Sigstore stack with RH SSO,
follow [quick start](../../quick-start-with-keycloak.md)

The chart enhances the scaffold chart by taking care of the following:

* Provision Namespaces
* Configure `RoleBindings` to enable access to the `anyuid` SecurityContextConstraint
* Inject Fulcio root and Rekor signing keys

### Scaffold customization

Similar to any Helm dependency, values from the upstream `scaffold` chart can be customized by embedding the properties
within the `scaffold` property similar to the following:

```yaml
scaffold:
  fulcio:
    namespace:
      name: fulcio-system
      create: false
...
```

### Sample Implementation

#### Prerequisites

The following must be satisfied prior to deploying the sample implementation:

* Fulcio root CA certificate and signing keys
    * More information in [requirements-keys-certs.md](../../requirements-keys-certs.md)
* OpenID Token Issuer endpoint
    * Keycloak/RHSSO requirements can be followed and deployed in OpenShift with [keycloak-example.md](../../keycloak-example.md)

#### Update the values file

Helm values files are available in the examples directory that provides a baseline to work off of.
It can be customized based on an individual target environment.
Perform the following modifications to the [example values file](../../examples/values-sigstore-openshift.yaml)
to curate the deployment of the chart:

1. Modify the OIDC Issuer URL in the fulcio config section of the values file as necessary.

2. Perform any additional customizations as desired

### Installing the Chart

When logged in as an elevated OpenShift user, execute the following to install the chart referencing the
customized values file. The OPENSHIFT_APPS_SUBDOMAIN will be substituted in the values file with `envsubst` below:

```shell
OPENSHIFT_APPS_SUBDOMAIN=apps.$(oc get dns cluster -o jsonpath='{ .spec.baseDomain }') envsubst <  examples/values-sigstore-openshift.yaml | helm upgrade -i trusted-artifact-signer --debug charts/trusted-artifact-signer -n sigstore --create-namespace --values -
```

### Monitor Sigstore Components with Grafana

For real-time analytics through Grafana, refer to our [enable-grafana-monitoring.md](../../enable-grafana-monitoring.md) guide.

### Sign and/or verify artifacts!

Follow [this](../../sign-verify.md) to sign and/or verify artifacts.

## Requirements

Kubernetes: `>= 1.19.0-0`

| Repository | Name | Version |
|------------|------|---------|
| https://sigstore.github.io/helm-charts | scaffold(scaffold) | 0.6.32 |

## Values

| Key | Description | Type | Default |
|-----|-------------|------|---------|
| configs.clientserver.consoleDownload | This can only be enabled if the OpenShift CRD is registered. | bool | `true` |
| configs.clientserver.image.pullPolicy |  | string | `"IfNotPresent"` |
| configs.clientserver.image.registry |  | string | `"quay.io"` |
| configs.clientserver.image.repository |  | string | `"tdalton/serve"` |
| configs.clientserver.image.version |  | string | `"latest"` |
| configs.clientserver.name |  | string | `"tas-clients"` |
| configs.clientserver.namespace |  | string | `"trusted-artifact-signer-clientserver"` |
| configs.clientserver.namespace_create |  | bool | `true` |
| configs.clientserver.route | Whether to create the OpenShift route resource | bool | `true` |
| configs.cosign_deploy.enabled |  | bool | `false` |
| configs.cosign_deploy.image | Image containing the cosign binary as well as environment variables with the base domain injected. | object | `{"pullPolicy":"IfNotPresent","registry":"quay.io","repository":"redhat-user-workloads/rhtas-tenant/rhtas-cli-1-0-beta/cosign","version":"d866dfcf39487daf81cee2407244a7f6202466b9"}` |
| configs.cosign_deploy.name | Name of deployment | string | `"cosign"` |
| configs.cosign_deploy.namespace |  | string | `"cosign"` |
| configs.cosign_deploy.namespace_create |  | bool | `true` |
| configs.cosign_deploy.rolebindings | names for rolebindings to add clusterroles to cosign serviceaccounts. The names must match the serviceaccount names in the cosign namespace. | list | `["cosign"]` |
| configs.ctlog.namespace |  | string | `"ctlog-system"` |
| configs.ctlog.namespace_create |  | bool | `true` |
| configs.ctlog.rolebindings | Names for rolebindings to add clusterroles to ctlog serviceaccounts. The names must match the serviceaccount names in the ctlog namespace. | list | `["ctlog","ctlog-createtree","trusted-artifact-signer-ctlog-createctconfig"]` |
| configs.fulcio.clusterMonitoring.enabled |  | bool | `true` |
| configs.fulcio.clusterMonitoring.endpoints[0].interval |  | string | `"30s"` |
| configs.fulcio.clusterMonitoring.endpoints[0].port |  | string | `"2112-tcp"` |
| configs.fulcio.clusterMonitoring.endpoints[0].scheme |  | string | `"http"` |
| configs.fulcio.namespace |  | string | `"fulcio-system"` |
| configs.fulcio.namespace_create |  | bool | `true` |
| configs.fulcio.rolebindings | Names for rolebindings to add clusterroles to fulcio serviceaccounts. The names must match the serviceaccount names in the fulcio namespace. | list | `["fulcio-createcerts","fulcio-server"]` |
| configs.fulcio.server.secret.name |  | string | `""` |
| configs.fulcio.server.secret.password | password to decrypt the signing key | string | `""` |
| configs.fulcio.server.secret.private_key | a PEM-encoded encrypted signing key | string | `""` |
| configs.fulcio.server.secret.private_key_file | file containing a PEM-encoded encrypted signing key | string | `""` |
| configs.fulcio.server.secret.public_key | signer public key | string | `""` |
| configs.fulcio.server.secret.public_key_file | file containing signer public key | string | `""` |
| configs.fulcio.server.secret.root_cert | fulcio root certificate authority (CA) | string | `""` |
| configs.fulcio.server.secret.root_cert_file | file containing fulcio root certificate authority (CA) | string | `""` |
| configs.rekor.clusterMonitoring.enabled |  | bool | `true` |
| configs.rekor.clusterMonitoring.endpoints[0].interval |  | string | `"30s"` |
| configs.rekor.clusterMonitoring.endpoints[0].port |  | string | `"2112-tcp"` |
| configs.rekor.clusterMonitoring.endpoints[0].scheme |  | string | `"http"` |
| configs.rekor.namespace |  | string | `"rekor-system"` |
| configs.rekor.namespace_create |  | bool | `true` |
| configs.rekor.rolebindings | names for rolebindings to add clusterroles to rekor serviceaccounts. The names must match the serviceaccount names in the rekor namespace. | list | `["rekor-redis","rekor-server","trusted-artifact-signer-rekor-createtree"]` |
| configs.rekor.signer | Signer holds secret that contains the private key used to sign entries and the tree head of the transparency log When this section is left out, scaffold.rekor creates the secret and key. | object | `{"secret":{"name":"","private_key":"","private_key_file":""}}` |
| configs.rekor.signer.secret.name | Name of the secret to create with the private key data. This name must match the value in scaffold.rekor.server.signer.signerFileSecretOptions.secretName. | string | `""` |
| configs.rekor.signer.secret.private_key | Private encrypted signing key | string | `""` |
| configs.rekor.signer.secret.private_key_file | File containing a private encrypted signing key | string | `""` |
| configs.trillian.namespace |  | string | `"trillian-system"` |
| configs.trillian.namespace_create |  | bool | `true` |
| configs.trillian.rolebindings | names for rolebindings to add clusterroles to trillian serviceaccounts. The names must match the serviceaccount names in the trillian namespace. | list | `["trillian-logserver","trillian-logsigner","trillian-mysql"]` |
| configs.tuf.namespace |  | string | `"tuf-system"` |
| configs.tuf.namespace_create |  | bool | `true` |
| configs.tuf.rolebindings | names for rolebindings to add clusterroles to tuf serviceaccounts. The names must match the serviceaccount names in the tuf namespace. | list | `["tuf","tuf-secret-copy-job"]` |
| global.appsSubdomain | DNS name to generate environment variables and consoleCLIDownload urls. By default, in OpenShift, the value for this is apps.$(oc get dns cluster -o jsonpath='{ .spec.baseDomain }') | string | `""` |
| rbac.clusterrole | clusterrole to be added to sigstore component serviceaccounts. | string | `"system:openshift:scc:anyuid"` |
| scaffold.copySecretJob.backoffLimit |  | int | `1000` |
| scaffold.copySecretJob.enabled |  | bool | `true` |
| scaffold.copySecretJob.imagePullPolicy |  | string | `"IfNotPresent"` |
| scaffold.copySecretJob.name |  | string | `"copy-secrets-job"` |
| scaffold.copySecretJob.registry |  | string | `"registry.redhat.io"` |
| scaffold.copySecretJob.repository |  | string | `"openshift4/ose-cli"` |
| scaffold.copySecretJob.serviceaccount |  | string | `"tuf-secret-copy-job"` |
| scaffold.copySecretJob.version |  | string | `"latest"` |
| scaffold.ctlog.createcerts.fullnameOverride |  | string | `"ctlog-createcerts"` |
| scaffold.ctlog.createctconfig.backoffLimit |  | int | `30` |
| scaffold.ctlog.createctconfig.enabled |  | bool | `true` |
| scaffold.ctlog.createctconfig.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.ctlog.createctconfig.image.registry |  | string | `"quay.io"` |
| scaffold.ctlog.createctconfig.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/createctconfig"` |
| scaffold.ctlog.createctconfig.image.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.ctlog.createctconfig.initContainerImage.curl.imagePullPolicy |  | string | `"IfNotPresent"` |
| scaffold.ctlog.createctconfig.initContainerImage.curl.registry |  | string | `"registry.access.redhat.com"` |
| scaffold.ctlog.createctconfig.initContainerImage.curl.repository |  | string | `"ubi9/ubi-minimal"` |
| scaffold.ctlog.createctconfig.initContainerImage.curl.version |  | string | `"latest"` |
| scaffold.ctlog.createtree.displayName |  | string | `"ctlog-tree"` |
| scaffold.ctlog.createtree.fullnameOverride |  | string | `"ctlog-createtree"` |
| scaffold.ctlog.createtree.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.ctlog.createtree.image.registry |  | string | `"quay.io"` |
| scaffold.ctlog.createtree.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/createtree"` |
| scaffold.ctlog.createtree.image.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.ctlog.enabled |  | bool | `true` |
| scaffold.ctlog.forceNamespace |  | string | `"ctlog-system"` |
| scaffold.ctlog.fullnameOverride |  | string | `"ctlog"` |
| scaffold.ctlog.namespace.create |  | bool | `false` |
| scaffold.ctlog.namespace.name |  | string | `"ctlog-system"` |
| scaffold.ctlog.server.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.ctlog.server.image.registry |  | string | `"quay.io"` |
| scaffold.ctlog.server.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/ct-server"` |
| scaffold.ctlog.server.image.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.fulcio.createcerts.enabled |  | bool | `false` |
| scaffold.fulcio.createcerts.fullnameOverride |  | string | `"fulcio-createcerts"` |
| scaffold.fulcio.createcerts.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.fulcio.createcerts.image.registry |  | string | `"quay.io"` |
| scaffold.fulcio.createcerts.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/createcerts"` |
| scaffold.fulcio.createcerts.image.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.fulcio.ctlog.createctconfig.logPrefix |  | string | `"sigstorescaffolding"` |
| scaffold.fulcio.ctlog.enabled |  | bool | `false` |
| scaffold.fulcio.enabled |  | bool | `true` |
| scaffold.fulcio.forceNamespace |  | string | `"fulcio-system"` |
| scaffold.fulcio.namespace.create |  | bool | `false` |
| scaffold.fulcio.namespace.name |  | string | `"fulcio-system"` |
| scaffold.fulcio.server.fullnameOverride |  | string | `"fulcio-server"` |
| scaffold.fulcio.server.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.fulcio.server.image.registry |  | string | `"quay.io"` |
| scaffold.fulcio.server.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/fulcio"` |
| scaffold.fulcio.server.image.version |  | string | `"30bf1017fa75195c051d92bf3b896c2f3f8016c8"` |
| scaffold.fulcio.server.ingress.http.annotations."route.openshift.io/termination" |  | string | `"edge"` |
| scaffold.fulcio.server.ingress.http.className |  | string | `""` |
| scaffold.fulcio.server.ingress.http.enabled |  | bool | `true` |
| scaffold.fulcio.server.ingress.http.hosts[0].host |  | string | `"fulcio.appsSubdomain"` |
| scaffold.fulcio.server.ingress.http.hosts[0].path |  | string | `"/"` |
| scaffold.fulcio.server.secret |  | string | `"fulcio-secret-rh"` |
| scaffold.rekor.backfillredis.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.rekor.backfillredis.image.registry |  | string | `"quay.io"` |
| scaffold.rekor.backfillredis.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/backfill-redis"` |
| scaffold.rekor.backfillredis.image.version |  | string | `"c1953fbfae65154ef43e6d4da01f9d0a5c44be11"` |
| scaffold.rekor.createtree.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.rekor.createtree.image.registry |  | string | `"quay.io"` |
| scaffold.rekor.createtree.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/createtree"` |
| scaffold.rekor.createtree.image.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.rekor.enabled |  | bool | `true` |
| scaffold.rekor.forceNamespace |  | string | `"rekor-system"` |
| scaffold.rekor.fullnameOverride |  | string | `"rekor"` |
| scaffold.rekor.namespace.create |  | bool | `false` |
| scaffold.rekor.namespace.name |  | string | `"rekor-system"` |
| scaffold.rekor.redis.fullnameOverride |  | string | `"rekor-redis"` |
| scaffold.rekor.server.fullnameOverride |  | string | `"rekor-server"` |
| scaffold.rekor.server.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.rekor.server.image.registry |  | string | `"quay.io"` |
| scaffold.rekor.server.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/rekor-server"` |
| scaffold.rekor.server.image.version |  | string | `"c1953fbfae65154ef43e6d4da01f9d0a5c44be11"` |
| scaffold.rekor.server.ingress.annotations."route.openshift.io/termination" |  | string | `"edge"` |
| scaffold.rekor.server.ingress.className |  | string | `""` |
| scaffold.rekor.server.ingress.hosts[0].host |  | string | `"rekor.appsSubdomain"` |
| scaffold.rekor.server.ingress.hosts[0].path |  | string | `"/"` |
| scaffold.rekor.server.signer |  | string | `"/key/private"` |
| scaffold.rekor.server.signerFileSecretOptions.privateKeySecretKey |  | string | `"private"` |
| scaffold.rekor.server.signerFileSecretOptions.secretMountPath |  | string | `"/key"` |
| scaffold.rekor.server.signerFileSecretOptions.secretMountSubPath |  | string | `"private"` |
| scaffold.rekor.server.signerFileSecretOptions.secretName |  | string | `"rekor-private-key"` |
| scaffold.rekor.trillian.enabled |  | bool | `false` |
| scaffold.trillian.createdb.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.trillian.createdb.image.registry |  | string | `"quay.io"` |
| scaffold.trillian.createdb.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/createdb"` |
| scaffold.trillian.createdb.image.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.trillian.enabled |  | bool | `true` |
| scaffold.trillian.forceNamespace |  | string | `"trillian-system"` |
| scaffold.trillian.fullnameOverride |  | string | `"trillian"` |
| scaffold.trillian.initContainerImage.curl.imagePullPolicy |  | string | `"IfNotPresent"` |
| scaffold.trillian.initContainerImage.curl.registry |  | string | `"registry.access.redhat.com"` |
| scaffold.trillian.initContainerImage.curl.repository |  | string | `"ubi9/ubi-minimal"` |
| scaffold.trillian.initContainerImage.curl.version |  | string | `"latest"` |
| scaffold.trillian.initContainerImage.netcat.registry |  | string | `"quay.io"` |
| scaffold.trillian.initContainerImage.netcat.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/trillian-netcat"` |
| scaffold.trillian.initContainerImage.netcat.version |  | string | `"da426b96919bcc346e9a927d8e6fa96b9a5cb240"` |
| scaffold.trillian.logServer.fullnameOverride |  | string | `"trillian-logserver"` |
| scaffold.trillian.logServer.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.trillian.logServer.image.registry |  | string | `"quay.io"` |
| scaffold.trillian.logServer.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/trillian-logserver"` |
| scaffold.trillian.logServer.image.version |  | string | `"da426b96919bcc346e9a927d8e6fa96b9a5cb240"` |
| scaffold.trillian.logServer.name |  | string | `"trillian-logserver"` |
| scaffold.trillian.logServer.portHTTP |  | int | `8090` |
| scaffold.trillian.logServer.portRPC |  | int | `8091` |
| scaffold.trillian.logSigner.fullnameOverride |  | string | `"trillian-logsigner"` |
| scaffold.trillian.logSigner.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.trillian.logSigner.image.registry |  | string | `"quay.io"` |
| scaffold.trillian.logSigner.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/trillian-logsigner"` |
| scaffold.trillian.logSigner.image.version |  | string | `"da426b96919bcc346e9a927d8e6fa96b9a5cb240"` |
| scaffold.trillian.logSigner.name |  | string | `"trillian-logsigner"` |
| scaffold.trillian.mysql.args |  | list | `[]` |
| scaffold.trillian.mysql.fullnameOverride |  | string | `"trillian-mysql"` |
| scaffold.trillian.mysql.gcp.scaffoldSQLProxy.registry |  | string | `"quay.io"` |
| scaffold.trillian.mysql.gcp.scaffoldSQLProxy.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/cloudsqlproxy"` |
| scaffold.trillian.mysql.gcp.scaffoldSQLProxy.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.trillian.mysql.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.trillian.mysql.image.registry |  | string | `"quay.io"` |
| scaffold.trillian.mysql.image.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/trillian-database"` |
| scaffold.trillian.mysql.image.version |  | string | `"da426b96919bcc346e9a927d8e6fa96b9a5cb240"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[0] |  | string | `"mysqladmin"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[1] |  | string | `"ping"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[2] |  | string | `"-h"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[3] |  | string | `"localhost"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[4] |  | string | `"-u"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[5] |  | string | `"$(MYSQL_USER)"` |
| scaffold.trillian.mysql.livenessProbe.exec.command[6] |  | string | `"-p$(MYSQL_PASSWORD)"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[0] |  | string | `"mysqladmin"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[1] |  | string | `"ping"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[2] |  | string | `"-h"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[3] |  | string | `"localhost"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[4] |  | string | `"-u"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[5] |  | string | `"$(MYSQL_USER)"` |
| scaffold.trillian.mysql.readinessProbe.exec.command[6] |  | string | `"-p$(MYSQL_PASSWORD)"` |
| scaffold.trillian.mysql.securityContext.fsGroup |  | int | `0` |
| scaffold.trillian.namespace.create |  | bool | `false` |
| scaffold.trillian.namespace.name |  | string | `"trillian-system"` |
| scaffold.trillian.redis.args[0] |  | string | `"/usr/bin/run-redis"` |
| scaffold.trillian.redis.args[1] |  | string | `"--bind"` |
| scaffold.trillian.redis.args[2] |  | string | `"0.0.0.0"` |
| scaffold.trillian.redis.args[3] |  | string | `"--appendonly"` |
| scaffold.trillian.redis.args[4] |  | string | `"yes"` |
| scaffold.trillian.redis.image.pullPolicy |  | string | `"IfNotPresent"` |
| scaffold.trillian.redis.image.registry |  | string | `"registry.redhat.io"` |
| scaffold.trillian.redis.image.repository |  | string | `"rhel9/redis-6"` |
| scaffold.trillian.redis.image.version |  | string | `"sha256:031a5a63611e1e6a9fec47492a32347417263b79ad3b63bcee72fc7d02d64c94"` |
| scaffold.tsa.enabled |  | bool | `false` |
| scaffold.tsa.forceNamespace |  | string | `"tsa-system"` |
| scaffold.tsa.namespace.create |  | bool | `false` |
| scaffold.tsa.namespace.name |  | string | `"tsa-system"` |
| scaffold.tsa.server.fullnameOverride |  | string | `"tsa-server"` |
| scaffold.tuf.deployment.registry |  | string | `"quay.io"` |
| scaffold.tuf.deployment.repository |  | string | `"redhat-user-workloads/rhtas-tenant/rhtas-stack-1-0-beta/tuf-server"` |
| scaffold.tuf.deployment.version |  | string | `"80894dcf5afe60bacfe0d07d23c536cf4ccb556d"` |
| scaffold.tuf.enabled |  | bool | `true` |
| scaffold.tuf.forceNamespace |  | string | `"tuf-system"` |
| scaffold.tuf.fullnameOverride |  | string | `"tuf"` |
| scaffold.tuf.ingress.annotations."route.openshift.io/termination" |  | string | `"edge"` |
| scaffold.tuf.ingress.className |  | string | `""` |
| scaffold.tuf.ingress.http.hosts[0].host |  | string | `"tuf.appsSubdomain"` |
| scaffold.tuf.ingress.http.hosts[0].path |  | string | `"/"` |
| scaffold.tuf.namespace.create |  | bool | `false` |
| scaffold.tuf.namespace.name |  | string | `"tuf-system"` |
| scaffold.tuf.secrets.ctlog.name |  | string | `"ctlog-public-key"` |
| scaffold.tuf.secrets.ctlog.path |  | string | `"ctfe.pub"` |
| scaffold.tuf.secrets.fulcio.name |  | string | `"fulcio-secret-rh"` |
| scaffold.tuf.secrets.fulcio.path |  | string | `"fulcio-cert"` |
| scaffold.tuf.secrets.rekor.name |  | string | `"rekor-public-key"` |
| scaffold.tuf.secrets.rekor.path |  | string | `"rekor-pubkey"` |

