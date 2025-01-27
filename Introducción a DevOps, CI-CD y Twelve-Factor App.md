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
  - Uso de herramientas como HashiCorp Vault para manejo seguro de secretos.

---

### **Gestión de Secretos con HashiCorp Vault**
HashiCorp Vault proporciona una solución segura para gestionar secretos, como credenciales y claves API. A continuación, se detalla cómo integrarlo en tu entorno:

#### **1. Configurar Vault en Kubernetes:**
- Implementar Vault en el clúster usando Helm:
  ```bash
  helm repo add hashicorp https://helm.releases.hashicorp.com
  helm install vault hashicorp/vault --set "server.dev.enabled=true"
  ```

- Configurar políticas de acceso en Vault para permitir que las aplicaciones obtengan secretos específicos:
  ```hcl
  path "secret/data/orders-api" {
    capabilities = ["read"]
  }
  ```

#### **2. Almacenar Secretos en Vault:**
- Usar la CLI de Vault para escribir secretos:
  ```bash
  vault kv put secret/orders-api DATABASE_URL=postgres://user:password@host:5432/db JWT_SECRET=myjwtsecret
  ```

#### **3. Integrar Vault con ArgoCD:**
- Configurar ArgoCD para usar secretos de Vault:
  ```yaml
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: orders-api
  spec:
    source:
      repoURL: git@github.com:tu-repo/orders-api.git
      path: k8s
      targetRevision: HEAD
      helm:
        parameters:
          - name: databaseUrl
            value: "{{ vault:secret/data/orders-api#DATABASE_URL }}"
          - name: jwtSecret
            value: "{{ vault:secret/data/orders-api#JWT_SECRET }}"
    destination:
      server: https://kubernetes.default.svc
      namespace: default
  ```

---

### **CI/CD con GitHub Actions y Vault**

#### **Pipeline Modificado para Integrar Vault:**
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
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Authenticate with Vault
      run: |
        export VAULT_ADDR=https://vault.your-domain.com
        vault login ${{ secrets.VAULT_TOKEN }}

    - name: Fetch Secrets from Vault
      run: |
        export DATABASE_URL=$(vault kv get -field=DATABASE_URL secret/orders-api)
        export JWT_SECRET=$(vault kv get -field=JWT_SECRET secret/orders-api)

    - name: ArgoCD Sync
      run: |
        argocd login --username admin --password ${{ secrets.ARGOCD_PASSWORD }} --insecure
        argocd app sync orders-api
```

Con estos pasos, Vault se integra completamente en el flujo CI/CD, asegurando que los secretos se gestionen y utilicen de manera segura tanto en GitHub Actions como en Kubernetes a través de ArgoCD.

