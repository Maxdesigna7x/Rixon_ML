# Sei (Chen, Wong y Troyanskaya, 2022)

Artículo: [10.1038/s41588-022-01102-2](https://doi.org/10.1038/s41588-022-01102-2). Código y modelos: [FunctionLab/sei-framework](https://github.com/FunctionLab/sei-framework).

## Problema

Sei conserva la idea “ADN -> actividad regulatoria”, pero a una escala de etiquetas muy superior: en vez de 919 perfiles, predice **21,907** perfiles de TF, histonas y accesibilidad de más de 1,300 líneas celulares y tejidos. Su objetivo adicional es convertir ese espacio enorme de perfiles en un vocabulario interpretable de actividades regulatorias.

## Entrada y salida

Entrada: 4 kb de ADN humano GRCh38 en one-hot. Salida primaria: 21,907 probabilidades de picos en el bp central. Las etiquetas son binarias al entrenar, aunque la salida continua representa probabilidades.

Después, Sei ejecuta el modelo sobre millones de ventanas del genoma y agrupa los perfiles predichos para descubrir **40 clases de secuencia**. Estas clases resumen programas como promotor, enhancer específico de tejido o actividad CTCF-cohesina. Esta segunda salida no es una etiqueta experimental directa: es una representación derivada, aprendida de los patrones de muchas predicciones.

## Arquitectura

```text
4 kb ADN -> CNN con rutas lineal/no lineal -> convoluciones residuales dilatadas
       -> proyección con bases B-spline -> capas densas -> 21,907 probabilidades
```

- La primera CNN aprende detectores de motivos y combinaciones locales.
- Las convoluciones **dilatadas** expanden el campo receptivo sin aplicar una LSTM sobre todos los nucleótidos.
- Los bloques residuales estabilizan el entrenamiento profundo.
- La proyección B-spline comprime la dimensión espacial: conserva una representación de dónde ocurren los patrones sin aplanar todos los 256 bins directamente.

## Resultados

El artículo informa AUROC medio 0.972 y AUPRC medio 0.409 a través de los 21,907 perfiles. Frente a DeepSEA Beluga en las 2,002 etiquetas compartidas, comunica una mejora media de 19% usando la transformación `AUROC / (1 - AUROC)`.

No es una comparación uno-a-uno con NCNet: cambia genoma de referencia, longitud de entrada, número y procedencia de etiquetas y protocolo. Lo importante es el salto de escala y la capa de interpretación de “clases”, no sólo la métrica.

## Qué aporta

Sei convierte una gran colección de ensayos heterogéneos en un mapa global de actividad regulatoria y permite describir una variante como “probablemente altera un enhancer neuronal”, no sólo como “cambia el perfil #13,842”. Sigue siendo un predictor local de 4 kb: las relaciones a muy larga distancia entre enhancer y promotor quedan mejor cubiertas por Enformer/Borzoi.
