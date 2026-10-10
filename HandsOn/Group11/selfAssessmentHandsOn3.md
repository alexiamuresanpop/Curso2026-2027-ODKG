# Hands-on assignment 3 – Self assessment

## Checklist

**The cleaned CSV file:**

- [X] Uses the .csv extension and UTF-8 encoding
- [X] Uses a single standard format for dates (ISO 8601, `yyyy-mm-dd`)
- [X] Uses a single standard format for times (`hh:mm:ss`)
- [X] Does not contain leading or trailing whitespace in text values
- [X] Uses a single label (`Desconocido`) for unknown values in categorical columns
- [X] Stores coordinates as numbers, with a dot as decimal separator
- [X] Stores the alcohol and drug test results as boolean values (`true`/`false`)
- [X] Defines a unique identifier for each person (`ID_persona`)
- [X] Defines an identifier for each vehicle (`ID_vehiculo`)
- [X] Does not contain auxiliary columns used only during the cleaning

**The OpenRefine history file:**

- [X] Is exported in JSON format
- [X] Contains all the operations applied to the original dataset (23 operations)
- [X] Allows reproducing the cleaning on the original CSV

## Comments on the self-assessment

Dataset: accidentes de tráfico de Madrid (33.322 filas, 14.388 accidentes, enero–agosto 2026). Cambios aplicados en OpenRefine (GREL):

**1. Fechas y horas.** `fecha` de `dd/mm/aaaa` a `aaaa-mm-dd`; `hora` con cero inicial (`3:00:00` → `03:00:00`).

```grel
if(isBlank(value), value, value.split('/').get(2) + '-' + value.split('/').get(1) + '-' + value.split('/').get(0))
```
```grel
if(isBlank(value), value, if(value.length() == 7, '0' + value, value))
```

**2. Espacios.** `trim` en `distrito` y `tipo_accidente`.

```grel
value.trim()
```

**3. Vacíos → `Desconocido`.** En `tipo_accidente`, `tipo_vehiculo`, `cod_lesividad` y `lesividad`:

```grel
if(isBlank(value), "Desconocido", value)
```

En `estado_meteorológico`, `se desconoce` → `Se desconoce` (edición en masa) → `Desconocido`:

```grel
if(value.toString().trim() == "Se desconoce", "Desconocido", value)
```

**4. Booleanos.** `positiva_alcohol` (`S`/`N`) y `positiva_droga` (`1`); vacíos a `Desconocido`.

```grel
if(isBlank(value), "Desconocido", if(value.toString().trim().toUppercase() == "S", true, if(value.toString().trim().toUppercase() == "N", false, value)))
```
```grel
if(isBlank(value), "Desconocido", if(value.toString().trim() == "1", true, value))
```

**5. Coordenadas UTM.** Coma decimal a punto, conversión a número, y vacíos/`0` a `null` (13 registros sin coordenadas). En `coordenada_x_utm` y `coordenada_y_utm`:

```grel
if(isBlank(value), null, toNumber(replace(value.toString(), ",", ".")))
```
```grel
if(isBlank(value), null, if(value == 0, null, value))
```

**6. Identificadores** (modo *record-based*). Columna auxiliar `accidente_grupo` (copia de `num_expediente` + *blank down*, movida al inicio) para agrupar filas por accidente; eliminada al final.

`ID_persona` (basada en `num_expediente`):

```grel
value + "-P" + (row.index - row.record.fromRowIndex + 1).toString()
```

`ID_vehiculo` (basada en `tipo_persona`; un contador por cada conductor del accidente):

```grel
with(
  filter(
    slice(row.record.cells["tipo_persona"].value, 0, row.index - row.record.fromRowIndex + 1),
    v, v == "Conductor"
  ).length(),
  n,
  if(n == 0, "", cells["num_expediente"].value + "-V" + n.toString())
)
```

**7. Exportación.** CSV limpio y `history.json` con las 23 operaciones.

**Limitaciones:** las coordenadas siguen en UTM (no se convierten a latitud/longitud); los peatones heredan el `ID_vehiculo` del conductor anterior; `positiva_droga` solo distingue `true` y `Desconocido`.
