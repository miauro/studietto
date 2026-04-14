# Studietto Scanetto

App local para seguir el estudio de la oposición a bombero del Ayuntamiento de Madrid (SEIS). Incluye repetición espaciada automática para los 40 temas del temario, registro de simulacros, calendario de repasos y un sistema independiente de seguimiento de los vehículos del SEIS.

## Cómo usarla

1. Abre `index.html` en cualquier navegador moderno (Chrome, Firefox, Edge, Safari).
2. No necesita servidor, instalación ni conexión a internet.
3. Los datos se guardan en `localStorage` del navegador (sin cuenta, sin nube).

---

## Vistas (6 pestañas)

| Pestaña          | Qué muestra                                                                                                                                                                             |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hoy**          | Repasos pendientes del día (vencidos + hoy), ordenados por prioridad, con acceso directo a registrar sesión. Los próximos 3 días también visibles.                                      |
| **Temario**      | Tabla de los 40 temas con nota, próximo repaso y estado. Ordenable por cualquier columna, filtrable por estado, bloque o búsqueda libre. Click → historial completo + registrar sesión. |
| **Simulacros**   | Registro de simulacros con aciertos, errores, blancos, nota neta y gráfica de evolución.                                                                                                |
| **Estadísticas** | Conteo por estado (rojo/amarillo/verde/sin empezar), nota media global, gráfica temporal, racha de días, horas totales y sesiones totales.                                              |
| **Calendario**   | Vista mensual de repasos programados y sesiones registradas. Navegación por meses, modo "solo pendientes", detalle por día y detección de días con sobrecarga (> cap).                  |
| **Vehículos**    | Seguimiento independiente de los 74 vehículos del SEIS con repetición espaciada por confianza (sin nota numérica). Ver sección completa más abajo.                                      |

---

## Lógica de repetición espaciada — Temas

### Intervalos estándar

Tras cada sesión con nota ≥ 7 se avanza al siguiente intervalo:

```
+1 → +3 → +7 → +14 → +21 → +30 días
```

### Intervalos para temas que empezaron en rojo

Si la primera puntuación fue < 6 y ahora el tema saca ≥ 7:

```
+1 → +2 → +4 → +7 → +14 → +21 → +30 días
```

### Reglas

- **Nota ≥ 7**: avanza intervalo.
- **Nota < 7**: reinicia a intervalo 0.
- La primera sesión siempre programa repaso a +1 día.

### Estados

| Estado      | Criterio (última nota) |
| ----------- | ---------------------- |
| Rojo        | < 6                    |
| Amarillo    | 6 – 7.99               |
| Verde       | ≥ 8                    |
| Sin empezar | Sin sesiones           |

---

## Pestaña Vehículos

Sistema completamente independiente del temario para aprender los 74 vehículos del SEIS de Madrid.

### Catálogo

El catálogo completo está **hardcodeado** en el código (`VEHICULOS_SEIS_CATALOG`). No se edita. Cubre 6 tipos:

| Tipo       | Color indicador |
| ---------- | --------------- |
| Fuego      | Rojo            |
| Salvamento | Verde           |
| Agua       | Azul            |
| Especiales | Morado          |
| Grúa       | Ámbar           |
| Ligero/Bus | Teal            |

### Estado por vehículo

Lo que sí se persiste (en `localStorage`, clave `vehiculos_sei_state`) es el **estado de repaso de cada vehículo**:

```json
{
  "V001": {
    "reviewSessions": [
      { "date": "2026-04-15", "confidence": "bien", "notes": "..." }
    ],
    "nextReviewDate": "2026-04-18",
    "intervalIndex": 1,
    "status": "bien"
  }
}
```

### Repetición espaciada — Vehículos

Sin nota numérica. Solo confianza declarada:

| Botón          | Efecto                                                                         |
| -------------- | ------------------------------------------------------------------------------ |
| 🟢 Lo sé bien  | `intervalIndex++`, próximo repaso = hoy + `intervals[nuevo índice]`            |
| 🟡 A medias    | `intervalIndex` sin cambios, próximo repaso = hoy + `intervals[índice actual]` |
| 🔴 No me lo sé | `intervalIndex = 0`, próximo repaso = hoy + 1 día                              |

Intervalos: `[1, 3, 7, 14, 21, 30]` días.

Estado del vehículo = confianza de la última sesión (`bien` / `medio` / `mal` / `sin_empezar`).

### Secciones de la pestaña

1. **Mini stats** (arriba): bien / medio / mal / sin empezar / pendientes hoy / racha de días.
2. **Repasos de vehículos hoy**: lista con botones de confianza grandes (tap-friendly). Cap diario configurable en Ajustes (por defecto 10). Prioridad: más atrasado primero, luego por peor estado.
3. **Catálogo completo**: tabla de los 74 vehículos agrupados por tipo, con filtros de tipo / propiedad / estado. Click en fila → panel de detalle con historial y opción de registrar repaso con notas.

### Export / Import vehículos

Los botones Export / Import de esta pestaña solo exportan/importan el **estado de repasos**, no el catálogo (que siempre está en el código).

---

## Simulacros

Cada simulacro registra:

- Fecha, aciertos, total de preguntas, errores, blancos
- Nota neta calculada automáticamente: `aciertos − errores/3`
- Tiempo (opcional)
- Desglose por tema (texto libre, ej: `T14: 2/5, T19: 1/3`)
- Notas generales

La gráfica muestra la evolución de nota neta a lo largo de los simulacros.

---

## Ajustes

| Campo                        | Por defecto | Descripción                                 |
| ---------------------------- | ----------- | ------------------------------------------- |
| Repasos máx./día (temas)     | 6           | Cap de la lista "Hoy" del temario           |
| Repasos máx./día (vehículos) | 10          | Cap de la lista de vehículos pendientes hoy |

---

## Backup

- **Export** (barra superior): descarga un `.json` con todos los datos del temario y simulacros.
- **Import** (barra superior): carga un `.json` exportado previamente (sobreescribe datos actuales).
- **Export / Import vehículos** (pestaña Vehículos): independiente, solo estado de vehículos.

---

## localStorage — claves usadas

| Clave                 | Contenido                                                       |
| --------------------- | --------------------------------------------------------------- |
| `studietto-scanetto`  | Temas (sesiones, estado, intervalos) + simulacros + tema visual |
| `studietto-settings`  | `maxRepasosDia`, `maxVehiculosDia`                              |
| `vehiculos_sei_state` | Estado de repasos de los 74 vehículos                           |

---

## Diseño

- Modo claro por defecto, toggle a modo oscuro.
- Mobile-responsive (funciona en iPad).
- Un solo archivo `index.html` — CSS y JS inline, sin dependencias externas salvo Google Fonts.
