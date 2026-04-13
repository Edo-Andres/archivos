### Resumen clave
**Sí es factible automatizar el login con Clave Única**, pero **no se permite automatizar directamente con credenciales (RUT + contraseña) sin interacción humana** (viola términos y seguridad). La única vía oficial y soportada es el flujo **OpenID Connect (OIDC / OAuth 2.0)** del Gobierno de Chile, que requiere registro de tu aplicación en la plataforma **CeroFilas** y aprobación gubernamental.

El gobierno provee **herramientas oficiales, documentación técnica completa, SDKs/ejemplos en Python, PHP, .NET, Java y Postman** para integración segura.

---

## 1. Factibilidad y restricciones legales/tecnicas
### ¿Es factible?
- ✅ **Autenticación programática oficial (OIDC/OAuth 2.0):** Sí, totalmente soportado, es el método estándar para integraciones públicas/privadas autorizadas.
- ❌ **Automatización "bot" directa (scraping, Selenium, requests con RUT+clave):** **NO permitido ni factible de forma estable/legal**
  - Clave Única usa **doble factor (2FA: SMS/correo/biometría)**, CAPTCHA, tokens anti-CSRF y sesiones dinámicas → scraping falla constantemente.
  - Viola los términos de uso del Gobierno Digital y la Ley de Protección de Datos (Ley 19.628) → riesgo de bloqueo de cuenta, sanciones o acciones legales.
  - Solo instituciones públicas y proveedores autorizados pueden integrar oficialmente; particulares/empresas privadas deben solicitar acceso via CeroFilas.

### ¿Quién puede integrar?
- Instituciones públicas del Estado (obligatorio en muchos trámites)
- Empresas proveedoras de certificación de firma electrónica avanzada
- Entidades privadas autorizadas por el Gobierno Digital (previo trámite en CeroFilas)

---

## 2. Herramientas oficiales para desarrolladores (Clave Única)
### Plataforma de registro y credenciales: **CeroFilas**
Es el portal oficial para solicitar credenciales de integración (**Client ID, Client Secret**) y certificar tu aplicación:
- URL: https://cerofilas.gob.cl/
- Pasos: Registrar tu app → completar formulario técnico → esperar aprobación → obtener credenciales de desarrollo/producción.

### Protocolo oficial: **OpenID Connect (OIDC) sobre OAuth 2.0**
Es el único flujo soportado; Clave Única actúa como proveedor de identidad (IdP).
- Endpoints oficiales (producción):
  - Autorización: `https://accounts.claveunica.gob.cl/openid/authorize/`
  - Token: `https://accounts.claveunica.gob.cl/openid/token/`
  - UserInfo (datos usuario: RUN, nombre, correo): `https://accounts.claveunica.gob.cl/openid/userinfo/`
  - Cierre sesión: `https://accounts.claveunica.gob.cl/openid/endsession/`

### Documentación y recursos oficiales
1. **Manual de Integración Clave Única (guía técnica completa):** https://wikiguias.digital.gob.cl/Manuales/Integraci%C3%B3n_Clave%C3%9Anica
2. **Ejemplos de código (lenguajes comunes):** Python, PHP, .NET, Java, Postman (pruebas directas).
3. **Entorno de pruebas (Sandbox):** Disponible para probar flujos sin credenciales reales, antes de producción.
4. **Librerías/paquetes comunitarios (no oficiales pero útiles):**
   - Python/Django: `django-clave-unica` (integración lista)
   - Node.js/Express: Paquetes OIDC genéricos adaptables a Clave Única

---

## 3. Flujo de automatización simple (OIDC, el único método válido)
El flujo estándar es **Authorization Code Flow** (para apps web; el más simple y seguro):
1. Tu app redirige al usuario al endpoint de autorización de Clave Única (con Client ID, redirect_uri, scope=openid profile email)
2. El usuario ingresa RUT + contraseña + completa 2FA (**interacción humana obligatoria aquí**)
3. Clave Única redirige de vuelta a tu app con un **código de autorización**
4. Tu app intercambia ese código por un **access_token + id_token** (llamada POST al endpoint /token, con Client Secret)
5. Usas el access_token para obtener datos del usuario desde /userinfo
6. Autenticas al usuario en tu sistema con los datos de Clave Única

### Ejemplo simplificado en Python (requests)
```python
import requests

# Credenciales obtenidas en CeroFilas (desarrollo)
CLIENT_ID = "TU_CLIENT_ID"
CLIENT_SECRET = "TU_CLIENT_SECRET"
REDIRECT_URI = "https://tu-app.com/callback"
AUTH_URL = "https://accounts.claveunica.gob.cl/openid/authorize/"
TOKEN_URL = "https://accounts.claveunica.gob.cl/openid/token/"
USERINFO_URL = "https://accounts.claveunica.gob.cl/openid/userinfo/"

# Paso 1: Generar URL de login (usuario accede aquí)
params = {
    "client_id": CLIENT_ID,
    "redirect_uri": REDIRECT_URI,
    "response_type": "code",
    "scope": "openid profile email",
    "state": "valor_aleatorio_anti_csrf"
}
login_url = requests.Request('GET', AUTH_URL, params=params).prepare().url
print("URL de login:", login_url)

# Paso 2: Después de login, recibes 'code' en el callback
# Paso 3: Intercambiar código por token
token_data = {
    "grant_type": "authorization_code",
    "code": "CODIGO_RECIBIDO",
    "redirect_uri": REDIRECT_URI,
    "client_id": CLIENT_ID,
    "client_secret": CLIENT_SECRET
}
token_response = requests.post(TOKEN_URL, data=token_data)
tokens = token_response.json()
access_token = tokens["access_token"]

# Paso 4: Obtener datos del usuario
headers = {"Authorization": f"Bearer {access_token}"}
user_info = requests.get(USERINFO_URL, headers=headers).json()
print("Datos usuario:", user_info)
```

### Nota crítica
- **No se puede omitir la interacción humana (login + 2FA):** Es un requisito de seguridad gubernamental; cualquier intento de automatizar este paso (Selenium, scraping) es inestable, bloqueable y no autorizado.
- La "automatización" se refiere a **gestionar tokens, validar sesiones y consumir datos** después de que el usuario se autentique manualmente una vez.

---

## 4. Pasos para implementar (de principio a fin)
1. **Solicitar acceso en CeroFilas:** Registrar tu aplicación, completar datos técnicos, esperar aprobación y obtener Client ID/Secret (desarrollo → producción).
2. **Configurar entorno:** Usar librerías OIDC (ej: `requests-oauthlib` en Python, `passport` en Node.js) para manejar el flujo.
3. **Implementar flujo Authorization Code:** Redirección → callback → intercambio código → token → userinfo.
4. **Probar en Sandbox:** Validar flujo sin datos reales.
5. **Certificar y pasar a producción:** Cumplir requisitos de seguridad, obtener aprobación final.

---

## Conclusión final
- **Automatización programática segura y oficial:** Sí, mediante **OIDC/OAuth 2.0**, con herramientas y documentación oficial del Gobierno de Chile.
- **Automatización bot/scraping (sin interacción):** No es factible, ilegal y bloqueado por seguridad.
- **Requisito indispensable:** Registro y aprobación en **CeroFilas** para obtener credenciales de integración.
