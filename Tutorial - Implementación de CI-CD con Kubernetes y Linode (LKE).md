### Tutorial: Implementación de CI/CD con Kubernetes y Linode (LKE)

Este documento detalla los pasos para desplegar tu aplicación Node.js con Kubernetes utilizando Linode Kubernetes Engine (LKE) y configurar un pipeline de CI/CD con GitHub Actions. Este tutorial incluye todos los detalles necesarios, incluso para quienes tienen poca experiencia.

---

### **Requisitos previos**
1. Una cuenta en Linode ([Regístrate aquí](https://www.linode.com)).
2. Docker instalado en tu máquina local.
3. Kubectl instalado y configurado.
4. Aplicación Node.js con un `Dockerfile` listo para el despliegue.
5. Acceso a tu repositorio de GitHub.

---

### **Estructura del Repositorio**

Crea un repositorio en GitHub con la siguiente estructura:

```
project-name/
├── .github/
│   └── workflows/
│       └── ci-cd.yaml
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── prometheus.yml
├── src/
│   ├── server.js
│   ├── ...otros archivos de la aplicación...
├── Dockerfile
├── package.json
├── README.md
```

- **`.github/workflows/ci-cd.yaml`:** Archivo de configuración de GitHub Actions.
- **`k8s/`:** Carpeta que contiene los archivos de configuración de Kubernetes.
- **`prometheus.yml`:** Archivo de configuración de Prometheus.

---

### **Paso 1: Crear un clúster en Linode Kubernetes Engine (LKE)**

1. **Configura un clúster:**
   - Inicia sesión en Linode y ve a la sección de Kubernetes.
   - Haz clic en **"Create Cluster"**.
   - Elige la región más cercana y selecciona un plan básico (por ejemplo, 2 vCPU y 4GB RAM).
   - Asigna un nombre al clúster y haz clic en **"Create Cluster"**.

2. **Descarga el archivo kubeconfig:**
   - Una vez creado el clúster, descarga el archivo kubeconfig desde el panel de control.
   - Configura kubectl para usar este archivo:
     ```bash
     export KUBECONFIG=/path/to/kubeconfig.yaml
     ```

3. **Verifica la conexión al clúster:**
   ```bash
   kubectl get nodes
   ```
   Debes ver una lista de los nodos activos en el clúster.

---

### **Paso 2: Configurar el despliegue con Kubernetes**

1. **Crea el archivo `k8s/deployment.yaml`:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: orders-api
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: orders-api
     template:
       metadata:
         labels:
           app: orders-api
       spec:
         containers:
         - name: orders-api
           image: orders-api:1.0
           ports:
           - containerPort: 3000
           env:
           - name: DATABASE_URL
             valueFrom:
               secretKeyRef:
                 name: database-secret
                 key: DATABASE_URL
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: orders-api-service
   spec:
     selector:
       app: orders-api
     ports:
     - protocol: TCP
       port: 80
       targetPort: 3000
     type: LoadBalancer
   ```

2. **Crea el archivo `k8s/prometheus.yml`:**
   ```yaml
   global:
     scrape_interval: 15s
   scrape_configs:
     - job_name: 'kubernetes'
       static_configs:
         - targets: ['orders-api-service.default.svc.cluster.local:3000']
   ```
   Este archivo es utilizado por Prometheus para recopilar métricas de tu aplicación.

3. **Configura los secretos de la base de datos:**
   ```bash
   kubectl create secret generic database-secret \
     --from-literal=DATABASE_URL="postgres://user:password@host:5432/db"
   ```

4. **Aplica los archivos YAML:**
   ```bash
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   ```

5. **Verifica el despliegue:**
   ```bash
   kubectl get pods
   ```
   Asegúrate de que los pods están en estado `Running`.

---

### **Paso 3: Configurar CI/CD con GitHub Actions**

1. **Configura el archivo `.github/workflows/ci-cd.yaml`:**
   ```yaml
   name: CI/CD Pipeline

   on:
     push:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout Code
         uses: actions/checkout@v3

       - name: Set up Node.js
         uses: actions/setup-node@v3
         with:
           node-version: 16

       - name: Install Dependencies
         run: npm install

       - name: Run Tests
         run: npm test

       - name: Build Docker Image
         run: |
           docker build -t orders-api:1.0 .

     deploy:
       needs: build
       runs-on: ubuntu-latest

       steps:
       - name: Set up kubectl
         uses: azure/setup-kubectl@v3
         with:
           version: 'latest'

       - name: Deploy to Kubernetes
         run: |
           kubectl apply -f k8s/deployment.yaml
           kubectl apply -f k8s/service.yaml
   ```

2. **Agrega secretos en GitHub:**
   - Ve a **Settings > Secrets and variables > Actions** en tu repositorio.
   - Agrega el archivo `kubeconfig` como un secreto llamado `KUBECONFIG`.
   - Asegúrate de agregar otros secretos necesarios (por ejemplo, claves de API).

3. **Prueba el pipeline:**
   - Realiza un cambio en el repositorio y haz un push a la rama `main`.
   - Ve a la página de Actions en tu repositorio para monitorear el progreso del pipeline.

---

### **Paso 4: Configurar Prometheus y Grafana para Monitoreo**

1. **Despliega Prometheus:**
   - Usa un operador para desplegar Prometheus en tu clúster:
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml
     ```

2. **Configura Grafana:**
   - Despliega Grafana:
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/grafana/helm-charts/main/charts/grafana/templates/deployment.yaml
     ```
   - Accede al dashboard de Grafana y configura Prometheus como fuente de datos.
   - Crea dashboards personalizados para monitorear métricas clave.

---

### **Paso 5: Manejo de Secretos con HashiCorp Vault**

1. **Despliega Vault:**
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/hashicorp/vault-helm/main/templates/deployment.yaml
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
   - Usa el operador de Vault para que las aplicaciones lean secretos directamente desde Vault.

---

### **Conclusión**

Este tutorial detalla todos los pasos necesarios para configurar y desplegar tu aplicación Node.js con Kubernetes en Linode. Asegura que cada archivo esté en el lugar correcto y que los secretos y configuraciones estén protegidos. Con la configuración de CI/CD y monitoreo, tu sistema será escalable, seguro y fácil de gestionar.

