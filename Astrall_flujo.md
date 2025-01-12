# Flujo del Proyecto Astrall: Generación y Envío de PDFs Personalizados

A continuación, se detalla el flujo desde que Shopify notifica las suscripciones (mediante webhook) hasta el envío final de los PDFs:

---

## **1. Notificación inicial y recepción de datos de suscripción en Astrall**

### **Descripción:**
Dentro del proyecto Astrall, Shopify gestiona las primeras interacciones con el cliente, incluyendo:
- Notificar al cliente sobre la creación de una nueva suscripción.
- Generar y enviar la boleta correspondiente al pago inicial.

Posteriormente, Shopify envía datos de las suscripciones y pagos mediante un Webhook. Este Webhook es procesado por API Astrall para almacenar la información relevante en una base de datos y continuar con las operaciones siguientes.

### **Servicios GCP:**
- **Cloud Run (Servicio HTTP):**
  - Recibe el webhook enviado por Shopify.
  - Valida y procesa los datos recibidos.
- **Firestore:**
  - Almacena información sobre los clientes, productos, y frecuencias de suscripción.

### **Parámetros recibidos por la API:**
Estos son los datos o parámetros que deben estar en el webhook enviado por Shopify y que serán procesados por la API:
```python
def parse_args():
    parser = argparse.ArgumentParser(description="Process input arguments")
    parser.add_argument("--doc_type", required=True, help="report|horoscope")
    parser.add_argument("--area", required=True, help="general|love|work|money|health")
    parser.add_argument("--doc_format", required=True, help="general|medium|short")
    parser.add_argument("--name", required=True, help="Name of the person")
    parser.add_argument("--email", required=True, help="Email address")
    parser.add_argument("--date", required=True, help="Date in YYYY-MM-DD format")
    parser.add_argument("--time", required=True, help="Time in HH:MM format")
    parser.add_argument("--location", required=True, help="Location")
    parser.add_argument("--input_filepath", required=True, help="Input file path")
    parser.add_argument("--output_filepath", required=True, help="Output file path")
    parser.add_argument("--use_storage", type=bool, default=False, help="Use Google Cloud Storage")

    if ENV == 'production':
        parser.add_argument("--order_id", required=True, help="")
    else:
        parser.add_argument("--order_id", help="", default='test')

    return parser.parse_args()
```

---

## **2. Generación de PDFs personalizados en Astrall**

### **Descripción:**
Se generan PDFs personalizados para los clientes utilizando los datos de la suscripción y la integración con OpenAI (ChatGPT). Los PDFs se almacenan temporalmente en GCP para su envío posterior.

### **Servicios GCP:**
- **Cloud Scheduler:**
  - Programa las tareas de generación de PDFs según la frecuencia (diaria, semanal, mensual).
- **Cloud Run Jobs:**
  - Filtra clientes según los siguientes criterios:
    - Suscripciones activas en Firestore.
    - Parámetros específicos como `--doc_type`, `--area`, y `--doc_format`.
  - Recupera la información personalizada de cada cliente desde Firestore.
  - Integra con la API de OpenAI (ChatGPT) para generar contenido dinámico basado en los parámetros filtrados.
  - Genera el PDF utilizando librerías como **FPDF** o **ReportLab**.
  - Almacena los PDFs generados de manera temporal en **Cloud Storage**, organizándolos por cliente y fecha.
- **Cloud Storage:**
  - Actúa como repositorio central para los PDFs generados, facilitando su posterior envío.

---

## **3. Envío de PDFs a los clientes en Astrall**

### **Descripción:**
Los PDFs generados se envían por defecto por correo electrónico a los clientes, junto con una notificación personalizada. Además, si el cliente tiene una suscripción para un `--doc_type: horoscope` con envío diario y solicita explícitamente otro canal (WhatsApp), el PDF también puede enviarse mediante WhatsApp. En todos los casos, se registra el estado del envío en la base de datos para seguimiento.

### **Servicios GCP:**
- **Cloud Pub/Sub:**
  - Orquesta los envíos al notificar qué clientes tienen PDFs listos para ser enviados.
- **Cloud Run (Servicio HTTP):**
  - Recupera los PDFs desde Cloud Storage y envía los correos electrónicos o mensajes de WhatsApp.
  - Actualiza el estado del cliente en Firestore.
- **SendGrid:**
  - Envía los correos electrónicos con los PDFs adjuntos.
- **WhatsApp API (Twilio opcional):**
  - Gestiona el envío de mensajes de WhatsApp si aplica.

---

## **Flujo Completo**
1. Shopify envía datos de suscripciones mediante un Webhook.
2. Cloud Run Servicio recibe los datos y los almacena en Firestore.
3. Cloud Scheduler activa tareas según la frecuencia definida.
4. Cloud Run Job filtra clientes, genera PDFs personalizados y los almacena en Cloud Storage.
5. Pub/Sub orquesta el envío de PDFs listos.
6. Cloud Run Servicio envía los PDFs a los clientes utilizando SendGrid para correos electrónicos y, opcionalmente, API para WhatsApp.

---

## **Beneficios de esta Arquitectura**
- **Escalabilidad:**
  - Cloud Run y Firestore escalan automáticamente según la demanda.
- **Separación de responsabilidades:**
  - Cada etapa tiene servicios dedicados, simplificando la gestión.
- **Costos bajos:**
  - Los servicios son serverless y se pagan solo por uso.
- **Flexibilidad y tolerancia a fallos:**
  - Pub/Sub permite manejar fallos y reintentar envíos fallidos de manera automática.
