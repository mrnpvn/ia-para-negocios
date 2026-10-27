---
title: "Los cinco números que un despacho debe mirar el lunes (sin abrir cuatro programas)"
description: "De los 27 KPIs de un despacho, solo cinco cambian una decisión: porcentaje vencido, mayor deudor, bloqueados +21 días, documentación recibida y días medios de cobro. Con umbrales, dato medido del catálogo n8n y el panel que los calcula solo cada lunes."
---

# Los cinco números que un despacho debe mirar el lunes (sin abrir cuatro programas)

Son las 08:00 de un lunes. El responsable de un despacho de nueve personas abre el programa de facturación para ver quién debe, la hoja de expedientes para ver qué está parado, el correo para ver qué papeles faltan y el banco para ver si entró algo. Veinte minutos y una conclusión que no es un dato: «vamos más o menos».

El problema no es que falten números, es que están repartidos y ninguno tiene un umbral que diga cuándo actuar. **Un número sin umbral es decoración.**

## Por qué el resumen en Excel no resuelve esto

Casi todos los despachos ya tienen una hoja de facturas y otra de expedientes. El fallo no es la hoja: es que nadie la convierte en una lectura semanal.

- **Una fila por cliente no es una serie.** Sabes que el cliente A debe 1.200 €, pero no si eso es peor que hace un mes.
- **El resumen se hace a mano y por tanto se deja de hacer.**
- **El atasco que cruza las tres áreas no lo ve nadie:** un expediente bloqueado porque falta un papel de un cliente que además debe dinero.
- **No hay histórico**, así que no se distingue una tendencia de un mal martes.

## El dato: hay plantillas de «KPI», pero ninguna es el panel de un despacho

Antes de montar nada conviene saber qué existe. Medido el **27 de octubre de 2026** recorriendo todas las páginas de resultados de la API pública del catálogo de n8n (`api.n8n.io/api/templates/search`):
| Búsqueda | Plantillas | Vistas (mediana) | Con 0 vistas | La más vista |
|---|---|---|---|---|
| `kpi` | 19 | 0 | 16 de 19 | 61 — un panel HTML estático desde Google Sheets |
| `dashboard` | 31 | 0 | 19 de 31 | 8.839 |
| `weekly report` | 97 | 0 | 74 de 97 | 6.165 |
| `report` | 574 | 0 | 388 de 574 | 36.976 |
| `indicadores` | 13 | **552** | 2 de 13 | 956 — un indicador técnico de Binance |
| `cuadro de mando` | 25 | 0 | 17 de 25 | 13.185 — y el buscador devuelve recetas y sorteos |

Tres lecturas:

- **Lo genérico está a cero.** 16 de las 19 plantillas de `kpi` no tienen ni una visita: publicar un panel no es distribuirlo.
- **El español sí tiene demanda… de otra cosa.** `indicadores` es la única búsqueda con mediana alta (552 vistas) y las doce primeras son de **trading**: la palabra se busca, pero no para gestionar un despacho.
- **Conclusión:** falta **el modelo de datos** —facturas, expedientes y documentos con estado y fecha— y decidir qué cinco cifras se miran.

## Los cinco números que sí cambian una decisión

Un panel de despacho puede calcular 27 KPIs. Pero un lunes por la mañana solo se decide con cinco, y estos son los que llevan umbral:

### 1. Porcentaje vencido (el termómetro de la caja)

Importe vencido entre importe pendiente: el único número que dice si el problema es de cobro o de volumen. **Verde** por debajo del 25 %, **ámbar** entre 25 % y 49 % (toca llamar esta semana) y **rojo** desde el 50 %, cuando ya hay facturas que no se van a cobrar solas.

### 2. El mayor deudor y el tramo de más de 90 días

El total se puede mirar y no hacer nada; un nombre y una cifra, no. Y el tramo importa más que el total: **lo que lleva más de 90 días vencido rara vez se cobra con un recordatorio**, se cobra con una llamada o se provisiona.

### 3. Expedientes bloqueados más de 21 días

Abiertos con más de 21 días desde el alta. No es productividad: es **la lista de llamadas de la semana**. Si sube dos semanas seguidas, el cuello está en la documentación, no en el equipo.

### 4. Porcentaje de documentación recibida y el servicio más lento

Qué parte de los documentos pedidos ha llegado y qué servicio acumula más antigüedad. Un despacho puede tener el 80 % global y un servicio al 40 %: el promedio esconde el caso que hay que arreglar.

### 5. Días medios de cobro

Días entre emisión y pago, sobre las facturas pagadas en los últimos 90 días. Se compara con la propia serie: si pasa de 45 a 60 días en un trimestre, el problema es de proceso y no de un cliente.

Encima de los cinco, un semáforo: **ROJO** si el porcentaje vencido es rojo, **ÁMBAR** si hay cobros en ámbar o algún expediente bloqueado, **VERDE** el resto. Sirve para saber en tres segundos si el lunes toca reunión.

## Cómo se calculan solos (20 nodos, lunes a las 08:00)

El panel de referencia hace el trabajo en cuatro pasos y devuelve **27 KPIs**, de los que los cinco de arriba son la parte que se decide:

- **Calcula.** Cada lunes a las 08:00 lee las hojas `Facturas`, `Onboarding` y `Docs` y calcula cobros (pendiente, vencido, cuatro tramos, cobrado 30 días, mayor deudor), expedientes (abiertos, tasa, bloqueados +21 días, servicio más lento) y actividad del mes.
- **Escribe el panel.** Limpia `KPIs` y deja una fila por indicador con su unidad. Si una hoja está vacía, el panel sale a cero: no falla.
- **Añade histórico.** Una fila por semana en `KPIs_Historico`: es lo que convierte 27 cifras en una tendencia; con menos de cuatro filas no hay gráfico, hay ruido.
- **Avisa con una acción.** Un mensaje a Telegram con las cifras clave y **una** recomendación. Si algo falla, el Error Trigger manda el nodo y el mensaje exactos en vez de quedarse en silencio.

Tolera fechas en `dd/mm/aaaa`, `aaaa-mm-dd` y seriales de Sheets, e importes en `1.234,56` o `1234.56`: los dos detalles que hunden cualquier panel hecho a mano.

Para que las hojas tengan datos que leer hacen falta los otros dos flujos: el de **[cobros vencidos](https://mrnpvn.github.io/flowkit/cobros-vencidos/)** y el de **[recogida de documentación](/ia-para-negocios/posts/documentos-de-clientes-sin-perseguir.html)**. Con el de **[plazos fiscales a 21 días](/ia-para-negocios/posts/plazos-fiscales-sin-perder-un-cliente.html)** comparten ficha de cliente: un dato, tres usos.

## Montarlo en diez minutos

Requisitos: n8n 1.40 o superior, un libro de Google Sheets con cinco pestañas (`Facturas`, `Onboarding`, `Docs`, `KPIs`, `KPIs_Historico`) y un bot de Telegram para el equipo.

1. Crea `KPIs` y `KPIs_Historico` **en el mismo libro** y pega las cabeceras tal cual (`Seccion | KPI | Valor | Unidad | Detalle` y `Fecha | Pendiente_eur | Vencido_eur | ...`): cambiar una mayúscula rompe la escritura.
2. Elige libro y pestaña en los seis nodos de Sheets y pon el `chatId` del equipo en los dos de Telegram.
3. Pulsa prueba manual: deben aparecer 27 filas en `KPIs` y una nueva en `KPIs_Historico`.
4. Actívalo: a partir de ahí corre solo los lunes a las 08:00.

Si es tu primer flujo, empieza por **[N8N para principiantes: tu primer flujo en 30 minutos](/ia-para-negocios/posts/n8n-principiantes-30-minutos.html)**.

## Los tres errores que convierten el panel en decoración

1. **Medir sin umbral.** Un 32 % vencido no dice nada si no está escrito que por encima del 25 % se llama. El umbral es parte del indicador, no del informe.
2. **Mandarlo por correo al final del día.** Un panel entre veinte correos no se lee; un mensaje de seis líneas en el chat del equipo, sí.
3. **Cambiar los umbrales a mitad de la serie.** El histórico guarda lo ya calculado: si mueves los tramos en la semana seis, la gráfica compara dos cosas distintas.

## RGPD en cinco líneas

- Los datos son de clientes del despacho: nombres, importes y estado de documentos. Base habitual, interés legítimo en controlar cobro y operación (art. 6.1.f).
- El flujo corre en **tu** n8n con **tus** credenciales; no pasa por servidores nuestros.
- El panel informa, no decide: no hay decisiones automatizadas sobre personas.
- El chat de Telegram con nombres y cifras debe ser privado y del equipo, y el histórico necesita una retención definida.

## Lo que este flujo no hace

No envía correos ni escribe a clientes, no emite facturas, no calcula impuestos y no dice si un cliente es rentable. Lee tus hojas, escribe tu panel y avisa a tu equipo: la decisión —a quién se llama el lunes— sigue siendo humana, que es donde debe estar.

## Si prefieres el atajo

El panel completo —flujo de 20 nodos, guía de instalación, las cinco pestañas con sus cabeceras y la rutina del lunes de 10 minutos— va dentro del **[Pack Asesoría de FlowKit (29 €)](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria)**, con el onboarding documental y los cobros vencidos: tres automatizaciones con licencia para tu despacho y tus clientes.

Si aún decides por dónde empezar, el **[kit de arranque IA es gratis](https://mrnpvn.gumroad.com/l/kit-de-arranque-ia)** y el **[catálogo de flujos gratuitos](https://mrnpvn.github.io/flowkit/)** tiene los flujos abiertos para importar hoy.

## Preguntas frecuentes

**¿Cuántos KPIs necesito?** Cinco decisiones y un semáforo. Los 27 existen para elegir los cinco que mueven tu despacho; si empiezas por el panel completo, acabas con una hoja que nadie lee.

**¿Sirve si ya uso un ERP?** Sí: exporta facturas y expedientes a la hoja o sustituye solo los nodos de lectura; el cálculo y el panel no cambian.

**¿Y si las hojas están vacías?** La primera ejecución sale a cero y en verde: sirve para comprobar credenciales y cabeceras antes de meter datos reales.

**¿Necesito n8n de pago?** No: la versión self-hosted es gratuita y el coste real es el servidor.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Los cinco números que un despacho debe mirar el lunes (sin abrir cuatro programas)",
  "description": "De los 27 KPIs de un despacho, solo cinco cambian una decisión: porcentaje vencido, mayor deudor, bloqueados +21 días, documentación recibida y días medios de cobro. Con umbrales, dato medido del catálogo n8n y el panel que los calcula solo cada lunes.",
  "inLanguage": "es",
  "datePublished": "2026-10-27",
  "dateModified": "2026-10-27",
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
    "@id": "https://mrnpvn.github.io/ia-para-negocios/posts/cinco-numeros-despacho-lunes.html"
  },
  "about": ["KPIs de despacho", "automatización n8n", "asesorías y gestorías", "panel de control"]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Cuántos KPIs necesita un despacho de verdad?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Cinco decisiones y un semáforo: porcentaje vencido, mayor deudor y tramo de más de 90 días, expedientes bloqueados más de 21 días, porcentaje de documentación recibida y días medios de cobro. El resto de indicadores sirve para diagnosticar cuando uno de esos cinco se mueve."
      }
    },
    {
      "@type": "Question",
      "name": "¿Sirve el panel si ya uso un ERP?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí. Exporta facturas y expedientes a la hoja o sustituye solo los nodos de lectura; el cálculo de los indicadores, el panel y el histórico no cambian."
      }
    },
    {
      "@type": "Question",
      "name": "¿Qué pasa si las hojas están vacías la primera vez?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "El panel sale a cero y la salud en verde. Es una ejecución válida y sirve para comprobar credenciales y cabeceras antes de meter datos reales."
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
