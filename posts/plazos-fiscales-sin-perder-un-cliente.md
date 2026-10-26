---
title: "Cómo no perder un plazo fiscal de un cliente (sin libreta ni Excel)"
description: "Sistema de cinco piezas para que ningún modelo fiscal de tus clientes se pase de fecha: ficha que decide el calendario, aviso a 21 días, un solo mensaje al día y baja sin borrar. Con dato medido del catálogo n8n."
---

# Cómo no perder un plazo fiscal de un cliente (sin libreta ni Excel)

Son las 19:30 de un 18 de abril. Mañana vence el 303 del primer trimestre y hay un cliente que se dio de alta en marzo: no está en la hoja, porque la hoja la actualiza una compañera que está de baja. La fecha no se perdió por falta de conocimiento —todo el mundo sabe que el 303 vence el 20—, se perdió porque vivía en la cabeza de una persona y no en un sitio que alguien pueda consultar.

El fallo real de la mayoría de asesorías pequeñas no es de calendario: **es de dónde vive el dato**. Cuando vive en una libreta, el sistema falla en cuanto falta la persona que la escribe.

## Por qué la libreta funciona hasta que deja de funcionar

Cualquier despacho con cinco clientes lleva los plazos sin sistema. Con treinta, aparecen los síntomas:

- **La fecha no es verificable.** Nadie responde en diez segundos «¿qué vence en los próximos 21 días?» sin abrir tres sitios.
- **El régimen decide los modelos.** Un autónomo en módulos no presenta lo mismo que una SL con dos empleados; si esa condición no está guardada al lado del cliente, el aviso no puede saber qué le toca.
- **No distingue activo de baja.** Se avisa de clientes dados de baja en enero, y el ruido hace que el equipo deje de mirar.
- **No queda registro.** A «¿esto se avisó?» se responde «creo que sí».

## El dato: en el catálogo público de n8n no hay nada que resuelva esto

Antes de montar nada conviene saber qué existe ya. Medido el **26 de octubre de 2026** recorriendo todas las páginas de resultados de la API pública del catálogo de n8n (`api.n8n.io/api/templates/search`):

| Búsqueda | Plantillas | Vistas (mediana) | La más vista |
|---|---|---|---|
| `deadline` | 21 | 0 | 1.859 — un recordatorio de tareas de Notion |
| `tax` | 32 | 0 | 9.214 — un asistente de consulta fiscal, no un calendario |
| `tax deadline` | **1** | **0** | 0 — publicada en octubre de 2025 |
| `compliance` | 94 | 0 | 3.892 — un resumen de noticias de seguridad |
| `plazo`, `fiscal`, `vencimiento`, `recordatorio` | **0** | — | — |

Tres lecturas, y la tercera es la que importa:

- **Lo que hay son recordatorios genéricos.** El más visto de `deadline` avisa de tareas de Notion: sirve para cualquier cosa y por eso no sirve para esto.
- **Lo específico está a cero.** La única plantilla de `tax deadline` tiene 0 vistas, igual que 70 de las 94 de `compliance`: publicar no es distribuir.
- **No hay nada en español.** Las cuatro búsquedas en castellano devuelven 0 plantillas.

Y la consecuencia práctica: **mientras el régimen y los empleados estén en una nota de texto, ningún recordatorio puede calcular los modelos**. La pieza que falta no es un nodo, es el modelo de datos.

## El sistema mínimo: cinco piezas

1. **Una ficha por cliente con lo que decide el calendario:** régimen, si tiene empleados, si está activo y la fecha de alta.
2. **El calendario como datos, no como memoria:** cada modelo con su día, su mes y su condición («solo si tiene IVA», «solo si tiene empleados»).
3. **Una antelación fija y generosa:** 21 días. Con tres días no se avisa de un plazo, se avisa de un problema.
4. **Un solo aviso al día, agrupado y para el equipo.** Cuarenta mensajes sueltos se ignoran; una lista ordenada, se lee.
5. **Baja sin borrar:** el cliente pasa a `Activo = No`, conserva su historial y deja de generar avisos.

## Cómo está montado el flujo (11 nodos, 20-30 minutos)

El flujo de referencia tiene dos partes independientes en el mismo lienzo:

- **Alta de cliente:** formulario → normalización de datos → guardar fila en la hoja → aviso al equipo con el nombre y el régimen.
- **Recordatorios (cada día a las 08:00):** leer clientes → calcular vencimientos de los próximos 21 días → **un único mensaje** a Telegram con lo que viene, ordenado por fecha. Si no hay nada en 21 días, el cálculo devuelve 0 elementos y no se envía nada: el silencio también es una función.

Los modelos que cubre la versión 1.0.0, con las fechas generales del calendario estatal:

| Modelo | Qué es | Vence | A quién avisa |
|---|---|---|---|
| 303 | IVA trimestral | 20 ene / abr / jul / oct | regímenes con IVA |
| 130 | IRPF pago fraccionado (estimación directa) | 20 ene / abr / jul / oct | autónomos en directa |
| 131 | IRPF pago fraccionado (módulos) | 20 ene / abr / jul / oct | autónomos en módulos |
| 111 | Retenciones del trabajo | 20 ene / abr / jul / oct | clientes con empleados |
| 390 | Resumen anual de IVA | 30 de enero | regímenes con IVA |
| 190 | Resumen anual de retenciones | 31 de enero | clientes con empleados |
| 347 | Operaciones con terceros | 28 de febrero | regímenes con IVA |
| 100 | Declaración de la renta | 30 de junio | autónomos en directa |

**Aviso importante:** son las fechas generales estatales. El calendario foral (Navarra, País Vasco) cambia, y confirmar la fecha oficial es trabajo del asesor, no del flujo.

## Montarlo en 20-30 minutos

Requisitos reales: una instancia de **n8n** (self-hosted gratis), una hoja de Google y un bot de Telegram para el chat del equipo.

1. Copia las cabeceras de la hoja tal cual: `Nombre | NIF | Email | Telefono | Regimen | Empleados | Notas | Fecha alta | Activo`. Cambiar un nombre de columna rompe la escritura.
2. Conecta tres credenciales: Google Sheets, el token del bot de Telegram y el chat del equipo.
3. Ajusta el calendario en el bloque de código: `DIAS_AVISO = 21` y una línea por modelo con su condición.
4. Da de alta un cliente de prueba, ejecuta los recordatorios, comprueba el mensaje y borra la fila.

Si es tu primer flujo, empieza por **[N8N para principiantes: tu primer flujo en 30 minutos](/ia-para-negocios/posts/n8n-principiantes-30-minutos.html)**. Y si lo que duele es la documentación del cliente, está en **[cómo pedirla sin perseguirlo](/ia-para-negocios/posts/documentos-de-clientes-sin-perseguir.html)**: los dos flujos comparten la misma ficha de cliente.

## Los tres errores que hunden el sistema

1. **Avisar con poco margen.** A 3 días vista solo puedes rezar. A 21 días puedes pedir papeles y corregir un dato.
2. **Un mensaje por cliente.** Con 40 clientes son 40 notificaciones al día y el equipo silencia el chat. Un mensaje agrupado se lee entero.
3. **Mezclar aviso interno con comunicación al cliente.** El recordatorio al equipo es una herramienta de trabajo; escribir al cliente es otra cosa, con otro tono y otra base legal. Si se mezclan, el flujo acaba pareciendo spam y se apaga.

## RGPD en cinco líneas

- Son datos identificativos y de contacto de clientes de la asesoría; la base habitual es la ejecución del contrato y el cumplimiento de obligaciones legales.
- El flujo corre en **tu** n8n con **tus** credenciales: los datos no pasan por servidores nuestros.
- Si usas Google Sheets, necesitas el contrato de encargado de tratamiento (incluido en Google Workspace).
- Define una retención razonable y borra lo que ya no necesites.
- El chat de Telegram con los avisos debe estar limitado al personal autorizado.

## Lo que este flujo no hace

No presenta modelos ante la AEAT, no calcula cuotas ni intereses de demora, no interpreta normativa foral y no decide si un cliente debe estar dado de alta: eso es criterio profesional. Lo que hace es que ninguna fecha dependa de que alguien se acuerde.

## Si prefieres el atajo

El flujo completo —guía de instalación, guía del calendario, plantillas de mensajes y de RGPD— va dentro del **[Pack Asesoría de FlowKit (29 €)](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria)**, con el onboarding documental y el panel de KPIs: tres automatizaciones y licencia de uso en tu despacho y con tus clientes.

Si aún decides por dónde empezar, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)**. Y el **[catálogo de flujos gratuitos](https://mrnpvn.github.io/flowkit/)** tiene los flujos abiertos para importar hoy, incluida la **[guía de cobros vencidos](https://mrnpvn.github.io/flowkit/cobros-vencidos/)** para cuando el problema no es la fecha sino el impago.

## Preguntas frecuentes

**¿Qué pasa si cambia el calendario fiscal?** Se edita un bloque de código con una línea por modelo; es un cambio de un minuto.

**¿Sirve si ya uso un ERP?** Sí: exporta la cartera de clientes a la hoja o sustituye solo el nodo de lectura.

**¿El aviso le llega al cliente?** No. Va al chat interno del equipo, que es donde se decide qué pedir y cuándo presentar.

**¿Necesito n8n de pago?** No. La versión self-hosted es gratuita; el coste real es el servidor donde la tengas.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Cómo no perder un plazo fiscal de un cliente (sin libreta ni Excel)",
  "description": "Sistema de cinco piezas para que ningún modelo fiscal de tus clientes se pase de fecha: ficha que decide el calendario, aviso a 21 días, un solo mensaje al día y baja sin borrar. Con dato medido del catálogo n8n.",
  "inLanguage": "es",
  "datePublished": "2026-10-26",
  "dateModified": "2026-10-26",
  "author": {
    "@type": "Person",
    "name": "MRNPVN",
    "url": "https://mrnpvn.gumroad.com"
  },
  "publisher": {
    "@type": "Organization",
    "name": "MRNPVN",
    "url": "https://mrnpvn.gumroad.com"
  },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/plazos-fiscales-sin-perder-un-cliente.html"
  },
  "about": ["plazos fiscales", "automatización n8n", "asesorías y gestorías"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Qué pasa si cambia el calendario fiscal?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Se edita un bloque de código con una línea por modelo; es un cambio de un minuto. Conviene revisarlo una vez al mes y anotar la fecha del cambio."
      }
    },
    {
      "@type": "Question",
      "name": "¿Sirve si ya uso un ERP?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí: exporta la cartera de clientes a la hoja o sustituye solo el nodo de lectura. El resto del flujo no cambia."
      }
    },
    {
      "@type": "Question",
      "name": "¿El aviso le llega al cliente?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. Va al chat interno del equipo, que es donde se decide qué pedir y cuándo presentar."
      }
    },
    {
      "@type": "Question",
      "name": "¿Necesito n8n de pago?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. La versión self-hosted es gratuita; el coste real es el servidor donde la tengas."
      }
    }
  ]
}
</script>
