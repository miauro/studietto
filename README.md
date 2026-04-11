# Studietto Scanetto

App local para seguir el estudio de los 40 temas de la oposición a bombero del Ayuntamiento de Madrid, con repetición espaciada automática, seguimiento de simulacros y dashboard de diagnóstico.

## Cómo usarla

1. Abre `index.html` en cualquier navegador moderno (Chrome, Firefox, Edge, Safari).
2. No necesita servidor, instalación ni conexión a internet.
3. Los datos se guardan en `localStorage` del navegador.

## Vistas (5 pestañas)

| Pestaña | Qué muestra |
|---------|------------|
| **Hoy** | Repasos pendientes (vencidos + hoy) y los de los próximos 3 días. Botón flotante `+` para registrar cualquier sesión. |
| **Temario** | Tabla de los 40 temas con columnas de nota base, nota actual, delta, próximo repaso y estado. Ordenable y filtrable. Click en un tema para ver historial completo. |
| **Simulacros** | Registro de simulacros semanales con aciertos, errores, blancos, nota neta (aciertos - errores/3), desglose por tema y gráfica de evolución. |
| **Diagnóstico** | Dashboard de prioridades: todos los temas ordenados por nota base (peor primero), con barras visuales comparando base vs actual, delta de progreso y resumen de movimiento (cuántos mejoraron, cuántos rojo→aprobado). |
| **Estadísticas** | Conteo por estado, nota media global, gráfica de evolución temporal, racha de días, horas totales y sesiones totales. |

## Datos base (baseline)

Cada tema tiene un campo `baselineScore` separado del historial de sesiones de estudio. Esto permite:
- Ver el delta (progreso real) en Temario y Diagnóstico.
- No contaminar la lógica de repetición espaciada con datos iniciales.
- Importar puntuaciones de un test diagnóstico sin afectar la programación.

Para importar: botón **Diag. Inicial** → pegar JSON con formato:
```json
[
  {"id": "C1", "score": 7.5},
  {"id": "E14", "score": 2.3}
]
```
IDs: `C1`–`C8` para común, `E1`–`E32` para específico.

## Simulacros

Cada simulacro registra:
- Fecha, aciertos, total de preguntas, errores, blancos
- Nota neta calculada automáticamente: `aciertos - errores/3`
- Tiempo (opcional)
- Desglose por tema (texto libre, ej: "T14: 2/5, T19: 1/3")
- Notas generales

La gráfica muestra la evolución de nota neta a lo largo de los simulacros.

## Lógica de repetición espaciada

### Intervalos estándar
Tras cada sesión con nota ≥ 7, se avanza al siguiente intervalo:
```
Día +1 → +3 → +7 → +14 → +21 → +30
```

### Intervalos para temas que empezaron en rojo
Si la primera puntuación fue < 6 y ahora saca ≥ 7:
```
Día +1 → +2 → +4 → +7 → +14 → +21 → +30
```

### Reglas
- **Nota ≥ 7**: avanza al siguiente intervalo.
- **Nota < 7**: reinicia a intervalo 0 (empieza de nuevo).
- La primera sesión siempre programa repaso a +1 día.

### Estados por color
| Estado | Criterio (última nota) |
|--------|----------------------|
| Rojo | < 6 |
| Amarillo | 6 – 7.99 |
| Verde | ≥ 8 |
| Sin empezar | Sin sesiones |

## Backup

- **Export**: descarga un `.json` con todos los datos.
- **Import**: carga un `.json` exportado previamente (sobreescribe datos actuales).

## Diseño

- Inspirado en Nothing / ChainGPT Labs: tipografía monoespaciada, bordes rectos, grid limpio, acento naranja.
- Modo claro por defecto, con toggle a modo oscuro.
- Mobile-responsive (funciona en iPad).
- Sin dependencias externas (salvo Google Fonts para Space Mono + Inter).

## Supuestos

- Un solo archivo HTML con CSS y JS inline.
- `localStorage` (~5-10 MB) es suficiente para el volumen de datos esperado.
- La nota neta de simulacros usa la fórmula estándar de oposiciones: `aciertos - errores/3`.
- Las sesiones se pueden eliminar individualmente desde el detalle de cada tema.
