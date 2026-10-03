# Panel Mythos v9 — plan maestro

Este plan sale de tres investigaciones (sobre unas 100 fuentes): dashboards profesionales, el oficio visual y los paneles de vendedores de Mercado Libre y DTC. También de la auditoría de la v8.1.
Este archivo no lleva cifras del negocio. El panel lee los datos en vivo desde `/api/command-center`, que pasa por el Worker `mythos-panel` (Cloudflare Access) y llega a n8n.

## Qué estaba mal en la v8.1

| Pantalla | Falla | Cómo lo resuelve la v9 |
|---|---|---|
| Hoy | «Tu parte ▼35%» en rojo el día 3 del mes | Antes del día 7 no se proyecta. El mismo tramo del mes anterior se muestra en gris, con la cantidad de órdenes |
| Hoy | La ficha «Puesto como vendedor» quedaba sola en la grilla | La grilla es de 3 KPIs fijos y no quedan huérfanas |
| Hoy | Agenda: todas las fechas decían «en 2 días» | Los días se calculan con fecha calendario argentina |
| Ventas | −0,1% sin explicar por qué | Árbol facturación = órdenes × ticket, y por formato visitas × unidades por visita contra su ritmo de 30 días |
| Pauta | 4 campañas × 4 períodos en tarjetas de texto | Una tabla compacta con un punto de color contra el objetivo propio de cada campaña |
| Competidores | Todas las marcas aparecían como «nueva» | Se eliminó la etiqueta. Sólo se muestran cambios de precio de 5% o más, por litro y de los últimos 3 días |
| Categoría | Búsquedas de aceite de coco y de girasol | Filtro: sólo búsquedas de aceite de oliva, clasificadas en marca rival, formato o genérica |
| Plan | 13 vencidas y 41 predicciones acumuladas | La bandeja tiene un tope de 5, prioridad calculada y deduplicación. El resto va a Plan, ordenado |

## Estructura: de 11 pantallas a 6

1. **Hoy.** Responde en 10 segundos: ¿voy bien?, ¿qué se rompió?, ¿qué hago? Lleva una frase-mensaje, 3 KPIs (7 días, hoy contra un día típico a la misma hora, tu parte del mes), la bandeja de acciones con un máximo de 5 y «Lo que controlás» (stock en Full, cuenta, TACOS, preguntas y recompra).
2. **Ventas.** El porqué: árbol de métricas, tabla por formato, facturación diaria de 60 días con media de 7 días y el día en curso rayado, y cuándo compran (por hora y por día).
3. **Pauta.** TACOS de 7, 14, 30 y 90 días; tabla de campañas con el ROAS de cada ventana contra su objetivo; TACOS por formato.
4. **Mercado.** Competidores y categoría juntos: precio por litro, visitas de la semana, cambios de precio importantes y qué busca la gente.
5. **Plata.** Meses cerrados y el mes en curso (estimado), y lo que queda por unidad (precio, comisión, envío de Full, impuestos y neto).
6. **Plan.** Fechas comerciales con cuenta regresiva, todas las acciones con su corte («cómo saber si funcionó») y los límites del panel.

## Reglas duras (las aplica el código)

1. Ningún número va sin base explícita: «vs. 7 días anteriores», «vs. sábado típico a esta hora», «vs. mismo tramo del mes anterior».
2. **El color se gana.** El rojo o el verde sólo aparecen si la semana sale de los límites naturales: media ± 2,66 × rango móvil de las últimas 7 semanas (XmR de Wheeler). Si no, gris y la frase «dentro de lo normal». Con menos de 5 unidades, la variación va en gris.
3. Signo y flecha siempre (▲ ▼). El rojo, el verde y el oro de la marca se confunden con deuteranopía.
4. Sin proyección del mes antes del día 7, y desde entonces como rango (±10%), nunca como número exacto.
5. **Bandeja con tope de 5.** Peso = impacto × urgencia. Hoy hay 6 reglas:
   1. una promo que define el precio y vence en 2 días o menos;
   2. la lectura del día;
   3. la reposición a Full;
   4. un formato frenado (visitas −40% o más, pauta casi sin gasto y stock para más de 90 días);
   5. campañas en rojo o debajo del objetivo en las 4 ventanas;
   6. las acciones de n8n, deduplicadas.
6. Cada acción dice qué hacer, cómo saber si funcionó y, si viene de la lectura, qué la desmentiría.
7. La frescura queda a la vista: «Leído hoy 10:35 · próximo 12:00». El punto se pone ámbar con más de 5 h y rojo con más de 12 h.
8. Si una pantalla falla, falla sola: el resto sigue funcionando.

## Sistema visual

- **Fondo** #080C09. Las superficies se elevan con más luz, no con sombra. Borde de 1 px al 8% y un grano casi invisible (opacidad 4,5%).
- **El oro (#C9A058) es foco:** uno fuerte por bloque (serie actual, KPI principal, Mythos en las comparaciones). El contexto va en crema translúcida.
- **Tipografías:** Source Serif 4 en títulos y texto; Space Mono en todas las cifras (tabulares). Georgia tiene números de altura variable y desalinea los KPIs.
- **Movimiento:** easing `cubic-bezier(.23,1,.32,1)`, 120 a 320 ms, gráficos que se dibujan en 700 ms, View Transitions entre pantallas y `prefers-reduced-motion` respetado.
- **Gráficos:** SVG propio, sin librerías (0 KB). El ancho del dibujo es el ancho real de la pantalla, así el texto del eje queda en 10 px en teléfono y en Mac.

## Lo que falta para el máximo (necesita datos nuevos de n8n)

| Mejora | Dato que falta | Dónde se agrega |
|---|---|---|
| Visitas y conversión vs. semana anterior (no sólo vs. ritmo de 30 d) | `vis7_prev` por formato | Snapshot cada 4 h |
| Anotaciones en el gráfico (cambio de precio, pausa de pauta, quiebre) | Lista de eventos con fecha | Snapshot |
| Predicciones con aciertos (puntaje Brier) | Tabla `predicciones` con resolución | Data table nueva |
| WhatsApp: «para contactar hoy» y cohortes de recompra | CRM de WhatsApp leído por n8n | Lectura del CRM en el snapshot |
| Distancia al umbral de reputación en ventas | `seller_reputation.metrics` completo | Snapshot |
| Antigüedad del stock en Full (recargo a los 120 d) | Fecha de ingreso por lote | Informe Full mensual |
| Margen después del costo del aceite y la pauta | Costo por unidad por formato | Tabla «Precios Lista» |
| Bandeja sincronizada entre teléfono y Mac | Estado de acciones en n8n | Webhook «marcar hecha» |

## Cómo se publica

`panel/index.html` es un único archivo, sin dependencias, y reemplaza al `index.html` de los assets del Worker `mythos-panel`. Usa las mismas dos rutas que la v8.1: `GET /api/command-center` y `POST /api/actualizar`. No hace falta tocar el Worker ni n8n.
