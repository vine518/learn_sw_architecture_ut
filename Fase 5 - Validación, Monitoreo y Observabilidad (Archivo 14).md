### Fase 5: Validación, Monitoreo y Observabilidad (Archivo 14)

#### **Objetivo:**
Asegurar que el sistema funcione según lo esperado, integrando herramientas de monitoreo, pruebas de carga y validación para garantizar estabilidad, escalabilidad y observabilidad.

---

### **Pasos para la Validación, Monitoreo y Observabilidad**

#### **Paso 1: Pruebas de Carga**
- Utilizar herramientas como Apache JMeter o k6 para probar la capacidad del sistema bajo carga.
  
**Ejemplo con k6:**
1. Instalar k6:
   ```bash
   brew install k6  # macOS
   sudo apt install k6  # Linux
   ```
2. Crear un archivo de prueba (`14-load-test.js`):
   ```javascript
   import http from 'k6/http';
   import { check } from 'k6';

   export default function () {
     const res = http.get('http://orders-api-service:3000/orders');
     check(res, {
       'status is 200': (r) => r.status === 200,
     });
   }
   ```
3. Ejecutar la prueba:
   ```bash
   k6 run 14-load-test.js
   ```

#### **Paso 2: Configurar Prometheus y Grafana**
- **Prometheus:**
  - Configura Prometheus para recolectar métricas del clúster y servicios.
  - Ejemplo de `14-prometheus-config.yml`:
    ```yaml
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'kubernetes'
        static_configs:
          - targets: ['orders-api-service:3000']
    ```
  - Aplica el manifiesto:
    ```bash
    kubectl apply -f 14-prometheus-config.yml
    ```

- **Grafana:**
  - Instalar Grafana usando Helm:
    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    helm install grafana grafana/grafana
    ```
  - Acceder al tablero de Grafana:
    ```bash
    kubectl port-forward svc/grafana 3000:80 -n grafana
    ```
    - URL: [http://localhost:3000](http://localhost:3000).
    - Usuario/Contraseña predeterminados: admin/admin.

#### **Paso 3: Implementar Trazas Distribuidas**
- Usa Jaeger o Zipkin para supervisar el flujo de solicitudes entre servicios:
  - Instalar Jaeger:
    ```bash
    kubectl create namespace observability
    helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
    helm install jaeger jaegertracing/jaeger --namespace observability
    ```
  - Configura servicios para enviar trazas a Jaeger mediante las bibliotecas proporcionadas por tu lenguaje de programación.

#### **Paso 4: Configurar Alertas**
- Usa Alertmanager para recibir notificaciones basadas en eventos de Prometheus:
  - Configura reglas de alerta:
    ```yaml
    groups:
      - name: alert-rules
        rules:
          - alert: HighLatency
            expr: http_request_duration_seconds_bucket{le="0.5"} > 0.1
            for: 1m
            labels:
              severity: warning
            annotations:
              summary: "Alta latencia detectada"
    ```
  - Aplica el archivo:
    ```bash
    kubectl apply -f 14-alertmanager-rules.yml
    ```

#### **Paso 5: Validación de Integridad**
- Automatizar pruebas de extremo a extremo con Cypress o Selenium:
  - **Cypress:** Ideal para validar flujos de usuario en aplicaciones frontend.
  - **Selenium:** Para pruebas funcionales en navegadores.

#### **Paso 6: Validación de Seguridad**
- **Escaneo de Vulnerabilidades:**
  - Usa Trivy para escanear imágenes Docker y configuraciones de Kubernetes:
    ```bash
    trivy image orders-api:1.0
    ```
- **Políticas de Seguridad:**
  - Implementa Open Policy Agent (OPA) o Kyverno para asegurar configuraciones correctas:
    ```bash
    kubectl apply -f 14-k8s-security-policies.yml
    ```

---

### **Prácticas Recomendadas**

#### **1. Seguridad:**
- Habilitar TLS en todos los servicios expuestos.
- Implementar autenticación y autorización con OAuth2 o RBAC.

#### **2. Monitoreo Continuo:**
- Configurar métricas clave como disponibilidad (SLA), latencia (SLO) y error rate (SLI).
- Usar dashboards preconfigurados en Grafana para detectar anomalías rápidamente.

#### **3. Automatización:**
- Automatizar pruebas de carga como parte del pipeline CI/CD.
- Integrar validación de seguridad con herramientas como Trivy o Falco.

#### **4. Escalabilidad:**
- Implementar autoescalado basado en métricas de Prometheus.
- Monitorear el uso de recursos del clúster y optimizar configuraciones.

#### **5. Gestión de Logs:**
- Centralizar logs con ELK Stack o Loki.
- Configurar retención de logs basada en políticas de cumplimiento.

---

### **Entregables:**
1. **Métricas Monitorizadas:**
   - Prometheus configurado para recolección de datos.
   - Grafana con dashboards personalizados.
2. **Trazas Distribuidas:**
   - Jaeger integrado con servicios clave.
3. **Alertas Configuradas:**
   - Reglas de alerta en Prometheus y Alertmanager.
4. **Pruebas de Carga y Seguridad:**
   - Scripts de prueba automatizados con k6 y Trivy.
5. **Validación de Seguridad:**
   - Escaneo de vulnerabilidades y políticas aplicadas.
6. **Documentación:**
   - Guía de monitoreo, pruebas y validación completa.

---

Con estas configuraciones, los estudiantes tendrán un entorno seguro, monitoreado y validado para garantizar que el sistema pueda operar en escenarios de producción reales.

