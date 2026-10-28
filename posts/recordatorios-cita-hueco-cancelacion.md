---
title: "Recordatorios de cita que se confirman solos (y avisan del hueco cuando alguien cancela)"
description: "Un recordatorio que solo informa no cambia la agenda. El sistema mínimo para que el paciente confirme o cancele con un clic, el estado se actualice solo y el equipo sepa del hueco a las 09:30 antes de perderlo. Con dato medido del catálogo n8n y el flujo de 21 nodos que lo hace."
---

# Recordatorios de cita que se confirman solos (y avisan del hueco cuando alguien cancela)

Son las 09:10 de un martes. Una paciente no aparece; la recepcionista llama y salta el buzón de voz. A las 09:25 el hueco de 45 minutos ya está perdido, y en la sala hay dos personas de pie que la semana pasada no encontraron sitio.

El centro manda un SMS desde hace años, así que no falla el recordatorio: falla que **el recordatorio no tiene forma de volver**. El mensaje sale, el paciente lo lee y la agenda —el único sitio donde el hueco existe— no se entera de nada hasta que alguien llama.

## Por qué el recordatorio que ya mandas no cambia la agenda

Un recordatorio informa. Lo que falta es una **acción** que cambie un estado:

- Si el paciente **confirma**, alguien tiene que saberlo.
- Si **cancela**, el hueco tiene que salir de la agenda **el día antes**, no a las 09:10 del día de la cita: con 18 horas de margen todavía se puede ofrecer; con 20 minutos, no.

Un SMS que dice «responda 1 para confirmar» muere en el móvil del paciente. El salto real es que **el clic escriba en tu agenda** y que la cancelación avise a quien puede rellenar el hueco.

## El dato: hay cientos de recordatorios, y ni uno resuelve el circuito

Antes de construir nada, qué existe. Medido el **28 de octubre de 2026** recorriendo todas las páginas de la API pública del catálogo de n8n (`api.n8n.io/api/templates/search`):

| Búsqueda | Plantillas | Vistas (mediana) | Con 0 vistas | La más vista |
|---|---|---|---|---|
| `appointment reminder` | 20 | 0 | 17 de 20 | 5.335 — recordatorio por **voz** con IA |
| `booking` | 94 | 0 | 59 de 94 | 4.708 — reservas por WhatsApp |
| `clinic` | 20 | 1 | 7 de 20 | 12.047 — gestión con WhatsApp y Calendar |
| `no-show` | 12 | 0 | **12 de 12** | 0 — ninguna tiene una sola visita |
| `confirm appointment` | 3 | 1 | 1 de 3 | 2 |
| `waitlist` | 7 | 0 | 5 de 7 | 1.959 — formulario de lista, no relleno de huecos |
| `lista de espera` | 1 | 0 | 1 de 1 | 0 — y no es de citas |

Tres lecturas:

- **Lo que existe es el mensaje, no el circuito.** Hay recordatorios de voz, de WhatsApp y con IA, y casi todos dependen de cuentas de pago (Twilio, WhatsApp Business, una API de voz). Ninguno de los 3 de `confirm appointment` pasa de **2 visitas**.
- **El relleno del hueco no está publicado.** `waitlist` devuelve formularios de lista de espera y `lista de espera` un falso positivo: nadie ha publicado el tramo que convierte una cancelación en un aviso con hora exacta.
- **Publicar no es distribuir.** Las 12 plantillas de `no-show` están a cero vistas: una plantilla sin mecanismo propio no se encuentra.

## El mecanismo en cinco piezas

Cinco piezas, ninguna con IA:

1. **Una cita, una fila con llave.** Cada cita lleva un `ID` largo y difícil de adivinar: viaja dentro del enlace y permite actualizar *esa* cita y no otra. No se reutiliza entre pacientes.
2. **Un correo la tarde anterior, con dos salidas.** A las 18:00 sale un mensaje por paciente con la fecha, la hora y dos enlaces: **confirmar** y **cancelar**. Sin el motivo de la consulta, sin datos clínicos, sin adjuntos.
3. **El clic escribe en la agenda.** Cada enlace es un webhook: actualiza el estado (`Confirmada` o `Cancelada`), avisa al equipo y devuelve una página legible en el móvil. Si el enlace está roto, el paciente ve un mensaje claro y no un error técnico.
4. **La cancelación dice qué hueco queda libre.** El aviso al equipo no dice «ha habido una cancelación»: dice **quién** y **a qué hora**, lo único que permite ofrecer el hueco esa misma tarde.
5. **Un resumen que nombra a quien no se puede avisar.** A las 18:05, la agenda de mañana por hora, con tratamiento, estado y **`SIN EMAIL`** en quien no recibió recordatorio. Esa marca es la lista de llamadas de la tarde.

## Cómo es el flujo de verdad

El flujo publicado (`recordatorio-citas.json`) tiene **21 nodos** y **tres partes independientes** en el mismo archivo, para que un fallo en una no tumbe las otras:

- **A) Nueva cita** — formulario → normalizar → guardar en la hoja → aviso al equipo por Telegram.
- **B) Recordatorios** — cada día a las **18:00** → lee las citas → un correo por paciente con los dos enlaces → resumen al equipo. Si no hay citas mañana, **no envía nada**.
- **C) Enlaces del paciente** — webhook → valida ID y acción → actualiza el estado → avisa al equipo → página de respuesta (200 si es válido, 400 si no).

La hoja tiene **11 columnas** con estos nombres exactos: `ID | Paciente | Telefono | Email | Tratamiento | Fecha | Hora | Estado | Notas | Alta | Actualizado`. Los estados son `Pendiente`, `Confirmada`, `Cancelada` y `Atendida` (esta a mano al terminar la consulta). El email va con **continuar en caso de fallo**: si un correo rebota, el resumen al equipo sale igual y el error queda en las ejecuciones de n8n.

## Montarlo: 30-40 minutos y una URL pública

Tres credenciales (Google Sheets, Telegram del equipo y SMTP), la hoja con las cabeceras exactas y **una URL pública de n8n**: los enlaces van al móvil del paciente, así que un `localhost` no sirve.

La prueba que vale son cinco minutos: da de alta una cita tuya para mañana, espera el correo, pulsa **cancelar** y mira que la hoja cambie de estado y que el aviso llegue al equipo con la hora.

## Cuatro errores que lo hunden

- **Mandar el motivo de la consulta por correo.** Un email que dice «limpieza» o «revisión de implante» es un dato de salud (art. 9 RGPD) viajando por un canal que no controlas. Fecha y hora bastan para confirmar una cita.
- **Enlaces eternos.** El enlace no caduca: quien tenga el ID puede cancelar. Por eso el ID debe ser largo, único y no publicado; para caducidad, una columna `Enlace_valido_hasta` y una comprobación al leer el enlace.
- **Avisar del hueco a quien no puede rellenarlo.** El aviso tiene que ir al chat donde se decide la lista de espera, no al correo del gerente que lo lee a las 20:00.
- **Confiar solo en el email.** Los `SIN EMAIL` de la agenda de mañana son llamadas pendientes; sin esa marca, el equipo cree que todos están avisados.

## Lo que este flujo no hace

No **rellena** el hueco solo: avisa para que una persona lo ofrezca — automatizar el ofrecimiento a la lista de espera es otro tramo, y con datos de salud de por medio conviene decidirlo a mano. No es historia clínica, no factura, no cobra la señal de la reserva y no sustituye la llamada del paciente urgente. Y no verifica que quien pulsa el enlace sea el paciente: el enlace es la llave, nada más.

## Lo que hay que decir del RGPD

Los datos son de pacientes: nombre, teléfono, email y estado de la cita. Base habitual, gestión de la relación asistencial y administrativa. El flujo corre en **tu** n8n con **tus** credenciales, los correos solo llevan fecha y hora, el chat de Telegram debe ser privado del equipo y la hoja necesita una retención definida.

## Si prefieres el atajo

El flujo completo —**21 nodos, tres partes, guía con las 11 columnas, pruebas y problemas frecuentes**— está abierto en el repo: **[recordatorio-citas.json](https://github.com/mrnpvn/flowkit/blob/main/workflows/clinica/recordatorio-citas.json)** y su **[guía paso a paso](https://github.com/mrnpvn/flowkit/blob/main/docs/FLUJO-CLINICA.md)**. Gratis, licencia MIT, se importa en dos minutos.

Si además la web no convierte, la **[plantilla web para clínicas dentales](https://mrnpvn.gumroad.com/l/dental-website-template)** resuelve la otra mitad (secciones, FAQ, cita como llamada a la acción), y el **[pack de prompts para clínicas dentales](https://mrnpvn.gumroad.com/l/ai-prompts-dentales)** cubre los textos que se repiten cada semana. Para empezar por lo pequeño, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)**, y en el **[catálogo de flujos gratuitos](https://mrnpvn.github.io/flowkit/)** están los demás.

Si vienes de cero con n8n, empieza por **[tu primer flujo en 30 minutos](/ia-para-negocios/posts/n8n-principiantes-30-minutos.html)**; y si lo que falla es la agenda llena de huecos, mira **[qué secciones llenan la agenda de una web dental](/ia-para-negocios/posts/web-clinica-dental-que-llena-agenda.html)** y **[por qué la mayoría de webs de clínica no venden](/ia-para-negocios/posts/web-clinica-dental-que-vende.html)**.

## Preguntas frecuentes

**¿Esto sustituye a los SMS?** No: los complementa. El SMS tiene más tasa de lectura; el enlace con dos acciones es lo que cambia la agenda. Puedes usar SMS para el aviso y este correo para la acción.

**¿Y si el paciente no tiene email?** Aparece como `SIN EMAIL` en el resumen de las 18:05: esa es la lista de llamadas. El flujo no inventa direcciones ni manda nada a quien no la tiene.

**¿Necesito WhatsApp Business o IA de voz?** No. El flujo usa formulario, Google Sheets, SMTP y Telegram. Las alternativas con voz o WhatsApp que aparecen en el catálogo dependen de cuentas de pago y no cambian el mecanismo: lo que falta siempre es el estado, no el canal.

**¿Se puede usar en una clínica de fisioterapia o en un centro de estética?** Sí: el flujo no sabe de medicina, solo de citas. Cambia el nombre del centro y la lista de tratamientos de la hoja.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Recordatorios de cita que se confirman solos (y avisan del hueco cuando alguien cancela)",
  "description": "Un recordatorio que solo informa no cambia la agenda. El sistema mínimo para que el paciente confirme o cancele con un clic, el estado se actualice solo y el equipo sepa del hueco antes de perderlo. Con dato medido del catálogo n8n y el flujo de 21 nodos que lo hace.",
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
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/recordatorios-cita-hueco-cancelacion.html"
  },
  "about": ["recordatorios de cita", "automatización n8n", "clínicas", "cancelaciones"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Sustituye a los SMS de recordatorio?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No, los complementa. El SMS tiene más tasa de lectura; lo que cambia la agenda es que el paciente pueda confirmar o cancelar con un clic y que ese clic actualice el estado el día anterior."
      }
    },
    {
      "@type": "Question",
      "name": "¿Qué pasa si el paciente no tiene email?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Aparece marcado como SIN EMAIL en el resumen de la agenda de mañana que recibe el equipo a las 18:05. Esa marca es la lista de llamadas de la tarde; el flujo no inventa direcciones."
      }
    },
    {
      "@type": "Question",
      "name": "¿Necesito WhatsApp Business o IA de voz?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. El flujo público usa formulario, Google Sheets, SMTP y Telegram. Las plantillas con voz o WhatsApp del catálogo dependen de cuentas de pago y no cambian el mecanismo: lo que falta en la agenda es el estado de la cita, no el canal."
      }
    },
    {
      "@type": "Question",
      "name": "¿Sirve para fisioterapia o un centro de estética?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí. El flujo solo entiende de citas: fecha, hora, paciente y estado. Se cambia el nombre del centro y la lista de tratamientos de la hoja."
      }
    },
    {
      "@type": "Question",
      "name": "¿El enlace de cancelar caduca?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No de serie. El enlace es la llave y quien tenga el ID puede cancelar, por eso debe ser largo, único y no publicado. Para caducidad, una columna con la fecha límite y una comprobación al leer el enlace."
      }
    }
  ]
}
</script>
