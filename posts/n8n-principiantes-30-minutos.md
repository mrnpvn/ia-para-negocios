# N8N para principiantes: automatiza tu primer flujo de trabajo en 30 minutos (guía paso a paso)

Si alguna vez pensaste "esto lo debería hacer solo" y no lo hiciste, es porque no tienes un sistema. N8N es ese sistema: una herramienta gratuita que conecta tus apps y ejecuta tareas automáticamente. Sin programar.

## Qué es N8N (en 30 segundos)

N8N es como Zapier pero gratis y de código abierto. Conectas servicios (Gmail, Google Sheets, Twitter, WhatsApp, IA) y defines reglas: "cuando llegue un email, extrae los datos, guárdalos en una hoja y envía un resumen por Telegram". Sin tocar una línea de código.

## Tu primer flujo en 30 minutos

### Paso 1: Instalar N8N (5 minutos)
```bash
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```
Abre http://localhost:5678 y tienes N8N funcionando.

### Paso 2: Crear tu primer trigger (5 minutos)
Un trigger es "cuando pase X, haz Y". Ejemplos:
- **Webhook**: Cuando alguien llene un formulario
- **Schedule**: Cada lunes a las 9:00
- **Email**: Cuando llegue un email con asunto "Lead"

### Paso 3: Conectar el primer nodo (10 minutos)
Selecciona el servicio que quieres automatizar (Gmail, Sheets, Twitter) y conecta tu cuenta. N8N te guía paso a paso.

### Paso 4: Añadir lógica (5 minutos)
Condiciones: "Si el email contiene 'presupuesto', clasifícalo como lead caliente". Esto es lo que hace que la automatización sea inteligente, no solo un zapp.

### Paso 5: Activar y probar (5 minutos)
Activa el flujo, dispara el trigger manualmente y verifica que todo funciona.

## Los 3 flujos que todo freelancer necesita

1. **Captura de leads**: Formulario web → Clasificación con IA → Email de respuesta → Registro en hoja
2. **Publicación social**: Cada 4 horas → Genera contenido con IA → Publica en redes → Log en hoja
3. **Onboarding de cliente**: Nueva venta → Email de bienvenida → Checklist → Recordatorio día 3

## El pack que te da los workflows listos

**[N8N Automation Workflow Pack](https://mrnpvn.gumroad.com/l/n8n-automation-workflow-pack)** — 2 workflows listos para importar (Social Content Factory + Social Media AI Agent) + documentación completa. No necesitas inventar la rueda.

Si quieres el sistema completo (guía + prompts + workflows), el **[Kit Completo Autónomo](https://mrnpvn.gumroad.com/l/kit-completo-autonomo)** es la mejor opción.

El catálogo completo está en **[la tienda MRNPVN](https://mrnpvn.gumroad.com)**.