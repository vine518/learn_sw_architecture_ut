### Fase 4: Despliegue e Integración Continua

#### **Objetivo:**
Implementar CI/CD y desplegar el sistema utilizando Docker, Kubernetes, y GitHub Actions, asegurando que el sistema sea escalable y monitoreado adecuadamente. Inyectar variables de entorno y secretos para configuraciones seguras.

---

#### **Paso a Paso:**

1. **Crear un Dockerfile:**
   - Configurar la aplicación para su contenerización.
   ```dockerfile
   FROM node:16
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["node", "src/server.js"]
   ```

2. **Configurar Kubernetes:**
   - Crear un archivo YAML para el despliegue.
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
                 name: orders-api-secrets
                 key: DATABASE_URL
           - name: JWT_SECRET
             valueFrom:
               secretKeyRef:
                 name: orders-api-secrets
                 key: JWT_SECRET
   ```

3. **Crear un Servicio Kubernetes para Exposición:**
   ```yaml
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

4. **Configurar Secrets en Kubernetes:**
   ```bash
   kubectl create secret generic orders-api-secrets \
     --from-literal=DATABASE_URL=postgres://replicator:replicator_password@postgres:5432/clean_architecture_db \
     --from-literal=JWT_SECRET=your_jwt_secret
   ```

5. **Configurar el Pipeline CI/CD en GitHub Actions:**
   - Crear el archivo `.github/workflows/ci-cd.yaml`:
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

       - name: Push Docker Image to Registry
         run: |
           echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
           docker tag orders-api:1.0 ${{ secrets.DOCKER_USERNAME }}/orders-api:1.0
           docker push ${{ secrets.DOCKER_USERNAME }}/orders-api:1.0

     deploy:
       needs: build
       runs-on: ubuntu-latest

       steps:
       - name: Set up kubectl
         uses: azure/setup-kubectl@v3
         with:
           version: 'latest'

       - name: Configure kubeconfig
         run: |
           echo "${{ secrets.KUBECONFIG }}" > $HOME/.kube/config

       - name: Deploy to Kubernetes
         run: |
           kubectl apply -f k8s/deployment.yaml
           kubectl apply -f k8s/service.yaml
   ```

6. **Incluir Monitoreo y Alertas:**
   - **Prometheus:** Configurar para recopilar métricas del sistema.
   - **Grafana:** Crear dashboards para visualizar métricas clave como uso de CPU, memoria, y solicitudes por segundo.
   - Configuración de alertas básicas para tiempo de inactividad o alta carga.

   Ejemplo de configuración Prometheus para Kubernetes:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: prometheus-config
   data:
     prometheus.yml: |
       global:
         scrape_interval: 15s
       scrape_configs:
         - job_name: 'kubernetes'
           static_configs:
             - targets: ['orders-api-service.default.svc.cluster.local:3000']
   ```

7. **Documentar el Proceso:**
   - Incluir en la documentación cómo acceder a los logs, monitorear la API, y realizar troubleshooting.

---

#### **Entregables:**
- **Contenedor Docker funcional:**
   - Imagen de Docker publicada en un registro de contenedores.
- **Despliegue en Kubernetes:**
   - Recursos YAML aplicados para despliegue, secretos y servicio.
- **Pipeline CI/CD operativo:**
   - Configurado para automatizar pruebas, construcción y despliegue.
- **Monitoreo básico:**
   - Dashboards funcionales en Grafana y alertas configuradas.

