### Automación de Clúster de Kubernetes en DigitalOcean con Terraform, Vault, ArgoCD, Helm y GitHub Actions

Este documento detalla cómo usar Terraform para crear un clúster de Kubernetes en DigitalOcean de forma automatizada, integrar HashiCorp Vault para la gestión de secretos, implementar ArgoCD para GitOps, y utilizar Helm para configuraciones reutilizables. También se explica cómo configurar GitHub Actions para automatizar todo el flujo CI/CD.

---

### **Requisitos Previos**

1. **Terraform instalado:**
   - Descarga Terraform desde [aquí](https://www.terraform.io/downloads).

2. **Cuenta en DigitalOcean:**
   - Regístrate y genera un token de acceso desde [DigitalOcean API](https://cloud.digitalocean.com/account/api).

3. **Instalación de ArgoCD:**
   - Configura ArgoCD en el clúster Kubernetes para implementar GitOps. Consulta la sección "Instalación de ArgoCD" más adelante.

4. **Instalación de Helm:**
   - Descarga e instala Helm desde [Helm.sh](https://helm.sh/docs/intro/install/).

5. **Instalación de Vault:**
   - Usa Helm para instalar HashiCorp Vault en el clúster Kubernetes.

6. **GitHub Repository:**
   - Un repositorio donde se almacenará la configuración, manifiestos de Kubernetes, y workflows de GitHub Actions.

---

### **Estructura del Proyecto**

Organiza los archivos de tu proyecto como sigue:

```
terraform-digitalocean-k8s/
├── main.tf
├── variables.tf
├── outputs.tf
├── .github/
│   └── workflows/
│       └── terraform.yml
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── secret.yaml
```

---

### **Automatización del Clúster con Terraform**

#### **Archivo `main.tf`**
Define los recursos necesarios para el clúster Kubernetes:

```hcl
provider "digitalocean" {
  token = var.digitalocean_token
}

resource "digitalocean_kubernetes_cluster" "k8s_cluster" {
  name    = "my-k8s-cluster"
  region  = "nyc1"
  version = "1.25.2-do.0"

  node_pool {
    name       = "default-pool"
    size       = "s-2vcpu-4gb"
    node_count = 2
  }
}

output "kubeconfig" {
  value     = digitalocean_kubernetes_cluster.k8s_cluster.kube_configs[0].raw_config
  sensitive = true
}
```

#### **Archivo `variables.tf`**
Declara las variables necesarias:

```hcl
variable "digitalocean_token" {
  description = "Token de acceso a DigitalOcean"
  type        = string
  sensitive   = true
}
```

#### **Pipeline de CI/CD (terraform.yml)**
Automatiza la aplicación de Terraform:

```yaml
name: Terraform Kubernetes Cluster

on:
  push:
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.3.7

    - name: Terraform Init
      env:
        DIGITALOCEAN_TOKEN: ${{ secrets.DIGITALOCEAN_TOKEN }}
      run: terraform init

    - name: Terraform Apply
      env:
        DIGITALOCEAN_TOKEN: ${{ secrets.DIGITALOCEAN_TOKEN }}
      run: terraform apply -var "digitalocean_token=${{ secrets.DIGITALOCEAN_TOKEN }}" -auto-approve
```

---

### **Gestión de Secretos con HashiCorp Vault**

#### **Instalación de Vault en el Clúster**

1. Agrega el repositorio de Helm:
   ```bash
   helm repo add hashicorp https://helm.releases.hashicorp.com
   ```

2. Instala Vault en el clúster:
   ```bash
   helm install vault hashicorp/vault --set "server.dev.enabled=true"
   ```

#### **Configuración de Secretos**

1. Inicializa Vault:
   ```bash
   vault operator init
   vault operator unseal
   ```

2. Guarda un secreto en Vault:
   ```bash
   vault kv put secret/database URL="postgres://user:password@host:5432/db"
   ```

3. Configura aplicaciones para leer secretos directamente desde Vault utilizando el operador de Kubernetes.

---

### **Despliegue de Aplicaciones con Helm y ArgoCD**

#### **Archivo `Chart.yaml`**
Define el Helm Chart general:

```yaml
apiVersion: v2
name: orders-api
version: 1.0.0
description: Helm chart para despliegue de orders-api
```

#### **Archivo `values.yaml`**
Define valores predeterminados para configuraciones reutilizables:

```yaml
replicaCount: 2
image:
  repository: registry.digitalocean.com/my-registry/orders-api
  tag: "1.0"
  pullPolicy: IfNotPresent
service:
  type: LoadBalancer
  port: 80
env:
  DATABASE_URL: ""
  JWT_SECRET: ""
```

#### **Configuración de ArgoCD**

1. Instala ArgoCD en el clúster:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. Configura la aplicación en ArgoCD:
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: orders-api
     namespace: argocd
   spec:
     source:
       repoURL: git@github.com:helm-charts.git
       targetRevision: HEAD
       path: orders-api
     destination:
       server: https://kubernetes.default.svc
       namespace: default
     project: default
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   ```

3. Sincroniza ArgoCD con el clúster:
   ```bash
   argocd login <ARGOCD_SERVER>
   argocd app sync orders-api
   ```

---

### **Automatización del Pipeline con GitHub Actions**

#### **Pipeline de CI/CD (deploy.yml)**

```yaml
name: Deploy Application

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Set up Docker CLI
      run: |
        apt-get update && apt-get install -y docker.io

    - name: Log in to DigitalOcean Container Registry
      run: |
        echo "${{ secrets.DOCKER_PASSWORD }}" | docker login ${{ secrets.DOCKER_REGISTRY }} -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

    - name: Build and Push Docker Image
      run: |
        docker build -t ${{ secrets.DOCKER_REGISTRY }}/orders-api:1.0 .
        docker push ${{ secrets.DOCKER_REGISTRY }}/orders-api:1.0

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: ArgoCD Sync
      run: |
        argocd login --username admin --password ${{ secrets.ARGOCD_PASSWORD }} --insecure
        argocd app sync orders-api
```

---

### **Monitoreo con Prometheus y Grafana**

1. **Despliega Prometheus y Grafana usando Helm:**
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo add grafana https://grafana.github.io/helm-charts
   helm install prometheus prometheus-community/prometheus
   helm install grafana grafana/grafana
   ```

2. **Configura Grafana:**
   - Obtén la contraseña por defecto:
     ```bash
     kubectl get secret grafana-admin -o jsonpath="{.data.admin-password}" | base64 --decode
     ```
   - Accede al dashboard en el servicio de Grafana.

---

### **Conclusión**

Con esta configuración:
- Terraform automatiza la creación del clúster Kubernetes en DigitalOcean.
- HashiCorp Vault gestiona los secretos de forma segura.
- Helm permite configuraciones reutilizables y ArgoCD habilita GitOps.
- GitHub Actions automatiza el pipeline CI/CD.
- Prometheus y Grafana garantizan el monitoreo continuo del clúster.

