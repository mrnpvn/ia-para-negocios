---
title: "Facturas vencidas: el aviso que se cobra es uno por cliente, no uno por factura"
description: "Por qué reclamar factura por factura alarga la mora y cómo montar el sistema que sí cobra: un solo correo por cliente con todas sus facturas, tres niveles que no se repiten y un enlace que registra el pago. Con dato medido del catálogo n8n y el flujo de 29 nodos que lo hace."
---

# Facturas vencidas: el aviso que se cobra es uno por cliente, no uno por factura

Un despacho cierra el mes con catorce facturas pendientes de tres clientes. Alguien filtra la hoja por «vencidas» y manda un recordatorio a cada fila: el cliente A recibe cuatro correos en la misma mañana, el cliente B tres y el que más debe, cinco. A mediodía uno llama enfadado, y no es para pagar. El aviso estaba mal construido: **se envió por factura cuando la relación es por cliente**.

## El dato: 25 plantillas de recordatorio y la mediana de vistas es cero

Medido el **29 de octubre de 2026** recorriendo todas las páginas de la API pública del catálogo de n8n (`api.n8n.io/api/templates/search`):

| Búsqueda | Plantillas | Vistas (mediana) | Con 0 vistas | La más vista |
|---|---|---|---|---|
| `invoice reminder` | 25 | 0 | **19 de 25** | 28 — recordatorios con iFirma y Gmail |
| `overdue invoice` | 17 | 0 | 13 de 17 | 28 |
| `payment reminder` | 24 | 0 | 20 de 24 | 1.066 — tarjetas de bancos de Taiwán |
| `dunning` | 1 | 0 | 1 de 1 | 0 |
| `cobros` | **0** | — | — | — |
| `impago` | **0** | — | — | — |
| `reclamar pago` | 1 | 0 | 1 de 1 | 0 |
| `factura vencida` | 2 | 0 | 2 de 2 | 0 |

- **El nicho entero está a cero.** Lo más visto en «recordatorio de factura vencida» tiene **28 visitas**. La única con audiencia (1.066) resuelve otro problema: tarjetas de un banco taiwanés.
- **Todas son «un correo y adiós».** Ninguna guarda el estado que decide la mora: nivel de aviso de cada factura, fecha del último envío, qué se pagó y qué quedó sin reclamar.
- **En español no hay nada.** `cobros` e `impago` devuelven cero resultados.

Un catálogo vacío no es una oportunidad de marketing: es la prueba de que **el recordatorio suelto no es el mecanismo**.

## Por qué el aviso por factura no cobra

1. **Duplica y desgasta.** Cuatro avisos para cuatro facturas; el cuarto ya no es un recordatorio, es una amenaza.
2. **Escala a destiempo.** Al que debe 900 € desde hace tres meses se le escribe igual que al que se le pasó un recibo de 80 € el martes. El tono deja de informar.
3. **No sabe qué se ha pagado.** Sin registro se reclama lo cobrado o se calla lo vivo; los dos errores cuestan credibilidad.
4. **No deja rastro.** «¿A este ya le hemos reclamado?» se responde buscando en el buzón de una persona. Eso no es un proceso, es memoria.

## El mecanismo en cinco piezas

Ninguna necesita IA. Todas necesitan **una hoja con estado**.

1. **Agrupar por cliente, no por factura.** Un correo por dirección de email con todas sus facturas vencidas, el total y los datos de transferencia una sola vez.
2. **Tres niveles declarados.** Primer aviso a los **5 días** del vencimiento, segundo a los **15**, tercero a los **30**. Son constantes al principio del código; lo que no se cambia es que existan por escrito.
3. **Un nivel se envía una sola vez.** Cada factura guarda su `Nivel`; si el calculado no lo supera, **no se envía nada**. Quien lleva 40 días recibe el nivel 3, no el 1 «de bienvenida».
4. **Nunca dos avisos el mismo día.** Si `Ultimo_Recordatorio` es hoy, la fila se salta. Es lo que impide el bombardeo cuando alguien reejecuta el flujo.
5. **Un enlace que registra el pago.** Cada correo lleva una dirección con las facturas de ese cliente: al abrirla, marca el pago sin usuario ni contraseña, el equipo recibe el aviso y la factura pasa a `Pagada`. El registro no depende de que alguien conteste.

El tono sube con el nivel y la regla no cambia: el nivel 1 deja abierta la puerta al retraso bancario; el 2 pide **fecha prevista de pago**; el 3 pide regularizar esa semana y ofrece aplazamiento. El cliente pequeño y el grande reciben el mismo proceso.

## Cómo es el flujo de verdad

El flujo del pack tiene **29 nodos** —24 funcionales y 5 notas— y cuatro partes independientes:

- **A) Recordatorios (cada día, 09:00)** — lee la hoja → descarta lo que no está `Pendiente` → calcula días de mora → aplica nivel y reglas → agrupa por cliente → un correo por cliente → marca nivel, contador y fecha del último aviso.
- **B) Informe de cartera (lunes, 09:00)** — total pendiente, tramos **al día · 1-30 · 31-60 · 61-90 · +90 días**, cinco mayores deudores y dos avisos: facturas **sin email** y facturas con **fecha inválida**.
- **C) Enlace «marcar como pagada»** — valida el enlace, actualiza la hoja, avisa por Telegram y devuelve una página HTML: **200** si es válido, **400** si no.
- **D) Red de seguridad** — si un nodo falla tras los reintentos, un Error Trigger avisa con el nodo y el mensaje exactos.

La hoja tiene **13 columnas**: `ID | Cliente | Email | Concepto | Importe | Fecha_Emision | Fecha_Vencimiento | Estado | Nivel | Recordatorios | Ultimo_Recordatorio | Fecha_Pago | Notas`. `Estado` admite `Pendiente`, `Pagada` e `Incobrable`; `Nivel`, `Recordatorios` y `Ultimo_Recordatorio` se rellenan solos. Los importes se leen en formato español (`1.234,56`) o con punto decimal, y las fechas en `dd/mm/aaaa` o `aaaa-mm-dd`.

Dos detalles deciden si aguanta: **si el correo falla, la fila no se marca** (se reintenta al día siguiente), y **el enlace usa un identificador no adivinable**, porque con IDs predecibles cualquiera marca como pagada la deuda de otro.

## Montarlo: una tarde y tres credenciales

Google Sheets, SMTP y Telegram; ninguna de pago. Antes de activar hay que tocar tres constantes: URL pública de tu n8n, nombre de la empresa e IBAN. La prueba que vale no es «envía un correo», son cuatro filas:

| Fila | Vence | Estado | Qué debe pasar |
|---|---|---|---|
| F-001 | hace 6 días | `Pendiente` | Nivel 1, un correo |
| F-002 | hace 20 días | `Pendiente` | Nivel 2; mismo cliente que F-001 → **un solo correo con las dos** |
| F-003 | hace 35 días | `Pendiente` | Nivel 3 |
| F-004 | hace 40 días | `Pagada` | **Ningún correo** |

Si llegan tres correos en vez de dos, la agrupación está mal. Si F-004 recibe algo, el filtro de estado está mal. Si el segundo día salen los mismos avisos, la regla de «una vez por nivel» no lee la hoja.

## Cuatro errores que lo hunden

- **No separar `Pagada` de `Incobrable`.** Todo lo que no sea `Pendiente` deja de reclamarse, y eso incluye la deuda que das por perdida: el informe mentirá sobre cuánto esperas cobrar.
- **Facturas sin email.** El flujo no las inventa; las cuenta en el informe del lunes.
- **Un IBAN mal puesto en el correo.** Debe ser el de tu empresa y comprobarse antes de activar; un dígito mal y el cliente transfiere a una cuenta inexistente.
- **Medir correos enviados.** El indicador es la **antigüedad media de la cartera** y los días que tarda en cobrarse cada tramo; enviar más y cobrar igual es empeorar.

## Lo que este flujo no hace

No calcula **intereses de demora** ni los reclama: es una decisión jurídica y contable. No presenta monitorios ni comunica a ficheros de morosidad. No toca tu ERP ni tu programa de facturación: lee y escribe en una hoja que tú controlas. Y no decide aplazamientos: los propone y los aprueba una persona.

## Lo que hay que decir del RGPD

Nombre, NIF, importes y concepto son datos personales si el cliente es autónomo o persona física. Base: ejecución del contrato y gestión de cobro. La hoja necesita retención definida (la fiscal manda) y forma de borrar o anonimizar. El flujo corre en **tu** n8n con **tus** credenciales y no envía nada a terceros. Nada de datos de salud ni de terceros en `Notas`.

## Si prefieres el atajo

La guía del mecanismo —escalera, errores que alargan la mora, métricas y cuándo no compensa— está **gratis**: **[Cobros vencidos: reclamar sin perder al cliente](https://mrnpvn.github.io/flowkit/cobros-vencidos/)**.

El flujo montado —**29 nodos, las dos pestañas, los tres textos por nivel, el enlace de pago y el informe del lunes**— va en el **[Pack Asesoría de FlowKit (29 €)](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria)**, con el onboarding documental y el panel de KPIs. Si empiezas de cero, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)**.

Para cerrar el círculo: **[los cinco números del lunes](/ia-para-negocios/posts/cinco-numeros-despacho-lunes.html)**, **[no perder un plazo fiscal](/ia-para-negocios/posts/plazos-fiscales-sin-perder-un-cliente.html)** y, si nunca has importado un flujo, **[tu primer flujo en 30 minutos](/ia-para-negocios/posts/n8n-principiantes-30-minutos.html)**.

## Preguntas frecuentes

**¿Por qué un correo por cliente y no por factura?** Porque la relación es con el cliente. Agrupando por email se envía un mensaje con todas sus facturas, el total y los datos de transferencia una sola vez.

**¿Cuántos días antes de reclamar?** No hay cifra legal única: depende de lo pactado. Conviene que el primero salga a los pocos días del vencimiento, cuando aún se lee como recordatorio administrativo. El flujo usa 5, 15 y 30 días, editables.

**¿Cómo evita reclamar dos veces lo mismo?** Con dos campos de la hoja: `Nivel` (si el calculado no lo supera, no se envía) y `Ultimo_Recordatorio` (si es hoy, la fila se salta).

**¿Hace falta una IA o un CRM?** No. Sheets, SMTP y Telegram. La decisión es aritmética de fechas y comparación de niveles; una IA no la mejora y añade cuenta de pago y caja negra.

**¿El enlace de pago es seguro?** Es un atajo de registro, no autenticación: solo cambia el estado de esas facturas y el identificador debe ser no adivinable.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Facturas vencidas: el aviso que se cobra es uno por cliente, no uno por factura",
  "description": "Por qué reclamar factura por factura alarga la mora y cómo montar el sistema que sí cobra: un solo correo por cliente con todas sus facturas, tres niveles que no se repiten y un enlace que registra el pago.",
  "inLanguage": "es",
  "datePublished": "2026-10-29",
  "dateModified": "2026-10-29",
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
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/facturas-vencidas-aviso-por-cliente.html"
  },
  "about": ["facturas vencidas", "cobros", "automatización n8n", "gestión de cartera"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Por qué un solo correo por cliente y no uno por factura?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Porque la relación es con el cliente, no con cada factura. Un cliente con cuatro facturas vencidas recibe cuatro avisos si se envía por factura, y el cuarto deja de leerse como recordatorio. Agrupando por email se envía un mensaje con todas sus facturas, el total y los datos de transferencia una sola vez."
      }
    },
    {
      "@type": "Question",
      "name": "¿Cuántos días hay que esperar antes de reclamar una factura?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No hay una cifra legal única: depende de lo pactado en el contrato y del plazo de pago aplicable. Conviene que el primer aviso salga a los pocos días del vencimiento, cuando aún se interpreta como recordatorio administrativo. El flujo usa 5, 15 y 30 días como valores por defecto editables."
      }
    },
    {
      "@type": "Question",
      "name": "¿Cómo evita el sistema reclamar dos veces la misma factura?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Cada factura guarda en la hoja su Nivel y la fecha del Ultimo_Recordatorio. Si el nivel calculado no supera el guardado, no se envía nada; y si el último aviso es de hoy, la fila se salta. Así cada nivel se envía una sola vez y nunca hay dos avisos el mismo día, aunque se reejecute el flujo."
      }
    },
    {
      "@type": "Question",
      "name": "¿Hace falta una IA o un CRM para esto?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. El flujo usa Google Sheets, SMTP y Telegram. Decidir a quién y con qué tono se escribe es aritmética de fechas y una comparación de niveles; una IA no mejora esa decisión y añade una cuenta de pago y una caja negra."
      }
    },
    {
      "@type": "Question",
      "name": "¿El enlace de pago es seguro?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Es un atajo de registro, no una autenticación: sirve para que el cliente que ya recibe la factura marque el pago sin usuario ni contraseña. Por eso el identificador debe ser no adivinable y solo permite cambiar el estado de esas facturas, sin abrir ningún otro dato."
      }
    }
  ]
}
</script>
