Gracias por el contexto, Ednar. Con lo que describes, **el problema no es la transcripción ni que la reunión sea programada**, sino **el tipo de licencia base que tienes**. Te explico con precisión qué está pasando y qué necesitas configurar para que **Copilot y @Facilitator** aparezcan **dentro de reuniones de Teams**.

***

## ✅ Conclusión rápida (la causa principal)

👉 **Microsoft 365 Business Basic + Copilot Business NO habilita Copilot en reuniones de Teams.**  
Aunque tengas **Copilot Business comprado**, **no se activa en reuniones** si la **licencia base** es *Business Basic*. [\[thefix.it.com\]](https://thefix.it.com/how-do-i-enable-copilot-in-microsoft-teams-meeting-fix-guide/), [\[easytweaks.com\]](https://www.easytweaks.com/enable-teams-copilot-meetings-transcription/)

Para **Copilot en reuniones de Teams y Facilitator**, Microsoft exige:

*   **Licencia base compatible**
*   **Licencia adicional Microsoft 365 Copilot**
*   **Políticas correctas en Teams Admin Center**

***

## 1️⃣ Licencias necesarias (punto crítico)

### ❌ Lo que tienes ahora

*   ✅ Microsoft 365 **Business Basic**
*   ✅ Microsoft 365 **Copilot Business (add-on)**

👉 **Esto NO es suficiente para Copilot en reuniones de Teams**. [\[thefix.it.com\]](https://thefix.it.com/how-do-i-enable-copilot-in-microsoft-teams-meeting-fix-guide/)

### ✅ Lo que SÍ funciona (opciones válidas)

Debes tener **una de estas licencias base**:

| Licencia base                       | ¿Copilot en reuniones Teams? |
| ----------------------------------- | ---------------------------- |
| Microsoft 365 Business **Standard** | ✅ Sí                         |
| Microsoft 365 Business **Premium**  | ✅ Sí                         |
| Microsoft 365 E3 / E5               | ✅ Sí                         |

Y además:

*   ✅ **Microsoft 365 Copilot** (el add-on de \~USD 30/usuario/mes)

Esto está documentado por Microsoft y por múltiples guías técnicas. [\[thefix.it.com\]](https://thefix.it.com/how-do-i-enable-copilot-in-microsoft-teams-meeting-fix-guide/), [\[easytweaks.com\]](https://www.easytweaks.com/enable-teams-copilot-meetings-transcription/), [\[linkedin.com\]](https://www.linkedin.com/pulse/using-copilot-teams-meetings-practical-step-by-step-guide-menard-zecee)

***

## 2️⃣ Requisitos específicos para Copilot en reuniones de Teams

Una vez corregida la licencia base:

### ✅ Requisitos funcionales

*   Reunión **programada** (no llamada instantánea)
*   El **organizador pertenece a tu tenant**
*   Usuario con rol **Organizador o Presentador**
*   **Transcripción habilitada**
*   Cliente **New Microsoft Teams** (desktop o web)

Copilot **no aparece** si:

*   La reunión fue creada por un usuario externo
*   Usas Teams clásico
*   No tienes licencia Copilot **asignada al usuario**
*   La licencia base no es compatible. [\[linkedin.com\]](https://www.linkedin.com/pulse/using-copilot-teams-meetings-practical-step-by-step-guide-menard-zecee)

***

## 3️⃣ Configuración obligatoria en Teams Admin Center

### Paso 1: Políticas de reunión

Ruta:

    Teams Admin Center
    → Meetings
    → Meeting policies
    → Global (o política asignada)

Verifica:

*   ✅ **Transcription = On**
*   ✅ **Allow Copilot = On**
*   ✅ **Recording = On** (opcional, pero recomendado)

Copilot **usa la transcripción como fuente de contexto**. [\[thefix.it.com\]](https://thefix.it.com/how-do-i-enable-copilot-in-microsoft-teams-meeting-fix-guide/)

***

## 4️⃣ Activar Copilot dentro de la reunión

### Durante la reunión

*   Botón **Copilot** en la barra superior
*   Opciones:
    *   “Recap so far”
    *   “What decisions were made?”
    *   “Who said what?”

Si el botón **no aparece**, el problema es **licencia**, no configuración. [\[linkedin.com\]](https://www.linkedin.com/pulse/using-copilot-teams-meetings-practical-step-by-step-guide-menard-zecee)

***

## 5️⃣ @Facilitator (agente de reuniones)

### Qué es

Facilitator es un **agente basado en Copilot** que:

*   Genera notas en tiempo real
*   Controla agenda y tiempos
*   Crea tareas (Planner / Loop)
*   Responde en el chat con `@Facilitator`

### Requisitos

*   ✅ **Microsoft 365 Copilot**
*   ✅ Reunión **programada**
*   ✅ App Facilitator permitida (por defecto lo está)

Microsoft lo confirma explícitamente:

> “A Microsoft 365 Copilot license is required to add Facilitator to a meeting” [\[support.mi...rosoft.com\]](https://support.microsoft.com/en-us/office/facilitator-in-microsoft-teams-meetings-37657f91-39b5-40eb-9421-45141e3ce9f6)

### Cómo activarlo

**Antes de la reunión**

*   Calendario de Teams → Editar reunión
*   Activar **Facilitator**

**Durante la reunión**

*   `Más acciones (…) → Turn on Facilitator`
*   Luego usar en el chat:
        @Facilitator crea la agenda
        @Facilitator resume decisiones

***

## 6️⃣ Por qué no lo ves aunque seas admin

Ser **Global Admin o Teams Admin NO habilita Copilot** si:

*   La **licencia base no es compatible**
*   El **usuario no tiene Copilot asignado**

Copilot **es por usuario**, no por tenant. [\[easytweaks.com\]](https://www.easytweaks.com/enable-teams-copilot-meetings-transcription/)

***

## ✅ Qué te recomiendo hacer ahora (pasos concretos)

1.  **Cambiar licencia base**
    *   De **Business Basic → Business Standard o Premium**
2.  Mantener **Microsoft 365 Copilot** asignado
3.  Esperar propagación (normalmente pocas horas)
4.  Verificar políticas de reunión
5.  Crear una reunión nueva y probar Copilot + @Facilitator

***

## 📎 Referencias oficiales

*   Facilitator en Teams (Microsoft Support)  
    [Facilitator in Microsoft Teams meetings](https://support.microsoft.com/en-us/office/facilitator-in-microsoft-teams-meetings-37657f91-39b5-40eb-9421-45141e3ce9f6)
*   Guía técnica Copilot en reuniones  
    [How do I enable Copilot in Teams meetings](https://thefix.it.com/how-do-i-enable-copilot-in-microsoft-teams-meeting-fix-guide/)

***

Si quieres, en el siguiente mensaje puedo:

*   Validar **exactamente qué licencia tienes asignada al usuario**
*   Decirte **cuál es la opción más barata** para tu caso
*   Explicarte **cómo probar Copilot sin compromiso anual**

Solo dime 👍
