### Automación de Clúster de Kubernetes en DigitalOcean con Terraform

Este documento detalla cómo usar Terraform para crear un clúster de Kubernetes en DigitalOcean de forma automatizada. Además, se explica cómo integrar esta configuración con GitHub Actions para una automatización completa.

---

### **Requisitos Previos**

1. **Terraform instalado:**
   - Descarga Terraform desde [aquí](https://www.terraform.io/downloads).

2. **Cuenta en DigitalOcean:**
   - Regístrate y genera un token de acceso desde [DigitalOcean API](https://cloud.digitalocean.com/account/api).

3. **Configuración local de CLI de Terraform:**
   - Verifica la instalación ejecutando:
     ```bash
     terraform --version
     ```

4. **GitHub Repository:**
   - Un repositorio donde se almacenará la configuración y los workflows de GitHub Actions.

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
```

---

### **Archivo `main.tf`**

Define los recursos de DigitalOcean necesarios para el clúster:

```hcl
provider "digitalocean" {
  token = var.digitalocean_token
}

resource "digitalocean_kubernetes_cluster" "k8s_cluster" {
  name    = "my-k8s-cluster"
  region  = "nyc1"  # Cambia la región según tus necesidades
  version = "1.25.2-do.0"

  node_pool {
    name       = "default-pool"
    size       = "s-2vcpu-4gb"  # Tamaño del nodo
    node_count = 2               # Número de nodos
  }
}

output "kubeconfig" {
  value     = digitalocean_kubernetes_cluster.k8s_cluster.kube_configs[0].raw_config
  sensitive = true
}
```

---

### **Archivo `variables.tf`**

Declara las variables necesarias:

```hcl
variable "digitalocean_token" {
  description = "Token de acceso a DigitalOcean"
  type        = string
  sensitive   = true
}
```

---

### **Archivo `outputs.tf`**

Define los outputs relevantes para el clúster:

```hcl
output "kubeconfig" {
  description = "Configuración para acceder al clúster Kubernetes."
  value       = digitalocean_kubernetes_cluster.k8s_cluster.kube_configs[0].raw_config
  sensitive   = true
}
```

---

### **Inicializar y Aplicar Terraform Localmente**

1. **Inicializa Terraform:**
   ```bash
   terraform init
   ```

2. **Planifica los cambios:**
   ```bash
   terraform plan -var "digitalocean_token=<TU_TOKEN>"
   ```

3. **Aplica los cambios:**
   ```bash
   terraform apply -var "digitalocean_token=<TU_TOKEN>" -auto-approve
   ```

---

### **Automatización con GitHub Actions**

1. **Crea el archivo `.github/workflows/terraform.yml`:**
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

       - name: Terraform Plan
         env:
           DIGITALOCEAN_TOKEN: ${{ secrets.DIGITALOCEAN_TOKEN }}
         run: terraform plan -var "digitalocean_token=${{ secrets.DIGITALOCEAN_TOKEN }}"

       - name: Terraform Apply
         env:
           DIGITALOCEAN_TOKEN: ${{ secrets.DIGITALOCEAN_TOKEN }}
         run: terraform apply -var "digitalocean_token=${{ secrets.DIGITALOCEAN_TOKEN }}" -auto-approve
   ```

2. **Agrega el token de DigitalOcean como un secreto en GitHub:**
   - Ve a **Settings > Secrets and variables > Actions**.
   - Agrega un secreto llamado `DIGITALOCEAN_TOKEN` con tu token de acceso.

3. **Realiza un push al repositorio:**
   - Realiza un commit y push de los archivos de Terraform al repositorio.
   - Verifica el estado del workflow en la pestaña **Actions** del repositorio.

---

### **Prometheus y Grafana para Kubernetes en DigitalOcean**

1. **Despliega Prometheus:**
   - Usa el operador de Prometheus:
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml
     ```

2. **Despliega Grafana:**
   - Usa Helm para instalar Grafana:
     ```bash
     helm repo add grafana https://grafana.github.io/helm-charts
     helm install grafana grafana/grafana
     ```

3. **Accede a Grafana:**
   - Usa el comando para obtener la contraseña por defecto:
     ```bash
     kubectl get secret grafana-admin -o jsonpath="{.data.admin-password}" | base64 --decode
     ```
   - Accede al dashboard usando el servicio de Grafana en el puerto expuesto.

---

### **HashiCorp Vault para Manejo de Secretos**

1. **Instala Vault en el clúster:**
   ```bash
   helm repo add hashicorp https://helm.releases.hashicorp.com
   helm install vault hashicorp/vault
   ```

2. **Inicializa Vault:**
   ```bash
   vault operator init
   vault operator unseal
   ```

3. **Configura secretos en Vault:**
   ```bash
   vault kv put secret/database URL="postgres://user:password@host:5432/db"
   ```

4. **Integra Vault con Kubernetes:**
   - Configura las aplicaciones para leer secretos directamente desde Vault usando el operador de Kubernetes.

---

### **Conclusión**

Con esta configuración, puedes usar Terraform para crear un clúster de Kubernetes en DigitalOcean de forma automatizada. Adicionalmente, la integración con GitHub Actions permite una infraestructura como código gestionada de manera continua. Prometheus, Grafana y Vault garantizan monitoreo y manejo seguro de secretos.

