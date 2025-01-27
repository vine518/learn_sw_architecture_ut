### Tutorial: Implementación de CI/CD con Kubernetes y DigitalOcean

Este documento describe los pasos para desplegar tu aplicación Node.js con Kubernetes y configurar un pipeline de CI/CD utilizando DigitalOcean como proveedor de infraestructura.

---

### **Requisitos previos**
1. Una cuenta en DigitalOcean ([Regístrate aquí](https://www.digitalocean.com)).
2. Docker instalado en tu máquina local.
3. Kubectl instalado y configurado.
4. Aplicación Node.js con un `Dockerfile` listo para el despliegue.
5. GitHub Actions configurado en tu repositorio.

---

### **Paso 1: Crear un clúster de Kubernetes en DigitalOcean**

1. **Configura un clúster:**
   - Inicia sesión en DigitalOcean y navega a la sección de Kubernetes.
   - Haz clic en **"Create Kubernetes Cluster"**.
   - Elige la región más cercana y selecciona un nodo básico (por ejemplo, un nodo con 2 GB RAM).
   - Haz clic en **"Create Cluster"**.

2. **Descarga el archivo kubeconfig:**
   - Una vez creado el clúster, descarga el archivo kubeconfig para acceder al clúster desde tu CLI.
   - Configura `kubectl` para usar este archivo:
     ```bash
     export KUBECONFIG=/path/to/kubeconfig.yaml
     ```

3. **Verifica la conexión al clúster:**
   ```bash
   kubectl get nodes
   ```
   Debes ver la lista de nodos disponibles en tu clúster.

---

### **Paso 2: Configurar el despliegue con Kubernetes**

1. **Crea el archivo `deployment.yaml`:**
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

2. **Configura los secretos de la base de datos:**
   ```bash
   kubectl create secret generic database-secret \
     --from-literal=DATABASE_URL="postgres://user:password@host:5432/db"
   ```

3. **Aplica los archivos YAML:**
   ```bash
   kubectl apply -f deployment.yaml
   ```

4. **Verifica el despliegue:**
   ```bash
   kubectl get pods
   ```
   Asegúrate de que los pods están en estado `Running`.

---

### **Paso 3: Configurar CI/CD con GitHub Actions**

1. **Crea el archivo `.github/workflows/ci-cd.yaml`:**
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
           kubectl apply -f deployment.yaml
           kubectl apply -f service.yaml
   ```

2. **Configura los secretos en GitHub:**
   - Ve a la configuración del repositorio en GitHub.
   - Agrega secretos para `KUBECONFIG` y otras credenciales necesarias.

---

### **Paso 4: Configurar Prometheus y Grafana para Monitoreo**

1. **Configura Prometheus:**
   - Crea un archivo `prometheus.yml`:
     ```yaml
     global:
       scrape_interval: 15s
     scrape_configs:
       - job_name: 'kubernetes'
         static_configs:
           - targets: ['orders-api-service.default.svc.cluster.local:3000']
     ```

   - Despliega Prometheus en tu clúster:
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml
     ```

2. **Configura Grafana:**
   - Despliega Grafana en tu clúster:
     ```bash
     kubectl apply -f https://raw.githubusercontent.com/grafana/helm-charts/main/charts/grafana/templates/deployment.yaml
     ```
   - Conecta Grafana a Prometheus como fuente de datos.
   - Crea dashboards personalizados para monitorear métricas clave como uso de CPU y RAM.

---

### **Paso 5: Manejo de Secretos con HashiCorp Vault**

1. **Despliega Vault en Kubernetes:**
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
   - Configura las aplicaciones para leer secretos directamente desde Vault usando el operador de Kubernetes.

---

### **Conclusión**

Con estos pasos, puedes implementar tu aplicación Node.js en DigitalOcean Kubernetes con un pipeline de CI/CD completo. Adicionalmente, has configurado monitoreo con Prometheus y Grafana, así como manejo de secretos con HashiCorp Vault. Esto asegura que tu sistema sea escalable, seguro y monitoreado eficientemente.

