---
title: "Leads inmobiliarios: la respuesta que sirve no es «te llamamos», es la cartera filtrada"
description: "Un acuse de recibo automático no vende una visita. El sistema mínimo para que el lead reciba en el mismo minuto tres inmuebles reales de tu cartera, el equipo sepa a quién llamar primero y ningún lead se quede 24 horas sin tocar. Con dato medido del catálogo n8n y el flujo de 16 nodos que lo hace."
---

# Leads inmobiliarios: la respuesta que sirve no es «te llamamos», es la cartera filtrada

Son las 21:40 de un domingo. Entra un formulario: piso, compra, zona centro, hasta 240.000 €. A las 21:41 el lead recibe un correo automático que dice que su solicitud se ha recibido y que un comercial se pondrá en contacto. A las 21:42 está mirando el anuncio de otra agencia.

El problema no es la velocidad: es que la respuesta correcta **no se puede escribir con una plantilla**, porque tiene que decir qué tienes tú que encaje con lo que él busca. Eso tu web no lo sabe; tu cartera sí.

## El dato: hay 94 plantillas inmobiliarias y el hueco sigue vacío

Antes de escribir nada, qué existe. Medido el **28 de octubre de 2026** recorriendo todas las páginas de la API pública del catálogo de n8n (`api.n8n.io/api/templates/search`):

| Búsqueda | Plantillas | Vistas (mediana) | Con 0 vistas | La más vista |
|---|---|---|---|---|
| `real estate` | 94 | 0 | **67 de 94** | 7.538 — seguimiento de mercado con scraping e IA |
| `real estate lead` | 40 | 0 | 25 de 40 | 6.215 — generación de leads con datos de terceros |
| `property` | 51 | 0 | 40 de 51 | 4.857 — inventario enriquecido con reconocimiento de imagen |
| `lead capture` | 15 | 3 | 7 de 15 | 5.280 — multi-formularios |
| `lead scoring` | 20 | 1 | 9 de 19 | 12.106 — captación y puntuación en LinkedIn |
| `lead qualification` | 30 | 0 | 17 de 29 | 14.246 — llamada automática de cualificación |
| `inmobiliaria` | **0** | — | — | — |
| `inmuebles` | **0** | — | — | — |
| `vivienda` | **0** | — | — | — |

Tres lecturas, y las tres importan:

- **Lo que se publica es captación y datos, no respuesta con producto.** Las tres más vistas del nicho compran datos de terceros, leen el mercado o enriquecen un inventario. Ninguna resuelve el tramo que decide la visita: contestar con las tres referencias que encajan.
- **Cero en español.** `inmobiliaria`, `inmuebles` y `vivienda` no devuelven nada. El mecanismo no está, y menos en el idioma en el que buscan las agencias.
- **Publicar no es distribuir.** 67 de las 94 de `real estate` tienen cero vistas: una plantilla sin mecanismo propio no se encuentra.

## Por qué «responder rápido» no es el mecanismo

La lógica habitual dice que si respondes en cinco minutos el lead no se va. Pero un acuse en cinco minutos y otro en cinco horas contienen lo mismo: **nada**.

Lo que cambia la conversación es que el correo lleve lo que el lead busca con sus palabras, **dos o tres inmuebles reales con referencia, zona, habitaciones, superficie y precio**, y una salida si nada encaja. Es lo único que una agencia puede enviar y una plantilla genérica no — y ya lo tienes en la hoja de inmuebles.

## El mecanismo en cinco piezas

Ninguna lleva IA. Todas llevan estado.

1. **Un formulario que pregunta lo que tu cartera sabe contestar.** Operación, tipo, zona, presupuesto, habitaciones, mensaje y una casilla de privacidad. Ni un campo más: cada campo extra es una razón para abandonar.

2. **Una puntuación de 0 a 100 que ordena la llamada de mañana.** Base 30 por rellenar el formulario; +15 por email; +15 por teléfono; +10 por presupuesto, zona o mensaje de más de 20 caracteres; +10 con consentimiento. De **70** arriba, prioridad **Alta**; de 45 a 69, **Media**; por debajo, **Baja**. No mide si tiene dinero: mide si está completo, que es lo único medible antes de llamar.

3. **Un emparejamiento contra la cartera, con descartes duros antes de puntuar.** Fuera lo que no está `Disponible`, lo que no coincide con la operación o el tipo y lo que se pasa del **presupuesto +10 %**. Lo que sobrevive puntúa: zona **+30** (**+15** si comparte una palabra), tipo **+20**, habitaciones **+15** (**+8** si difiere en una), dentro de presupuesto **+25** (**+10** si está por encima). Se tira lo que no llega a 10 y se envían **como máximo tres**.

4. **Un correo que ya trae la selección puesta.** Nombre, lo que busca, las fichas de los tres inmuebles y la salida para ajustar la búsqueda. Si la cartera no tiene nada, la frase cambia: no se inventa una coincidencia ni se envía una lista vacía.

5. **Un recordatorio que nombra a quien hay que llamar.** Cada día a las 10:00, los leads `Nuevo` con **más de 24 horas**: ID, nombre, zona, prioridad, comercial, teléfono y horas transcurridas, del más antiguo al más reciente. No dice «tienes leads pendientes»: dice a quién llamar. Al marcar `Contactado`, desaparece.

## Cómo es el flujo de verdad

El flujo publicado (`captacion-leads-respuesta.json`) tiene **16 nodos** —13 funcionales y 3 notas— y **dos partes independientes** en el mismo archivo:

- **A) Nuevo lead** — formulario → normalizar → leer la cartera → clasificar y emparejar → preparar fila → guardar en `Leads` → email al lead (solo con email **y** consentimiento) → aviso al equipo por Telegram.
- **B) Leads sin atender** — cada día a las 10:00 → leer `Leads` → filtrar `Nuevo` con más de 24 h → un solo mensaje al equipo. Si no hay ninguno, **no se envía nada**.

Dos pestañas: `Inmuebles` con **11 columnas** (`ID | Titulo | Operacion | Tipo | Zona | Precio | Habitaciones | Superficie | Estado | Enlace | Notas`) y `Leads` con **17** (`ID | Fecha | Nombre | Email | Telefono | Operacion | Tipo | Zona | Presupuesto | Habitaciones | Mensaje | Consentimiento | Puntuacion | Prioridad | Comercial | Inmuebles | Estado`). Los precios se interpretan con separadores (`280000`, `280.000`, `280.000 €`), porque nadie rellena una hoja igual que otra. La asignación por zona es una tabla de cinco claves al principio del código (`centro`, `norte`, `sur`, `este`, `oeste`): se cambia por los nombres de tu oficina y listo.

## Montarlo: una hora y tres credenciales

Google Sheets, Telegram y SMTP — ninguna de pago; con Gmail, una contraseña de aplicación resuelve el correo.

La prueba vale cinco minutos y necesita dos casos: **un lead que encaja** (dos inmuebles disponibles en su zona y presupuesto) y **uno que no** (400.000 € en una zona sin cartera). El primero recibe dos o tres fichas con referencia; el segundo, la frase de «un comercial revisará tu solicitud». Y el aviso interno tiene que decir prioridad y comercial, no «nuevo lead».

## Cuatro errores que lo hunden

- **Cartera que no se actualiza.** El flujo descarta lo que no está `Disponible`, pero si nadie cambia el estado al vender, propondrá inmuebles vendidos con total seguridad y sin error. Es la pieza más frágil del montaje.
- **Emparejar por texto libre.** «Piso en el centro» y «Centro» se cruzan; «zona centro» y «Centro histórico» necesitan compartir una palabra de más de dos letras. Si tu cartera escribe las zonas de diez maneras, la solución no es más código: es una lista cerrada.
- **Enviar el correo sin consentimiento.** El flujo exige la casilla marcada y, si falta, **no envía**: avisa al equipo y guarda `Consentimiento = No`.
- **Creer que la puntuación mide solvencia.** Un lead de 100 puntos puede no poder comprar: la puntuación ordena las llamadas, la solvencia la decide una persona.

## Lo que este flujo no hace

No **valora** inmuebles: quien quiere vender recibe acuse y promesa de valoración, pero ninguna cifra. No firma, no reserva, no cobra señal ni toca tu CRM: escribe en una hoja. No comprueba que el email exista, no hace seguimiento más allá del recordatorio de 24 h y no decide el precio de tu cartera: lo lee tal como está.

## Lo que hay que decir del RGPD

Nombre, teléfono, email, zona y presupuesto son datos personales, y el mensaje libre puede llevar más de lo que parece. Base: consentimiento explícito en el formulario, con la finalidad escrita al lado de la casilla. El flujo corre en **tu** n8n con **tus** credenciales, avisa a un chat privado del equipo, el correo solo habla de su solicitud y la hoja necesita retención definida y forma de borrar filas ante una supresión.

## Si prefieres el atajo

El flujo completo —**16 nodos, las dos pestañas con sus cabeceras, la tabla de puntuación, la guía de montaje y los problemas frecuentes**— está en el repo: **[captacion-leads-respuesta.json](https://github.com/mrnpvn/flowkit/blob/main/workflows/inmobiliaria/captacion-leads-respuesta.json)** y su **[guía paso a paso](https://github.com/mrnpvn/flowkit/blob/main/docs/FLUJO-INMOBILIARIA.md)**. Gratis, licencia MIT, se importa en dos minutos.

Si la web no recoge el formulario, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)**; para el sistema completo está el **[Kit Completo de Automatización](https://mrnpvn.gumroad.com/l/kit-completo-autonomo)**, y en el **[catálogo de flujos gratuitos](https://mrnpvn.github.io/flowkit/)** están los demás.

Si vienes de cero con n8n, empieza por **[tu primer flujo en 30 minutos](/ia-para-negocios/posts/n8n-principiantes-30-minutos.html)**. Si falla el papeleo tras la visita, **[cómo pedir documentación sin perseguir al cliente](/ia-para-negocios/posts/documentos-de-clientes-sin-perseguir.html)**; y si no sabes qué pasa en el negocio, **[los cinco números del lunes](/ia-para-negocios/posts/cinco-numeros-despacho-lunes.html)**.

## Preguntas frecuentes

**¿No basta con contestar rápido?** No. Rapidez sin contenido solo acorta el tiempo hasta «gracias, ya os llamo yo». Lo que reduce el abandono es que el correo traiga inmuebles concretos con referencia, zona, habitaciones y precio.

**¿Necesito un CRM o una IA?** No. Google Sheets, Telegram y SMTP. El emparejamiento es aritmética y comparación de texto; una IA no mejora nada aquí y añade una cuenta de pago y una caja negra.

**¿Qué pasa si la cartera no tiene nada que encaje?** El correo cambia de frase y el aviso dice «sin coincidencias». No se envía una lista vacía ni se propone un inmueble fuera de presupuesto para rellenar.

**¿Por qué solo tres y no todos?** Porque doce opciones no se leen y tres se comparan de un vistazo; el resto queda en el aviso interno para la llamada.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Leads inmobiliarios: la respuesta que sirve no es «te llamamos», es la cartera filtrada",
  "description": "Un acuse de recibo automático no vende una visita. El sistema mínimo para que el lead reciba en el mismo minuto tres inmuebles reales de tu cartera, el equipo sepa a quién llamar primero y ningún lead se quede 24 horas sin tocar.",
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
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/leads-inmobiliarios-respuesta-con-cartera.html"
  },
  "about": ["leads inmobiliarios", "automatización n8n", "inmobiliarias", "captación de leads"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿No basta con contestar rápido a un lead inmobiliario?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. Un acuse de recibo rápido y uno lento contienen la misma información: ninguna. Lo que reduce el abandono es que el correo traiga dos o tres inmuebles concretos de la cartera, con referencia, zona, habitaciones, superficie y precio."
      }
    },
    {
      "@type": "Question",
      "name": "¿Hace falta un CRM o una IA para responder con la cartera?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. El flujo usa Google Sheets, Telegram y SMTP. El emparejamiento se resuelve con descartes por operación, tipo y presupuesto más un 10 %, y una puntuación por zona, tipo, habitaciones y precio. Una IA no mejora esa decisión y añade una cuenta de pago."
      }
    },
    {
      "@type": "Question",
      "name": "¿Qué envía si ningún inmueble encaja?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Un correo distinto, que avisa de que un comercial revisará la solicitud para preparar una selección. No se inventa una coincidencia ni se propone un inmueble fuera de presupuesto solo para rellenar el mensaje."
      }
    },
    {
      "@type": "Question",
      "name": "¿Por qué solo tres inmuebles en el correo?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Porque tres opciones se comparan de un vistazo y doce no se leen. El resto de coincidencias queda en el aviso interno de Telegram para que el comercial decida en la llamada."
      }
    },
    {
      "@type": "Question",
      "name": "¿Sirve también para alquiler?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí. El campo Operacion acepta Venta o Alquiler y el filtro es el mismo. Lo importante es que la cartera tenga ese campo bien relleno, porque proponer un alquiler a quien busca comprar es peor que no responder."
      }
    }
  ]
}
</script>
