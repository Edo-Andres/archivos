# Informe  Arquitectura con GCP para Astrall

## **Propósito del Proyecto**

El proyecto Astrall proporciona servicios relacionados con la astrología, incluyendo la generación de informes personalizados en formato PDF para usuarios que compran o se suscriben a través de Shopify. Esta arquitectura está diseñada para ser escalable, paralela y eficiente, utilizando servicios de Google Cloud Platform (GCP).

---

## **Servicios de GCP Utilizados**

### **1. Cloud Run**

- **Propósito**: Ejecutar el backend Django y el generador de PDFs en un entorno serverless.
- **Características**:
  - Escalado automático basado en la carga.
  - Capacidad para manejar múltiples instancias en paralelo.
  - Aislamiento entre solicitudes.
  - Ideal para procesos más largos y complejos, como la interacción con modelos LLM.
- **Configuración Clave**:
  - – Concurrencia: 1 (por defecto) o configurada para múltiples solicitudes si el backend es thread-safe.
  - – Máximo de instancias configurables según la carga esperada.

### **2. Cloud Storage**

- **Propósito**: Almacenar los archivos PDF generados.
- **Características**:
  - Organización en carpetas según cliente y tipo de suscripción.
  - Acceso controlado mediante URLs firmadas para garantizar la seguridad.
- **Ejemplo de Organización**:
  - `/orders/{order_id}/{pdf_name}.pdf`

### **3. Cloud SQL**

- **Propósito**: Base de datos PostgreSQL administrada para persistir información sobre usuarios, pedidos y configuraciones astrológicas.
- **Características**:
  - Alta disponibilidad y recuperación ante fallos.
  - Capacidad para manejar múltiples conexiones concurrentes.

### **4. Pub/Sub**

- **Propósito**: Gestionar la comunicación asíncrona entre servicios.
- **Usos**:
  - Procesar eventos provenientes de Shopify (como compras o suscripciones).
  - Notificar la finalización de generación de PDFs.
- **Temas Configurados**:
  - `shopify-events`: Para recibir eventos desde Shopify.
  - `pdf-generation`: Para indicar que un PDF debe generarse.
  - `task-completion`: Para notificar tareas completadas.

### **5. Cloud Scheduler**

- **Propósito**: Programar tareas recurrentes para la generación de PDFs según la frecuencia de suscripción.
- **Configuración Clave**:
  - Frecuencias soportadas:
    - Diario: `*/24 * * *`.
    - Semanal: `0 0 * * 0`.
    - Mensual: `0 0 1 * *`.

### **6. Cloud Tasks**

- **Propósito**: Manejar tareas asíncronas como el envío de correos electrónicos a usuarios.
- **Características**:
  - Reintentos configurables en caso de fallos.
  - Ejecución independiente de tareas en paralelo.

### **7. Cloud Functions**

- **Propósito**: Ejecutar lógica ligera como la validación de webhooks y activación de otros servicios.
- **Usos**:
  - Validar y procesar webhooks de Shopify.
  - Activar la generación de PDFs mediante Pub/Sub.

### **8. Monitoring y Logging**

- **Propósito**: Supervisar el rendimiento y detectar fallos.
- **Servicios Utilizados**:
  - **Cloud Monitoring**: Métricas en tiempo real sobre uso de recursos y rendimiento.
  - **Cloud Logging**: Registro de eventos y errores para depuración.

---

## **Flujo de Trabajo Detallado**

### **1. Recepción de Pedido o Suscripción desde Shopify**

1. Shopify envía un **webhook** a una **Cloud Function**.
2. La función valida el evento y lo publica en el tema `shopify-events` de Pub/Sub.

### **2. Procesamiento en el Backend (Cloud Run)**

1. La aplicación Django escucha el mensaje en Pub/Sub.
2. Según el tipo de evento:
   - **Compra**:
     - Guarda los datos en **Cloud SQL**.
     - Genera el PDF interactuando con el modelo LLM de OpenAI.
     - Sube el archivo a **Cloud Storage**.
     - Publica un mensaje en el tema `task-completion` de Pub/Sub.
   - **Suscripción**:
     - Programa tareas recurrentes en **Cloud Scheduler** según la frecuencia elegida.

### **3. Generación de PDFs**

1. Cuando se solicita un PDF, **Cloud Run** ejecuta el generador.
2. Procesa los datos de usuario desde **Cloud SQL**.
3. Interactúa con el modelo LLM de OpenAI para generar contenido astrológico dinámico.
4. Renderiza el contenido generado utilizando plantillas y lo convierte en PDF.
5. Sube el archivo a **Cloud Storage**.
6. Publica un mensaje de finalización en Pub/Sub.

### **4. Entrega del PDF al Usuario**

1. Una tarea encolada mediante **Cloud Tasks** envía un correo al usuario con el enlace al PDF.
2. El enlace utiliza una **URL firmada** de **Cloud Storage** para acceso temporal.

### **5. Suscripciones Recurrentes**

1. **Cloud Scheduler** activa un evento según la frecuencia configurada.
2. La **Cloud Function** correspondiente genera nuevos PDFs para los usuarios suscritos y publica los resultados en Pub/Sub.

---

## **Diagrama de Flujo**

```plaintext
1. Pedido/Evento Shopify
    ↓
[Cloud Function: Valida Webhook]
    ↓
[Pub/Sub: Evento Pedido/Suscripción] → (Suscripciones Recurrentes por Cloud Scheduler)
    ↓
[Cloud Run: Backend Django]
    ↓
[Cloud SQL: Guarda Pedido]
    ↓
(Compra)      → [Cloud Run: Generación de PDFs (con LLM)] → [Cloud Storage: Subir PDF]
(Suscripción) → [Cloud Scheduler: Programar Eventos]
    ↓
[Pub/Sub: Finalización]
    ↓
[Cloud Tasks: Enviar Correo al Usuario con URL Firmada]
```

---

## **Ventajas de la Arquitectura**

1. **Escalabilidad**: Servicios serverless como Cloud Run y Pub/Sub aseguran que la aplicación pueda manejar picos de carga.
2. **Modularidad**: Los componentes están desacoplados, facilitando el mantenimiento y evolución del sistema.
3. **Flexibilidad**: Cloud Run es ideal para tareas complejas, como la interacción con modelos LLM.
4. **Seguridad**: Los datos y PDFs están protegidos con acceso controlado y URLs firmadas.
5. **Optimización de Costos**: Se paga solo por el uso real de los servicios.

