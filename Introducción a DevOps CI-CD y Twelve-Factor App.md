### Introducción a DevOps, CI/CD y Twelve-Factor App

#### **¿Qué es DevOps?**
DevOps es una metodología que integra el desarrollo de software (Dev) y las operaciones de TI (Ops) con el objetivo de mejorar la colaboración, la automatización y la entrega continua de software. DevOps enfatiza:
- **Cultura colaborativa:** Mejora la comunicación entre equipos de desarrollo y operaciones.
- **Automatización:** Reduce tareas repetitivas, como pruebas, despliegues y monitoreo.
- **Entrega continua:** Asegura que el software esté siempre en un estado desplegable.
- **Feedback rápido:** Detecta problemas temprano mediante pruebas y monitoreo automatizados.

#### **¿Qué es CI/CD?**
CI/CD significa Integración Continua (CI) y Despliegue Continuo (CD):
- **CI (Integración Continua):**
  - Proceso de integrar código frecuentemente en un repositorio compartido.
  - Incluye pruebas automatizadas para detectar errores temprano.
  - Ejemplo: Cada vez que un desarrollador hace un "push" al repositorio, se ejecutan pruebas.

- **CD (Despliegue Continuo):**
  - Automatiza el despliegue de software en entornos de prueba y producción.
  - Garantiza entregas consistentes y de alta calidad.
  - Ejemplo: Una nueva versión se despliega automáticamente en producción tras pasar pruebas.

---

### **Los Twelve-Factor App y DevOps**
La metodología Twelve-Factor describe principios para construir aplicaciones modernas, escalables y portátiles. Estos factores se alinean con las prácticas de DevOps y CI/CD:

#### **1. Codebase (Base de Código):**
- Una sola base de código versionada en control de versiones.
- Accesible para todos los desarrolladores.
- **DevOps Relacionado:**
  - Uso de repositorios Git centralizados (por ejemplo, GitHub o GitLab).
  - CI valida código en cada "push".

#### **2. Dependencies (Dependencias):**
- Declarar y aislar dependencias explícitamente.
- Usar herramientas como `npm` o `pip` para instalar dependencias.
- **DevOps Relacionado:**
  - Uso de "Docker" para contenerizar la aplicación y sus dependencias.
  - Ejemplo: Archivos `package.json` en Node.js o `requirements.txt` en Python.

#### **3. Config (Configuración):**
- Almacenar configuraciones (como credenciales o variables de entorno) fuera del código.
- Ejemplo: Variables como `DATABASE_URL`.
- **DevOps Relacionado:**
  - Configuración mediante variables de entorno en CI/CD pipelines o Kubernetes.
  - Uso de herramientas como HashiCorp Vault o AWS Secrets Manager para manejo seguro de secretos.
  - Ejemplo en GitHub Actions:
    ```yaml
    env:
      DATABASE_URL: "postgres://user:password@host:5432/db"
    ```

#### **4. Backing Services (Servicios de Apoyo):**
- Tratar bases de datos, colas y APIs externas como servicios adjuntos.
- **DevOps Relacionado:**
  - Definir servicios en Docker Compose o Kubernetes.
  - Ejemplo:
    ```yaml
    services:
      postgres:
        image: postgres:latest
        environment:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
    ```

#### **5. Build, Release, Run (Construcción, Lanzamiento y Ejecución):**
- Separar claramente las fases:
  - **Build:** Convierte el código en un ejecutable.
  - **Release:** Combina el build con la configuración.
  - **Run:** Ejecuta la aplicación en producción.
- **DevOps Relacionado:**
  - CI/CD pipelines manejan estas etapas.
  - Ejemplo en GitHub Actions:
    ```yaml
    jobs:
      build:
        steps:
          - run: npm install
          - run: npm run build
      deploy:
        steps:
          - run: kubectl apply -f deployment.yaml
    ```

#### **6. Processes (Procesos):**
- Ejecutar la aplicación como uno o más procesos sin estado.
- **DevOps Relacionado:**
  - Uso de contenedores para aislar procesos.
  - Ejemplo: Escalar procesos con Kubernetes.

#### **7. Port Binding (Enlace de Puertos):**
- Exponer la aplicación a través de un puerto configurado.
- **DevOps Relacionado:**
  - Configuración de puertos en Docker o Kubernetes.
  - Ejemplo en Kubernetes:
    ```yaml
    spec:
      containers:
        - ports:
            - containerPort: 3000
    ```

#### **8. Concurrency (Concurrencia):**
- Escalar la aplicación dividiendo el trabajo en procesos.
- **DevOps Relacionado:**
  - Escalado horizontal en Kubernetes mediante "replicas".
  - Ejemplo:
    ```yaml
    spec:
      replicas: 3
    ```

#### **9. Disposability (Desechabilidad):**
- Procesos deben iniciar y detenerse rápidamente.
- **DevOps Relacionado:**
  - Uso de contenedores para inicios rápidos y despliegues sin interrupción.

#### **10. Dev/Prod Parity (Paridad Desarrollo/Producción):**
- Mantener los entornos lo más similares posible.
- **DevOps Relacionado:**
  - Usar Docker para garantizar consistencia entre entornos.

#### **11. Logs:**
- Tratar los logs como flujos de eventos.
- **DevOps Relacionado:**
  - Enviar logs a sistemas centralizados como ELK Stack o Prometheus.
  - Ejemplo de configuración de logs:
    ```yaml
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    ```

#### **12. Admin Processes (Procesos Administrativos):**
- Ejecutar tareas administrativas como procesos desechables.
- **DevOps Relacionado:**
  - Usar scripts o "jobs" temporales en CI/CD pipelines.

---

### **Manejo de Secrets con Vault**
1. **Instalar y configurar HashiCorp Vault:**
   - Descargar e instalar Vault en un servidor seguro.
   - Inicializar Vault y crear claves de acceso.
     ```bash
     vault operator init
     vault operator unseal
     ```
   - Configurar secretos:
     ```bash
     vault kv put secret/database URL=postgres://user:password@host:5432/db
     ```

2. **Integrar Vault con CI/CD:**
   - Configurar GitHub Actions para obtener secretos:
     ```yaml
     steps:
       - name: Fetch secrets
         run: |
           export DATABASE_URL=$(vault kv get -field=URL secret/database)
     ```

---

### **Gobierno en DevOps y Cloud**
1. **DevOps Governance:**
   - Establecer políticas para pipelines CI/CD.
   - Uso de "code reviews" obligatorios antes de integrar código.
   - Auditoría de cambios con logs centralizados.

2. **Cloud Governance:**
   - Controlar costos con presupuestos y monitoreo en tiempo real.
   - Uso de etiquetas (tags) para clasificar recursos.
   - Configurar límites de uso en servicios como AWS o Azure.

3. **Database Governance:**
   - Configurar backups automáticos y pruebas de restauración.
   - Monitoreo de rendimiento con herramientas como CloudWatch (AWS).
   - Encriptación de datos en reposo y en tránsito.

4. **SOA Governance:**
   - Documentar todas las APIs con Swagger/OpenAPI.
   - Uso de gateways API para gestionar seguridad y límites de tasa.
   - Monitoreo del rendimiento de servicios y sus dependencias.

---

### **Prometheus y Grafana**
1. **Configurar Prometheus para monitoreo:**
   - Crear un archivo de configuración `prometheus.yml`:
     ```yaml
     global:
       scrape_interval: 15s
     scrape_configs:
       - job_name: 'kubernetes'
         static_configs:
           - targets: ['orders-api-service.default.svc.cluster.local:3000']
     ```

   - Ejecutar Prometheus en Docker:
     ```bash
     docker run -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
     ```

2. **Configurar Grafana para visualización:**
   - Ejecutar Grafana en Docker:
     ```bash
     docker run -d -p 3000:3000 grafana/grafana
     ```
   - Conectar Prometheus como fuente de datos.
   - Crear dashboards personalizados para monitorear métricas clave.

---

### **Código Kubernetes (deployment.yaml)**
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

### **CI/CD con GitHub Actions**
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

---

Con estos pasos y configuraciones completas, puedes implementar, monitorear y gestionar el sistema bajo las mejores prácticas de DevOps.

