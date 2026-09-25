# Modelo de dominio

Entidades y relaciones del producto, sin tecnología. Los nombres en código están en el
[glosario](glossary.md). No hay entidades nuevas: el fork añade configuración a una entidad de
Metabase (la tarjeta de dashboard) y un estado de página.

```
Dashboard
├── Filtro de dashboard                 tipo, valor actual; conectado a preguntas
└── Tarjeta de dashboard                posición, tamaño
    ├── Pregunta                        consulta, tipo de gráfico
    │   └── Resultado                   columnas (dimensiones, métricas) y filas
    └── Ajustes de visualización        título, colores, nombres visibles de las series, ...
        └── Selector de métrica (nuevo) activo (sí/no)
                                        métricas incluidas, en orden
                                        métrica inicial
```

Estado de página (va en la dirección del dashboard, no se guarda con él): métrica seleccionada de cada tarjeta con selector activo.

## Reglas estructurales
1. El selector pertenece a la tarjeta de dashboard, no a la pregunta: la misma pregunta puede tener selector en un dashboard y no en otro.
2. Solo puede activarse en tarjetas con gráfico cartesiano, dos o más métricas, sin desglose y sin series añadidas.
3. Las métricas incluidas son un subconjunto ordenado de las métricas de la pregunta, identificadas por la columna del resultado; dos o más, sin límite superior (los botones pasan a otra fila si no caben).
4. La métrica inicial es una de las incluidas; por defecto, la primera.
5. El nombre de cada botón es el nombre visible de la métrica en la tarjeta; no existe un nombre aparte.
6. La métrica seleccionada nace con el valor de la métrica inicial, o con el que indique la dirección de la página, cambia con cada pulsación y se refleja en la dirección; una dirección sin indicación vuelve a la inicial.
7. Si la pregunta cambia y una métrica incluida deja de existir, el selector la ignora; si ninguna existe, la tarjeta se muestra sin selector.

## Ciclo de vida resumido
```
editor activa el selector ──► guarda el dashboard ──► espectador abre el dashboard (métrica inicial)
        ──► pulsa un botón (la métrica seleccionada cambia, sin consulta nueva)
        ──► cambia un filtro (la tarjeta recarga datos y conserva la métrica seleccionada)
        ──► copia la dirección (lleva la métrica seleccionada) ──► otro espectador la abre con esa métrica
```
