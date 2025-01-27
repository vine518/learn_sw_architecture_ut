### Separación de Repositorios y Configuración Dinámica para GitOps

Este documento detalla cómo implementar un flujo GitOps dinámico y desacoplado utilizando Terraform para la automatización de clústeres Kubernetes, Helm para configuraciones reutilizables, y ArgoCD para la sincronización continua. También se explica cómo estructurar los repositorios de manera que los flujos de CI/CD sean independientes y escalables para múltiples aplicaciones.

---

### **Estructura General de los Repositorios**

Se recomienda dividir la infraestructura y las aplicaciones en repositorios separados:

1. **Repositorio de Infraestructura (terraform-infra):**
   - Contendrá los archivos Terraform para crear y gestionar el clúster Kubernetes y otros recursos necesarios.

2. **Repositorio de Helm Charts (helm-charts):**
   - Almacena los templates Helm reutilizables que pueden aplicarse a múltiples aplicaciones.

3. **Repositorio de Aplicaciones (app-repo):**
   - Contendrá el código de cada aplicación específica junto con sus configuraciones específicas (valores de Helm).

---

### **Repositorio 1: Infraestructura (terraform-infra)**

#### **Estructura del Repositorio:**
```
terraform-infra/
├── main.tf
├── variables.tf
├── outputs.tf
├── .github/
│   └── workflows/
│       └── terraform.yml
```

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

### **Repositorio 2: Helm Charts (helm-charts)**

#### **Estructura del Repositorio:**
```
helm-charts/
├── orders-api/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── secret.yaml
```

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

---

### **Repositorio 3: Aplicación Específica (app-repo)**

#### **Estructura del Repositorio:**
```
app-repo/
├── src/
├── Dockerfile
├── k8s/
│   └── argocd-application.yaml
├── .github/
│   └── workflows/
│       └── deploy.yml
```

#### **Archivo `Dockerfile`**
Define cómo se construye la imagen Docker a partir del código Node.js:

```dockerfile
# Base image
FROM node:16-alpine

# Set working directory
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Expose the application port
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```

#### **Archivo `argocd-application.yaml`**
Configura ArgoCD para gestionar el despliegue de la aplicación:

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

#### **Pipeline de CI/CD (deploy.yml)**
Automatiza el despliegue de la aplicación, incluyendo la subida de imágenes a DigitalOcean Container Registry:

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

### **Conclusión**

Con esta configuración:
- Terraform crea y gestiona la infraestructura de Kubernetes en DigitalOcean desde un repositorio dedicado.
- Helm proporciona plantillas reutilizables y centralizadas para configuraciones dinámicas en un repositorio separado.
- Las aplicaciones específicas se implementan desde sus propios repositorios, integrándose dinámicamente con Helm y ArgoCD.
- El Dockerfile permite construir imágenes Docker desde el código fuente Node.js.
- DigitalOcean Container Registry se utiliza para almacenar y gestionar imágenes Docker.

Este flujo permite escalabilidad, flexibilidad y una verdadera gestión GitOps desacoplada.

