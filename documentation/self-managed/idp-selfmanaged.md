# Self-Managed Identity Data Platform Deployment

This document serves as a guide for deploying Radiant Logic's Identity Data Platform with Helm charts on a Kubernetes cluster. It outlines the prerequisites and provides detailed, step-by-step instructions for the deployment.

## Overview

You can deploy the Identity Data Platform on the following Kubernetes platforms:

- Amazon Elastic Kubernetes Service (EKS)
- Microsoft Azure Kubernetes Service (AKS)

Helm is the only supported installation method (`helm upgrade --install`).

| Application | Chart | Chart version |
| :-- | :-- | :--: |
| Shared Services | `ida-shared-helm` | 3.5.3 |
| Identity Data Platform | `ido-helm` | 3.1.0 |

Ensure that you specify these versions when running the installation and update commands in this document.


## Design

The deployment consists of two charts:

1. **Shared Services** (`ida-shared-helm`): authentication (Keycloak), the API gateway (APISIX) and their database. Only one instance of Shared Services is required per cluster.
2. **Identity Data Platform** (`ido-helm`): the Identity Data Platform stack, including the bundled Identity Data Management. Deploy it in its own namespace.

All external traffic for the portal and for authentication goes through the Shared Services ingress. Identity Data Management is exposed on a second host name.

> **Note**
> Before deploying these charts, you need to install the Kubernetes operators listed in [Operators](#operators). The installation details are covered in [step 1](#1-install-the-operators).


## Prerequisites

### Tools and access

Ensure that you have the following:

- An EKS or AKS cluster running Kubernetes **1.34** or higher. Check the [Kubernetes releases](https://kubernetes.io/releases/) page to ensure that your version is still maintained.
- [Helm](https://helm.sh/) **3.x**. Helm 4 has not been tested and introduces breaking changes.
- [kubectl](https://kubernetes.io/docs/reference/kubectl/), within one minor version of your cluster, configured to access your cluster.

The following items are provided to you by Radiant Logic during onboarding:

- An Identity Data Platform license file.
- An Identity Data Management license key.
- Container registry access and image pull credentials.
- Credentials for the configuration Git repository (GitHub App ID, installation ID and private key).

You also need:

- **DNS**: two host names that you can point at the cluster's ingress load balancer:
  - `<APP_ENDPOINT>` for the portal and authentication, for example `identity-data-platform.example.com`.
  - `<DATA_MANAGEMENT_ENDPOINT>` for Identity Data Management, for example `identity-data-management.example.com`.
- **TLS certificate**: a certificate valid for both host names, stored as a Kubernetes TLS secret.
- **SMTP server**: the host and port of your SMTP relay, used to send verification emails.
- **Object storage**: an Amazon S3 bucket (EKS) or an Azure Blob Storage account and container (AKS). See [Object storage](#object-storage).
- **LLM provider** (optional): access to Amazon Bedrock (EKS) or Azure OpenAI (AKS) models for AI features. See [step 7](#7-configure-ai-features).

### Password requirements

Choose the passwords and keys of the values files according to these rules. They are compatible with FIPS mode.

| Credential | Requirements |
| :-- | :-- |
| Technical administrator (`analytics.technicalAdmin.password`) and other user passwords | At least 32 characters, with at least one uppercase letter, one lowercase letter, one digit and one special character. This satisfies the authentication password policy (16 characters minimum, mixed case, digit and special character) |
| Technical secrets: Identity Data Management root password, admin API key and service account password | At least 32 characters, letters and digits only (`a-z`, `A-Z`, `0-9`). Do not use special characters: some, such as `:`, are not accepted by Identity Data Management |

For example, to generate a 32-character alphanumeric secret:

```bash
openssl rand -base64 48 | tr -dc 'A-Za-z0-9' | head -c 32
```

### Operators

The following operators must be installed once per cluster before Shared Services. If Identity Analytics is already deployed on the cluster, CloudNativePG and the ingress controller are already present. Only the Flink operator needs to be added.

> **Note**
> **Ingress controller**: the Identity Data Platform works with any Kubernetes ingress controller. This guide uses [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) as an example configuration. If your organization uses another controller (for example a cloud load balancer controller), check with your Kubernetes or network administrator and adapt the ingress class and annotations in [step 2](#2-deploy-shared-services-chart) and [step 5](#5-expose-identity-data-management-and-configure-dns).

| Operator | Helm chart | Chart version | Purpose | Official documentation |
| :-- | :-- | :--: | :-- | :-- |
| CloudNativePG | `cnpg/cloudnative-pg` | 0.29.0 | Manages the PostgreSQL databases of Shared Services and the Identity Data Platform | [Installation and upgrades](https://cloudnative-pg.io/documentation/current/installation_upgrade/), [Helm chart](https://github.com/cloudnative-pg/charts) |
| Flink Kubernetes Operator | `flink-operator-repo/flink-kubernetes-operator` | 1.15.0 | Runs the graph pipeline | [Helm installation](https://nightlies.apache.org/flink/flink-kubernetes-operator-docs-release-1.15/docs/operations/helm/) |
| Ingress controller | for example `ingress-nginx/ingress-nginx` | — | Exposes the portal, authentication and Identity Data Management | [ingress-nginx installation](https://kubernetes.github.io/ingress-nginx/deploy/) |

### Storage requirements

The charts use block storage for their persistent volumes and object storage for pipeline data.

#### Storage class

A default storage class backed by SSD block storage with volume expansion enabled is required. Volumes must be created in the availability zone of the node that consumes them (`WaitForFirstConsumer`).

**EKS**: install the [Amazon EBS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) add-on, then create a `gp3` storage class:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

**AKS**: the Azure Disk CSI driver is built in. Create a Premium SSD storage class:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: premium-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

> **Note**
> Only one storage class can be the default. On AKS, remove the default annotation from the built-in `default` and `managed-csi` classes, or set `global.storageClass` in both values files instead of relying on the default.

#### Persistent volumes

The default volume sizes are listed below. You can change them in the values files (see [step 8](#8-update-resource-configuration)).

| Component | Default size |
| :-- | :-- |
| Shared Services: Keycloak database (data + WAL) | 32Gi + 32Gi |
| Shared Services: etcd | 32Gi |
| Analytics database (data + WAL) | 128Gi + 32Gi |
| Observation database (data + WAL) | 128Gi + 32Gi |
| Portal logs | 16Gi |
| Other components | 56Gi |
| Identity Data Management | 50Gi recommended |

Plan for about **550Gi** of block storage in total.

#### Object storage

The graph pipeline and data ingestion store their working data in object storage.

- **EKS**: an S3 bucket in the cluster's region. The Identity Data Platform needs read, write, list and delete access to the bucket.
- **AKS**: an Azure Storage account with a Blob container. The Identity Data Platform needs read, write, list and delete access to the container.

This guide uses access keys stored in a Kubernetes secret (see [step 3](#3-create-the-namespace-and-secrets)).

> **Note**
> Key-less access is also supported: an IAM role for service accounts on EKS (`objectStorage.s3.credentials.role: true`), or Azure workload identity on AKS (`objectStorage.azure.credentials.mode: workloadIdentity`). Your Radiant Logic solutions engineer can guide you through this setup.

### Cluster sizing

The table below gives an evaluation minimum, derived from the default resource requests of both charts, and the reference configuration Radiant Logic tests against. Your Radiant Logic solutions engineer will help you size a production cluster for your data volume.

| | Evaluation minimum | Tested reference |
| :-- | :-- | :-- |
| Total resources | ~20 vCPU / 48 GiB requested | ~66–70 vCPU / 260–276 GiB |
| EKS nodes | 3 × `m5.2xlarge` (8 vCPU / 32 GiB) | 1 × `c5.large`, 3 × `m5.xlarge`, 7 × `m5.2xlarge` |
| AKS nodes | 3 × `Standard_D8s_v3` (8 vCPU / 32 GiB) | 1 × `Standard_F2s_v2`, 2 × `Standard_D4s_v3`, 7 × `Standard_D8s_v3` |
| Block storage | ~550Gi | ~550Gi |

---

## Steps for Deployment

### 1. Install the operators

Follow these steps:

1. **Set kube-context**:

   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

2. **Create Storage Class** described in [Storage class](#storage-class):

   ```bash
   kubectl apply -f storage-class.yaml
   ```

3. **Install CloudNativePG**: for advanced configuration, see the official [CloudNativePG installation guide](https://cloudnative-pg.io/documentation/current/installation_upgrade/) and the [CloudNativePG Helm chart](https://github.com/cloudnative-pg/charts). Create a file named `cnpg-values.yaml`:

   ```yaml
   fullnameOverride: cnpg
   crds:
     create: true
   config:
     create: true
     name: cnpg-controller-manager-config
     data:
       INHERITED_LABELS: app.kubernetes.io/*, radiantlogic.io/*
       INHERITED_ANNOTATIONS: meta.helm.sh/*, helm.sh/*, prometheus.io/*
   ```

   ```bash
   helm repo add cnpg https://cloudnative-pg.github.io/charts
   helm repo update
   helm upgrade --install cnpg cnpg/cloudnative-pg \
     --namespace <OPERATORS_NAMESPACE> \
     --create-namespace \
     --version 0.29.0 \
     --values cnpg-values.yaml \
     --wait
   ```

4. **Install Flink Kubernetes Operator**: for advanced configuration, see the official [Flink Kubernetes Operator Helm installation guide](https://nightlies.apache.org/flink/flink-kubernetes-operator-docs-release-1.15/docs/operations/helm/). Create a file named `flink-operator-values.yaml`:

   ```yaml
   webhook:
     create: false
   ```

   ```bash
   helm repo add flink-operator-repo https://downloads.apache.org/flink/flink-kubernetes-operator-1.15.0/
   helm repo update
   helm upgrade --install flink-kubernetes-operator flink-operator-repo/flink-kubernetes-operator \
     --namespace <OPERATORS_NAMESPACE> \
     --create-namespace \
     --version 1.15.0 \
     --values flink-operator-values.yaml \
     --wait
   ```

5. **Install Ingress Controller**: the example below installs ingress-nginx. For advanced configuration, or to use another controller, see the official [ingress-nginx installation guide](https://kubernetes.github.io/ingress-nginx/deploy/) or your controller's documentation:

   ```bash
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
     --namespace ingress-nginx \
     --create-namespace \
     --set controller.service.type=LoadBalancer \
     --wait
   ```

   On AKS, also add `--set-string controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz`.

6. **Verify Installation**:

   ```bash
   kubectl get pods -n <OPERATORS_NAMESPACE>
   kubectl get crds | grep -E "postgresql.cnpg.io|flink.apache.org"
   kubectl get service -n ingress-nginx ingress-nginx-controller
   ```

   Expected CRDs include:

   - `clusters.postgresql.cnpg.io`
   - `flinkdeployments.flink.apache.org`

   Note the external address of your ingress controller (here, the `ingress-nginx-controller` service). You need it to configure DNS.

### 2. Deploy Shared Services Chart

> **Note**
> If Shared Services 3.5.3 is already deployed on the cluster for Identity Analytics, skip this step. Use its namespace as `<SHARED_NAMESPACE>` and its authentication endpoint as `<APP_ENDPOINT>` in the next steps.

Deploy Shared Services by following [Deploy Shared Services Chart](https://developer.radiantlogic.com/ia/descartes/igrc-platform/self-managed/#2-deploy-shared-services-chart) in the Identity Analytics guide, with these settings:

- `<SHARED_CHART_VERSION>`: `3.5.3`
- `global.externalDns`: `<APP_ENDPOINT>`

Before running the install command, update the `ingress` section of `shared-minimal.values.yaml` as follows. This allows large uploads and long-running requests, and enables TLS:

```yaml
ingress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "0"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "180"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "180"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  tls:
    - hosts:
        - <APP_ENDPOINT>
      secretName: <TLS_SECRET_NAME>
```

> **Note**
> The TLS secret must exist in `<SHARED_NAMESPACE>`. Create it from your certificate with `kubectl create secret tls <TLS_SECRET_NAME> --cert=<CERT_FILE> --key=<KEY_FILE> --namespace <SHARED_NAMESPACE>`.

### 3. Create the namespace and secrets

Follow these steps:

1. **Create Namespace**:

   ```bash
   kubectl create namespace <DATA_PLATFORM_NAMESPACE>
   ```

2. **Create Image Pull Secret** with the registry credentials provided by Radiant Logic:

   ```bash
   kubectl create secret docker-registry data-platform-registry-credentials \
     --docker-server=docker.io \
     --docker-username=<REGISTRY_USERNAME> \
     --docker-password=<REGISTRY_PASSWORD> \
     --namespace <DATA_PLATFORM_NAMESPACE>
   ```

3. **Create Object Storage Secret**: the Identity Data Platform reads the access keys of your bucket or container from this secret. Run only the command for your platform:
   - **EKS**: use the access key of an IAM user that has access to the S3 bucket.
   - **AKS**: use one of the access keys of the Azure Storage account.

   EKS:

   ```bash
   kubectl create secret generic object-storage-credentials \
     --from-literal=AWS_ACCESS_KEY_ID=<ACCESS_KEY_ID> \
     --from-literal=AWS_SECRET_ACCESS_KEY=<SECRET_ACCESS_KEY> \
     --namespace <DATA_PLATFORM_NAMESPACE>
   ```

   AKS:

   ```bash
   kubectl create secret generic object-storage-credentials \
     --from-literal=AZURE_STORAGE_ACCOUNT_KEY=<STORAGE_ACCOUNT_KEY> \
     --namespace <DATA_PLATFORM_NAMESPACE>
   ```

4. **Create Git Repository Secret** with the GitHub App private key provided by Radiant Logic:

   ```bash
   kubectl create secret generic git-credentials \
     --from-file=privateKey=<GITHUB_APP_PRIVATE_KEY_FILE> \
     --namespace <DATA_PLATFORM_NAMESPACE>
   ```

5. **Create TLS Secret** for Identity Data Management:

   ```bash
   kubectl create secret tls <TLS_SECRET_NAME> \
     --cert=<CERT_FILE> \
     --key=<KEY_FILE> \
     --namespace <DATA_PLATFORM_NAMESPACE>
   ```

### 4. Deploy the Identity Data Platform Chart

Ensure that you have the following before installing:

- Shared Services installed and running
- the license file and the Identity Data Management license key
- the GitHub App ID and installation ID
- your SMTP settings
- the object storage bucket or container name

Follow these steps for the installation:

1. **Set kube-context**.
2. **Create Configuration File**. Create a file named `data-platform.values.yaml`:

   ```yaml
   global:
     externalProtocol: "https"
     externalDns: "<APP_ENDPOINT>"       # This guide uses the Shared Services host for simplicity
     pathPrefix: "/<PATH_PREFIX>"       # URL path of the portal; must differ from any Identity Analytics path prefix
   imageCredentials:
     - existingSecret: data-platform-registry-credentials
   analytics:
     technicalAdmin:
       email: "<ADMIN_EMAIL>"
       password: ""                     # See Password requirements
   git:
     default:
       authType: githubapp
       appId: "<GITHUB_APP_ID>"
       installationId: "<GITHUB_INSTALLATION_ID>"
       existingSecret: git-credentials
   smtp:
     enabled: true
     host: "<SMTP_HOST>"
     port: 25
     from:
       mail: "<SENDER_EMAIL>"
       displayName: "Identity Data Platform"
     auth:
       enabled: false
       username: ""
       password: ""
     ssl: false
     startTls: false
   keycloakSettings:
     namespace: "<SHARED_NAMESPACE>"
     externalDns: "<APP_ENDPOINT>"
   apisix:
     namespace: "<SHARED_NAMESPACE>"
   iddm:
     externalDns: "<DATA_MANAGEMENT_ENDPOINT>"
     fid:
       license: "<DATA_MANAGEMENT_LICENSE_KEY>"
       rootPassword: ""                 # See Password requirements
       adminApiKey: ""                  # See Password requirements
     restServiceAccount:
       password: ""                     # See Password requirements
   aida:
     enabled: false                     # See step 7 to enable AI features
   ```

   Then append **one** of the two `objectStorage` blocks below to the end of `data-platform.values.yaml`, depending on your platform. It tells the Identity Data Platform which bucket or container to use and which secret holds its access keys (created in [step 3](#3-create-the-namespace-and-secrets)).

   EKS:

   ```yaml
   objectStorage:
     provider: s3
     bucket: "<BUCKET_NAME>"
     s3:
       region: "<AWS_REGION>"
       credentials:
         role: false
         existingSecret: object-storage-credentials
   ```

   AKS:

   ```yaml
   objectStorage:
     provider: azure
     bucket: "<CONTAINER_NAME>"
     azure:
       accountName: "<STORAGE_ACCOUNT_NAME>"
       scheme: wasbs                    # Use abfss if the account has hierarchical namespace enabled
       credentials:
         mode: storageKey
         existingSecret: object-storage-credentials
   ```

3. **Install Identity Data Platform**:

   ```bash
   helm upgrade --install data-platform \
     oci://docker.io/radiantone/ido-helm \
     --namespace <DATA_PLATFORM_NAMESPACE> \
     --version 3.1.0 \
     --set-file analytics.licenseFile=<LICENSE_FILE_PATH> \
     --values data-platform.values.yaml \
     --timeout 30m \
     --wait
   ```

4. **Verify Deployment**:

   ```bash
   kubectl get pods --namespace <DATA_PLATFORM_NAMESPACE>
   kubectl rollout status deployment/portal --namespace <DATA_PLATFORM_NAMESPACE> --timeout=20m
   ```

   The first start can take up to 20 minutes while the databases are initialized and the configuration is loaded from Git.

### 5. Expose Identity Data Management and configure DNS

Follow these steps:

1. **Create Identity Data Management Ingress**. Create a file named `data-management-ingress.yaml`:

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: identity-data-management
     namespace: <DATA_PLATFORM_NAMESPACE>
     annotations:
       nginx.ingress.kubernetes.io/proxy-body-size: "0"
       nginx.ingress.kubernetes.io/proxy-read-timeout: "180"
       nginx.ingress.kubernetes.io/proxy-send-timeout: "180"
       nginx.ingress.kubernetes.io/ssl-redirect: "true"
   spec:
     ingressClassName: nginx
     tls:
       - hosts:
           - <DATA_MANAGEMENT_ENDPOINT>
         secretName: <TLS_SECRET_NAME>
     rules:
       - host: <DATA_MANAGEMENT_ENDPOINT>
         http:
           paths:
             - path: /adap
               pathType: Prefix
               backend:
                 service:
                   name: fid-app
                   port:
                     number: 8089
             - path: /cp
               pathType: Prefix
               backend:
                 service:
                   name: fid-ext
                   port:
                     number: 7070
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: iddm-proxy-service
                   port:
                     number: 80
   ```

   ```bash
   kubectl apply -f data-management-ingress.yaml
   ```

2. **Configure DNS**. Point `<APP_ENDPOINT>` and `<DATA_MANAGEMENT_ENDPOINT>` at the external address of your ingress controller (with ingress-nginx, the `ingress-nginx-controller` service):
   - **EKS**: the address is a host name. Create CNAME records.
   - **AKS**: the address is an IP address. Create A records.

### 6. Access the Identity Data Platform

Follow these steps:

1. **Open the Portal**: open `https://<APP_ENDPOINT>/<PATH_PREFIX>` in a browser, where:
   - `<PATH_PREFIX>` is the value of `global.pathPrefix` in `data-platform.values.yaml`.
2. **Log In**: log in with the technical administrator account defined in `analytics.technicalAdmin`.
3. **Access Identity Data Management**: Identity Data Management is available at `https://<DATA_MANAGEMENT_ENDPOINT>`.

### 7. Configure AI features

The Identity Data Platform includes two optional AI features:

- **AIDA**: the AI assistant and AI-assisted access reviews. It requires an LLM provider.
- **MCP server**: lets MCP-compatible AI clients query the Identity Data Platform. It does not require an LLM provider.

Both are disabled by the values file in [step 4](#4-deploy-the-identity-data-platform-chart). Apply the changes below with the install command from step 4.

#### Enable AIDA

AIDA uses two model aliases: a small, fast model and a large model. Declare the models of your provider, then assign the aliases to the AIDA components. These should be added to `data-platform.values.yaml`.

**Example of Amazon Bedrock configuration (EKS)**

```yaml
aida:
  enabled: true
litellmProxy:
  providers:
    aws:
      models:
        - name: small-model
          provider: "bedrock/converse/<BEDROCK_SMALL_MODEL_ID>"
        - name: large-model
          provider: "bedrock/converse/<BEDROCK_LARGE_MODEL_ID>"
      credentials:
        enabled: true
        region: "<AWS_REGION>"
        accessKeyId: "<ACCESS_KEY_ID>"
        secretAccessKey: "<SECRET_ACCESS_KEY>"
```

**Example of Azure OpenAI configuration (AKS)**

```yaml
aida:
  enabled: true
litellmProxy:
  providers:
    azure:
      models:
        - name: small-model
          provider: "azure/<SMALL_MODEL_DEPLOYMENT_NAME>"
        - name: large-model
          provider: "azure/<LARGE_MODEL_DEPLOYMENT_NAME>"
      credentials:
        enabled: true
        apiBaseUrl: "https://<AZURE_OPENAI_RESOURCE>.openai.azure.com"
        apiKey: "<AZURE_OPENAI_API_KEY>"
```

On both platforms, assign the aliases. These should be added to `data-platform.values.yaml`:

```yaml
aiuar:
  models:
    task: large-model
    nl: large-model
aitalk:
  fastworkflow:
    parameterExtraction:
      model: small-model
    responseGeneration:
      model: large-model
    planner:
      model: small-model
    agent:
      model: small-model
    conversationStore:
      model: small-model
```

> **Note**
> Use models that support tool calling and a large context window: a single identity context can be around 100 KB. Your Radiant Logic solutions engineer can recommend the models to use with your provider. To use a LiteLLM proxy that you already run instead of the built-in one, see [Use external LiteLLM proxy](https://developer.radiantlogic.com/ia/descartes/igrc-platform/self-managed/06-upgrade/migration-3.5.1/#step-4-optional-use-external-litellm-proxy) in the Identity Analytics guide.

#### Enable the MCP server

Add the following to `data-platform.values.yaml`:

```yaml
mcpServer:
  enabled: true
```

The MCP server is available at `https://<APP_ENDPOINT>/<PATH_PREFIX>/mcp`, where:

- `<PATH_PREFIX>` is the value of `global.pathPrefix`.

Each MCP client needs its own client in Keycloak with the `mcp-access` role. Requests without this role are rejected. To configure Keycloak and connect your AI clients, see [MCP Server: Getting started](https://developer.radiantlogic.com/ido/v2/mcp-server/getting-started/#technical-setup-in-radiantone-identity-observability).

#### Deactivate AI features

Set `aida.enabled: false` or `mcpServer.enabled: false`, then run the install command from step 4 again.

### 8. Update resource configuration

You can adjust resources, volume sizes and pod placement in `data-platform.values.yaml`. Your Radiant Logic solutions engineer can guide you depending on your use case.

1. **Database Resources and Volume Sizes**:

   ```yaml
   database:              # Analytics database
     resources:
       requests:
         cpu: 1
         memory: 4Gi
     persistence:
       data:
         size: 128Gi
       wal:
         size: 32Gi
   observationDatabase:   # Observation database
     resources:
       requests:
         cpu: 1
         memory: 4Gi
     persistence:
       data:
         size: 128Gi
       wal:
         size: 32Gi
   ```

2. **Portal Resources**:

   ```yaml
   portal:
     resources:
       requests:
         cpu: 1
         memory: 8Gi
     persistence:
       log: 16Gi
   ```

3. **Pod Assignment**: to run pods on dedicated nodes, set node selectors or tolerations for all components with `global.nodeSelector` and `global.tolerations`, or per component. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

   ```yaml
   global:
     nodeSelector: {}
     tolerations: []
   ```

For Shared Services, use the keys described in the Identity Analytics guide. To change the Keycloak database volumes, use `database.persistence.data.size` and `database.persistence.wal.size`.


## Updating a deployment

To apply a change to a values file, run the same install command again with the updated file.

**Update Shared Services**:

```bash
helm upgrade --install <SHARED_RELEASE_NAME> \
  oci://docker.io/radiantone/ida-shared-helm \
  --namespace <SHARED_NAMESPACE> \
  --version 3.5.3 \
  --values shared-minimal.values.yaml
```

**Update Identity Data Platform**:

```bash
helm upgrade --install data-platform \
  oci://docker.io/radiantone/ido-helm \
  --namespace <DATA_PLATFORM_NAMESPACE> \
  --version 3.1.0 \
  --set-file analytics.licenseFile=<LICENSE_FILE_PATH> \
  --values data-platform.values.yaml \
  --timeout 30m \
  --wait
```

> **Note**
> If Shared Services also serves Identity Analytics, check the Identity Analytics guide before upgrading it.


## Troubleshooting your Kubernetes environment

Follow these steps to diagnose deployment issues:

1. **Check Events**:

   ```bash
   kubectl get events --namespace <DATA_PLATFORM_NAMESPACE> --sort-by=.lastTimestamp
   ```

2. **Describe a Specific Pod**:

   ```bash
   kubectl describe pod <POD_NAME> --namespace <DATA_PLATFORM_NAMESPACE>
   ```

3. **Check Pod Logs**:

   ```bash
   kubectl logs <POD_NAME> --namespace <DATA_PLATFORM_NAMESPACE>
   ```

4. **Check Databases and Graph Pipeline**:

   ```bash
   kubectl get clusters.postgresql.cnpg.io --namespace <DATA_PLATFORM_NAMESPACE>
   kubectl get flinkdeployments --namespace <DATA_PLATFORM_NAMESPACE>
   ```

Common causes of failure:

- **Pods in `Pending` state**: not enough CPU or memory on the nodes, or no default storage class.
- **`ImagePullBackOff`**: the image pull secret is missing or incorrect.
- **Graph pipeline not starting**: the Flink operator is not installed, or the object storage credentials are wrong.


## Deleting Identity Data Platform chart

1. **Uninstall Identity Data Platform**:

   ```bash
   helm uninstall data-platform --namespace <DATA_PLATFORM_NAMESPACE> --wait
   ```

2. **Delete Persistent Volume Claims** if necessary to free up space. This permanently deletes the data:

   ```bash
   kubectl get pvc --namespace <DATA_PLATFORM_NAMESPACE>
   kubectl delete pvc --all --namespace <DATA_PLATFORM_NAMESPACE>
   ```

3. **Delete Namespace**:

   ```bash
   kubectl delete namespace <DATA_PLATFORM_NAMESPACE>
   ```

The object storage bucket or container is not deleted. Delete it from your cloud provider console if needed.


## Deleting Shared Services chart

Only delete Shared Services after uninstalling the Identity Data Platform, and only if no Identity Analytics instance uses it. Follow [Deleting Shared Services chart](https://developer.radiantlogic.com/ia/descartes/igrc-platform/self-managed/#deleting-shared-services-chart) in the Identity Analytics guide.

**Remove Operators**: only remove the operators once no application uses them:

```bash
helm uninstall flink-kubernetes-operator --namespace <OPERATORS_NAMESPACE>
helm uninstall cnpg --namespace <OPERATORS_NAMESPACE>
helm uninstall ingress-nginx --namespace ingress-nginx
```
