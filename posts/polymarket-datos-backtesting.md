# Datos históricos de Polymarket para backtesting: dónde conseguirlos y qué mirar antes de comprarlos

Los mercados de predicción (Polymarket) se han convertido en una fuente de datos barata sobre probabilidades en eventos reales: elecciones, resultados deportivos, cripto, eventos geopolíticos. Para **backtesting de estrategias**, los datos históricos de precios son la materia prima que falta la mayoría de las veces.

## Para qué sirven de verdad

- **Calibrar modelos de probabilidad**: comparar el precio de mercado con el resultado real (ej: cuota 82% a favor y el evento ocurre el 84% de las veces en tu muestra).
- **Probar estrategias de disparidad**: cuando tu estimación se aleja significativamente del precio de mercado.
- **Medir liquidez y comportamiento de las cuotas** cerca del cierre (donde ocurren los movimientos grandes).

## Qué tiene que tener un dataset decente

1. **Rango amplio**: varios meses, no una semana. Las estrategias necesitan ciclos completos.
2. **Granularidad**: marcas de tiempo frecuentes (precio a intervalos cortos), no solo precios de cierre del día.
3. **Cobertura de mercados**: varios eventos del mismo tipo para que la muestra no sea un caso aislado.
4. **Formato limpio**: CSV o JSON listo para pandas, con símbolo del mercado, timestamp, precio de compra y de venta.

## Los errores típicos al hacer backtesting

- **Sesgo de supervivencia**: entrenas solo con eventos resueltos "bien" para tu estrategia.
- **Ignorar la liquidez**: un precio de mercado no es ejecutable si no hay volumen. El dataset debería permitir filtrar por volumen.
- **Mira hacia delante**: usas información del futuro (el resultado) para decidir señales del pasado. Separa siempre datos de entrenamiento y de prueba en el tiempo.

## El dataset listo para usar

**[Polymarket Historical Trading Data (6 meses)](https://mrnpvn.gumroad.com/l/polymarket-historical-data)** — 6 meses de datos históricos en formato CSV, con mercados, marcas de tiempo y precios bid/ask. Pensado para cargar directo en Python y hacer pruebas sin pasar semanas scrapeando.

Si además quieres automatizar la obtención de datos o análisis con N8N, el **[N8N Automation Workflow Pack](https://mrnpvn.gumroad.com/l/n8n-automation-workflow-pack)** incluye flujos que puedes adaptar. El resto del catálogo: **[la tienda MRNPVN](https://mrnpvn.gumroad.com)**.