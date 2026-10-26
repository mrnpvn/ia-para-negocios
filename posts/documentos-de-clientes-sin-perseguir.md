---
title: "Cómo pedir la documentación a un cliente sin perseguirlo cinco veces"
description: "Sistema de cuatro piezas para recoger documentación de clientes sin perseguirlos: checklist por servicio, estado por documento, recordatorios escalonados e informe semanal. Con dato medido del catálogo n8n y el flujo de 51 nodos que lo implementa."
---

# Cómo pedir la documentación a un cliente sin perseguirlo cinco veces

Un cliente nuevo, siete documentos, tres canales (WhatsApp, teléfono y correo) y un Excel que nadie actualiza. Ese es el proceso real de la mayoría de asesorías, gestorías y despachos, y explica por qué un alta que debería cerrarse en una semana se va a un mes.

El problema casi nunca es que el cliente no quiera enviarlo. Es que **nadie le ha dicho, hoy y con su nombre delante, qué le falta exactamente**.

## Por qué el Excel compartido y el WhatsApp no funcionan

No es una cuestión de disciplina del equipo: es que la herramienta no guarda el estado que hace falta.

- **La lista vive en la cabeza de una persona.** Cuando esa persona coge vacaciones, el alta se para y nadie sabe por qué.
- **Una fila por cliente no dice nada del documento.** "Pendiente" no distingue entre *no lo he pedido*, *lo pedí el martes*, *lo pidió otro compañero* y *lo mandó mal*.
- **Los recordatorios son manuales y emocionales.** Se escriben cuando hay un hueco, con el tono del día, y se dejan de escribir en cuanto entra trabajo urgente.
- **Nadie mide el atasco.** Sin un dato semanal, el "los clientes tardan mucho" es una sensación compartida en la oficina, no un problema con número.

## El sistema mínimo: cuatro piezas

Cualquier solución que funcione tiene estas cuatro, sea en una herramienta de pago o montada a mano:

1. **Una checklist distinta por servicio.** Un alta fiscal no pide lo mismo que un alta laboral. Si la lista es genérica, o pides de más (y molestas) o de menos (y vuelves a pedir).
2. **Una ficha por documento, no por cliente.** Así el estado es verificable: pedido → recibido → validado, con fecha en cada paso.
3. **Recordatorios escalonados, con tope y con tono.** Días 3, 7, 14 y 21, un solo mensaje por cliente y día, y el último avisa de que el expediente queda bloqueado. Sin tope, el sistema se convierte en spam y el cliente deja de leer.
4. **Un informe semanal que diga dónde se atasca.** Cuántos expedientes abiertos, cuántos completos, antigüedad media y qué documento concreto se repite como cuello de botella.

## El dato: este mecanismo casi no existe como plantilla

Antes de construir nada conviene saber qué hay hecho. Medido el **26 de octubre de 2026** consultando todas las páginas de resultados de la API pública del catálogo de n8n (`api.n8n.io/api/templates/search`):

| Búsqueda | Plantillas | Vistas (mediana) | La más vista |
|---|---|---|---|
| `invoice` | 344 | 0 | 1.145 |
| `onboarding` | 75 | 0 | 935 |
| `dossier` | 3 | 0 | 0 |
| `collect documents` | **1** | **0** | 0 |

Dos lecturas, y la segunda importa más que la primera:

- **No hay plantilla hecha para esto.** Una sola coincide con "collect documents", publicada en febrero de 2026, con cero visitas. No es un hueco de marketing: es que el mecanismo (checklist + estado por documento + escalado + informe) no se publica porque no cabe en un tutorial de cinco nodos.
- **El catálogo tiene mucho "aviso" y poco "sistema".** De las 75 plantillas que aparecen con `onboarding`, la mediana de visitas es 0 y solo una pasa de 900. Casi todas mandan un correo y paran ahí; ninguna lleva la cuenta de qué falta ni cierra el bucle cuando el cliente envía el papel.

Ese es exactamente el motivo por el que montar esto a mano cuesta días y por el que, cuando está montado, nadie lo desmonta.

## Cómo funciona por dentro (el flujo que ya está construido)

El flujo de onboarding documental que usamos como referencia tiene **51 nodos** y seis partes independientes:

- **Alta:** formulario → comprobación de duplicado por correo → **checklist automática según servicio** (fiscal 7 documentos, laboral 5, constitución 6, contabilidad 5, a medida 4) → una fila por documento en la hoja `Docs` con un identificador `token-CÓDIGO` → un correo al cliente con su checklist y un enlace privado → aviso al equipo con el ID de cada documento.
- **Recordatorios (09:00):** un solo correo por cliente con lo que le falta, en los días 3, 7, 14 y 21, cada nivel una única vez, nunca dos el mismo día, con tono que sube y confirmación de bloqueo a los 21.
- **Portal del cliente:** una dirección con token (`?k=TOKEN`) que muestra su progreso en modo lectura. Sin usuario, sin contraseña, sin acceso a nada más.
- **Recepción:** formulario con correo e IDs → marca `Recibido`, recalcula el porcentaje del expediente, avisa al equipo y, al 100 %, envía el correo de alta completada.
- **Informe semanal (lunes 09:00):** abiertos, completos, tasa, antigüedad media, bloqueados de más de 21 días y los tres documentos que más se atascan.
- **Errores:** cualquier fallo avisa al equipo en vez de quedarse en silencio.

Detalle que suele olvidarse: los envíos de correo van con `continueErrorOutput` y reintentos. Si falla un cliente, no se marca su fila como avisada y el lote sigue; si no, un rebote deja a alguien esperando para siempre.

## Cómo montarlo en 30-45 minutos

Requisitos reales: una instancia de **n8n** (self-hosted gratis o la community), una hoja de Google, una cuenta SMTP y un bot de Telegram para los avisos.

1. Copia la estructura de las dos hojas (`Docs` con 14 columnas, `Onboarding` con 13) tal cual vienen documentadas. Cambiar los nombres de columna rompe el flujo.
2. Conecta las credenciales: Sheets, SMTP y Telegram. Son tres, y se prueban una a una.
3. Ajusta la checklist por servicio a lo que tú pides de verdad. Es el único sitio donde merece la pena personalizar.
4. Lanza un alta de prueba con tu propio correo y sigue el expediente completo: alta, recordatorio, portal, recepción, informe.

Si nunca has importado un flujo, el punto de partida está en **[N8N para principiantes: automatiza tu primer flujo en 30 minutos](/ia-para-negocios/posts/n8n-principiantes-30-minutos.html)**, y los errores típicos de un despacho están en **[Errores al automatizar tu despacho de abogados](/ia-para-negocios/posts/automatizar-despacho-abogados.html)**.

## Los tres errores que hunden el sistema

1. **Pedirlo todo el primer día.** El cliente ve siete casillas y lo deja para mañana. Prioriza dos documentos bloqueantes y deja el resto para el recordatorio del día 3.
2. **Recordar todos los días.** Sin escalado ni tope, el cliente deja de leer y el equipo deja de mirar los avisos. Cuatro niveles y bloqueo es suficiente.
3. **No cerrar el bucle.** Si el cliente manda el papel y no recibe confirmación, vuelve a mandarlo o pregunta si llegó. Un "recibido, quedan 2" ahorra dos correos y una llamada.

## RGPD en cinco líneas

- Pide **solo** lo necesario para el servicio contratado, y dilo en la propia checklist.
- Indica finalidad y plazo de conservación al pedirlo, no después.
- No pidas por correo datos de salud o categorías especiales si el trámite no los exige (art. 9 RGPD).
- Guarda lo recibido en el repositorio acordado, no en el buzón del comercial.
- Define quién puede abrir el portal y con qué token. Un enlace de lectura por cliente, no uno compartido.

## Lo que este flujo no hace

Honestidad por delante: **no lee ni clasifica el contenido** de lo que envía el cliente (no hay OCR ni validación documental), **no firma electrónicamente**, **no sustituye a un CRM** y **no decide qué documento es legalmente exigible**: eso es criterio profesional, no automatización. Lo que hace es que ninguna petición se pierda y que el atasco tenga nombre y número.

## Si prefieres el atajo

El flujo completo, con su guía de implantación de 30 días, la plantilla de registro de actividades y el aviso de privacidad para los correos, está en el **[Pack Asesoría de FlowKit (29 €)](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria)**: tres automatizaciones (cobros vencidos, onboarding documental y panel de KPIs), guías paso a paso y licencia de uso en tu despacho y en tus clientes.

Si todavía estás decidiendo por dónde empezar, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)**: guía de 30 minutos, checklist de prioridades y 10 prompts. Y si quieres ver antes cómo encajan las piezas, el **[catálogo de flujos gratuitos](https://mrnpvn.github.io/flowkit/)** tiene cuatro flujos para importar hoy, incluido el de **[cobros vencidos](https://mrnpvn.github.io/flowkit/cobros-vencidos/)** y el de **[cambios de IBAN de proveedores](https://mrnpvn.github.io/flowkit/fraude-proveedores/)**.

## Preguntas frecuentes

**¿Cuánto tarda en estar funcionando?** Entre 30 y 45 minutos con las credenciales a mano; el alta de prueba es lo que más tarda.

**¿Necesito n8n de pago?** No. La versión self-hosted es gratuita; el coste real es el servidor donde la tengas.

**¿Sirve para un despacho pequeño?** Sí. Con más de cinco clientes activos ya compensa: por debajo, una hoja bien hecha y un recordatorio en el calendario bastan.

**¿Puedo cambiar los plazos de los recordatorios?** Sí, son cuatro valores en el nodo de configuración. Lo que no conviene quitar es el tope: un sistema sin bloqueo no termina nunca.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Cómo pedir la documentación a un cliente sin perseguirlo cinco veces",
  "description": "Sistema de cuatro piezas para recoger documentación de clientes sin perseguirlos: checklist por servicio, estado por documento, recordatorios escalonados e informe semanal. Con dato medido del catálogo n8n y el flujo de 51 nodos que lo implementa.",
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
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/documentos-de-clientes-sin-perseguir.html"
  },
  "about": ["gestión documental", "automatización n8n", "onboarding de clientes"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Cuánto tarda en estar funcionando?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Entre 30 y 45 minutos con las credenciales a mano; el alta de prueba es lo que más tarda."
      }
    },
    {
      "@type": "Question",
      "name": "¿Necesito n8n de pago?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. La versión self-hosted es gratuita; el coste real es el servidor donde la tengas."
      }
    },
    {
      "@type": "Question",
      "name": "¿Sirve para un despacho pequeño?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí. Con más de cinco clientes activos ya compensa: por debajo, una hoja bien hecha y un recordatorio en el calendario bastan."
      }
    },
    {
      "@type": "Question",
      "name": "¿Puedo cambiar los plazos de los recordatorios?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí, son cuatro valores en el nodo de configuración. Lo que no conviene quitar es el tope: un sistema sin bloqueo no termina nunca."
      }
    }
  ]
}
</script>
