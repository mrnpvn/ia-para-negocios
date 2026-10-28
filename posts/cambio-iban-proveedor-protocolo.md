---
title: "Cambio de IBAN de un proveedor: cómo verificarlo sin fiarte del correo"
description: "El aviso del cambio de cuenta llega dentro del hilo real, con el logo correcto y la factura adjunta. El protocolo que sí lo detiene: canal independiente, enfriamiento, enlace de un solo uso al destinatario que ya tenías y auditoría. Con dato medido del catálogo n8n y el flujo de 60 nodos que lo implementa."
---

# Cambio de IBAN de un proveedor: cómo verificarlo sin fiarte del correo

Son las 08:40 de un viernes. Llega un correo del proveedor de siempre: «hemos cambiado de banco, a partir de ahora paga en esta cuenta». Viene **dentro del hilo real**, con el logo, la firma y la factura del mes adjunta. Nada parece raro, porque **no hay nada raro**: el buzón del proveedor está comprometido y el atacante escribe desde él.

A las 12:00, contabilidad ha actualizado la cuenta en el ERP. La transferencia sale el lunes y llega a quien la espera. El fallo no fue de nadie en concreto: fue del **canal**. Se verificó un cambio de datos bancarios usando el mismo medio que el atacante controlaba.

## Por qué los tres controles habituales no controlan nada

Casi todas las empresas creen estar cubiertas con alguno de estos tres, y ninguno toca el problema:

- **Validar el IBAN.** El dígito de control (mod-97) detecta un error al teclear. La cuenta del atacante es una cuenta real, con su dígito correcto.
- **Confirmar respondiendo al correo.** Es el mismo buzón que está comprometido: le preguntas al atacante si es el atacante.
- **«Avisar» al proveedor de que no cambie nada.** Un aviso no es una aprobación: no bloquea la actualización ni deja rastro de quién la autorizó.

Lo que falta no es un control más, sino un **estado**: que la petición exista en algún sitio, que no se pueda aplicar sola y que alguien con un canal distinto la libere.

## El dato: el mecanismo no está publicado

Antes de construir nada, se midió qué existe. Recorrido completo de la API pública del catálogo de n8n el **28 de octubre de 2026** (`api.n8n.io/api/templates/search`, todas las páginas de cada término):

| Búsqueda | Plantillas | Vistas (mediana) | Con 0 vistas | La más vista |
|---|---|---|---|---|
| `bank change` | 1 | 0 | 1 de 1 | 0 |
| `vendor bank` | 1 | 0 | 1 de 1 | 0 — la misma plantilla |
| `iban` | 26 | 0 | 16 de 26 | 1.210 — carrito de Shopify, no es esto |
| `bank account change` | 28 | 0 | 23 de 28 | 14.662 — extractos a markdown |
| `fraud` | 15 | 0 | **15 de 15** | 0 |
| `invoice fraud` | 1 | 0 | 1 de 1 | 0 |
| `kyb` | 3 | 0 | 2 de 3 | 2 |
| `proveedor` · `cuenta bancaria` | 0 · 0 | — | — | — |

Tres lecturas:

- **La única plantilla del tema no resuelve el circuito.** Publicada en agosto, hace cribado con un modelo y DNS, y **lleva 0 vistas**. Ninguna tiene estado que sobreviva entre ejecuciones, enfriamiento y enlace de un solo uso.
- **Las 15 plantillas de `fraude` están a cero.** El fraude se automatiza para *detectar* el intento, casi nunca para **impedir que se aplique**.
- **En español no existe nada**: `proveedor` y `cuenta bancaria` devuelven 0 resultados.

## El mecanismo en seis piezas

Ninguna pieza necesita inteligencia artificial. Todas necesitan que alguien decida que el cambio **no** se aplica solo:

1. **Retención, no aplicación.** La petición entra en una hoja con estado (`PENDING_VERIFICATION`) y ahí se queda hasta que hay una decisión registrada. El ERP no se toca.
2. **El aviso va a la dirección que ya tenías**, la de la ficha del proveedor, no la del correo que acaba de llegar. Si el atacante controla el buzón del proveedor, sigue sin poder aprobar su propio cambio.
3. **Enfriamiento.** 48 horas por defecto; 72 si hay señales de riesgo. El tiempo es el control: el fraude funciona por urgencia.
4. **Enlace de un solo uso, guardado como hash.** De un token largo solo queda su SHA-256: una hoja filtrada no sirve para aprobar nada, los recordatorios no lo reenvían y un segundo clic responde «ya está gestionado» sin mover el estado.
5. **Confirmación telefónica con el número de siempre.** El flujo lleva hasta el borde del banco y ahí se para: la cuenta se cambia a mano tras una llamada.
6. **Rastro que no se puede editar.** Denegaciones, duplicados y caducidades van a una pestaña de auditoría append-only, y un vigilante diario escala a las 24, 48 y 72 horas hasta cerrar como `EXPIRED`.

## Cómo es el flujo de verdad

El flujo abierto (`vendor-bank-change-guard.json`) tiene **60 nodos —54 funcionales y 6 notas—, 44 grupos de conexión y cinco integraciones**: Google Sheets, SMTP, Telegram, formulario y webhook. Tres partes independientes, para que un fallo en una no tumbe las otras:

- **A) Entrada.** Formulario → normalizar el IBAN → formato, **mod-97** y país del IBAN contra país del proveedor → emparejar con el proveedor → puntuar riesgo → detectar duplicado → guardar con estado y token hasheado → correo al contacto de finanzas que ya constaba.
- **B) Decisión con un clic.** Si el token es inválido, desconocido, caducado o ya usado, el webhook devuelve **400** y lo anota; si se pulsa **antes** de que acabe el enfriamiento, responde «sigue en el periodo de enfriamiento» y **no toca el estado**; si se aprueba, marca `APPROVED` y avisa a finanzas con el IBAN enmascarado.
- **C) Vigilante diario a las 07:30.** Recordatorio a las 24 h, escalado al responsable a las 48 h, cierre como `EXPIRED` a las 72 h y un resumen con las peticiones abiertas y las de riesgo alto.

La puntuación de riesgo es lo que separa un aviso de un control: proveedor desconocido → revisión manual y **nunca** enlace de aprobación (100 puntos); proveedor inactivo +40; país del IBAN distinto +30; dominio del solicitante distinto al del proveedor +25; buzón gratuito +15; un cambio en los últimos 90 días +15, dos o más +30.

## Montarlo: 10-30 minutos

Tres pestañas con cabeceras exactas —`Vendors`, `BankChanges`, `Audit`—, tres nodos de configuración con la URL pública de n8n, el enfriamiento y el destinatario de escalado, y tres credenciales (Sheets, SMTP y Telegram).

La prueba que vale son cinco minutos: date de alta a ti mismo como proveedor, envía el formulario con un IBAN válido y comprueba que la fila queda en `PENDING_VERIFICATION`, que el correo llega a la dirección que ya constaba y que el enlace responde «sigue en el periodo de enfriamiento».

## Cuatro errores que lo hunden

- **Poner en la ficha del proveedor el correo que llega en la petición.** Es el error que anula todo lo demás: el atacante aprueba su propio cambio.
- **Guardar el token en claro** o mandarlo en el resumen diario: el hash es la defensa barata.
- **Aprobar desde el correo del comercial.** La aprobación va a finanzas, que es quien aplica el cambio.
- **Aplicar el cambio sin llamar.** El flujo demuestra que finanzas aprobó; no que la cuenta sea del proveedor.

## Lo que este flujo no hace

No verifica que el IBAN pertenezca al proveedor (los bancos no lo exponen), no comprueba listas de sanciones ni PEP, no aplica el cambio en tu banco ni en tu ERP y no demuestra la identidad de quien rellena el formulario: demuestra que **finanzas aprobó** el cambio tras un enfriamiento. Todo lo demás sigue siendo humano.

## RGPD en cuatro líneas

Se tratan datos del solicitante, contactos del proveedor y datos bancarios. Base habitual: ejecución de contrato más interés legítimo en prevención del fraude. Hoja restringida al rol de finanzas, dos pasos en la cuenta de Google y retención definida (por ejemplo, 12 meses). Los correos llevan el IBAN enmascarado.

## Si prefieres el atajo

El flujo completo —**60 nodos, tres partes, las cabeceras de las tres pestañas y las 45 pruebas de lógica**— está abierto en el repositorio: **[vendor-bank-change-guard.json](https://github.com/mrnpvn/flowkit/blob/main/workflows/finance/vendor-bank-change-guard.json)** y su **[guía paso a paso](https://github.com/mrnpvn/flowkit/blob/main/docs/FLOW-VENDOR-BANK-CHANGE-GUARD.md)**. Gratis, licencia MIT, se importa en dos minutos.

Si quieres el procedimiento escrito para el equipo, la guía **[fraude del cambio de IBAN: los tres controles que no sirven](https://mrnpvn.github.io/flowkit/fraude-proveedores/)** lo desarrolla, y el **[catálogo de flujos gratuitos](https://mrnpvn.github.io/flowkit/)** tiene los demás. Para el resto del despacho, el **[Pack Asesoría](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria)** añade cobros vencidos, onboarding documental y panel de KPIs; si prefieres empezar por lo pequeño, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)**.

Dentro del sitio, dos complementos naturales: **[cómo pedir la documentación a un cliente sin perseguirlo cinco veces](posts/documentos-de-clientes-sin-perseguir.html)** y **[los cinco números que un despacho debe mirar el lunes](posts/cinco-numeros-despacho-lunes.html)**: un cambio de cuenta sin control es el tipo de agujero que no aparece en ningún KPI hasta que ya es una pérdida. Si aún no has montado nada en n8n, empieza por **[tu primer flujo en 30 minutos](posts/n8n-principiantes-30-minutos.html)**.

## Preguntas frecuentes

**¿Esto sustituye la llamada al proveedor?** No: la hace obligatoria. Lo que evita es que la llamada sea la *única* barrera.

**¿Por qué 48 horas y no aprobar el mismo día?** Porque la urgencia es la herramienta del fraude: un cambio legítimo espera dos días sin coste; uno falso depende de que nadie espere.

**¿Qué pasa si se pulsa aprobar antes del enfriamiento?** Responde que sigue en el periodo de enfriamiento y no modifica el estado. Un segundo clic responde «ya está gestionado» sin cambiar nada.

**¿Qué pasa si el aprobador está de vacaciones?** A las 48 horas escala al responsable de finanzas y a las 72 cierra la petición como caducada, avisando al solicitante de que no se aplicó nada.

**¿Puedo usarlo con Outlook en vez de SMTP?** Sí: los seis nodos de correo llevan `to`, `subject` y `body` por elemento y se cambian por el nodo de Microsoft Outlook.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Cambio de IBAN de un proveedor: cómo verificarlo sin fiarte del correo",
  "description": "El aviso del cambio de cuenta llega dentro del hilo real, con el logo correcto y la factura adjunta. El protocolo que sí lo detiene: canal independiente, enfriamiento, enlace de un solo uso al destinatario que ya tenías y auditoría. Con dato medido del catálogo n8n y el flujo de 60 nodos que lo implementa.",
  "inLanguage": "es",
  "datePublished": "2026-10-28",
  "dateModified": "2026-10-28",
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
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/cambio-iban-proveedor-protocolo.html"
  },
  "about": ["cambio de IBAN", "fraude del proveedor", "automatización n8n", "verificación de proveedores"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Este flujo sustituye la llamada al proveedor?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No, la hace obligatoria. El flujo retiene el cambio, envía el enlace de aprobación al contacto de finanzas que ya constaba y obliga a confirmar por teléfono con el número de siempre antes de tocar el banco. Lo que evita es que la llamada sea la única barrera."
      }
    },
    {
      "@type": "Question",
      "name": "¿Por qué un enfriamiento de 48 horas antes de aprobar?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Porque la urgencia es la herramienta del fraude. Un cambio legítimo espera dos días sin coste operativo; uno falso depende de que nadie espere. El flujo usa 72 horas cuando la petición suma señales de riesgo."
      }
    },
    {
      "@type": "Question",
      "name": "¿Qué pasa si se pulsa el enlace de aprobar antes del enfriamiento?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "El webhook responde que la petición sigue en el periodo de enfriamiento y no modifica el estado. Aprobar solo es posible después de cooling_until, y el enlace es de un solo uso: un segundo clic responde que ya está gestionado sin cambiar nada."
      }
    },
    {
      "@type": "Question",
      "name": "¿Dónde se guarda el token del enlace?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Solo su hash SHA-256 en la hoja. El token viaja en el correo y nunca se vuelve a enviar en los recordatorios, porque reenviarlo sería una segunda copia de la credencial. Una hoja filtrada no permite aprobar ningún cambio."
      }
    },
    {
      "@type": "Question",
      "name": "¿Se puede usar con Outlook en lugar de SMTP?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí. Los seis nodos de correo llevan los campos to, subject y body preparados por elemento, así que se sustituyen por el nodo de Microsoft Outlook sin tocar el resto del flujo."
      }
    }
  ]
}
</script>
