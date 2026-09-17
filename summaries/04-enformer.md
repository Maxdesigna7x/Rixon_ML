# Enformer (Avsec et al., 2021)

Artículo: [10.1038/s41592-021-01252-x](https://doi.org/10.1038/s41592-021-01252-x). Código/modelos: [DeepMind Enformer](https://github.com/google-deepmind/deepmind-research/tree/master/enformer).

## El cambio de pregunta

La limitación de NCNet no es que no reconozca motivos, sino que ve sólo 1 kb. Enformer pregunta: **¿podemos predecir perfiles de cromatina y expresión usando una región suficientemente grande para incluir enhancers distales?** Es una extensión directa de secuencia-a-función, no una repetición del benchmark binario de 919 etiquetas.

## Entrada y salida

Entrada: 196,608 bp de ADN one-hot. Salida: señales cuantitativas a lo largo de 114,688 bp centrales, agrupadas en bins de 128 bp: 5,313 pistas humanas o 1,643 de ratón. Las pistas incluyen CAGE (actividad de transcripción), DNase/ATAC y ChIP-seq.

En vez de “¿hay pico sí/no en esta ventana?”, la salida es un perfil: para cada posición y ensayo, cuánto señal se espera. Esto es más cercano a la medición experimental y permite sumar o comparar regiones para puntuar variantes.

## Arquitectura

```text
196 kb ADN -> 7 bloques CNN + pooling -> 11 bloques Transformer
          -> recorte de bordes -> CNN puntuales -> cabeza humano o ratón
```

Las CNN iniciales detectan motivos y comprimen la secuencia a resolución de 128 bp. Los Transformers aplican autoatención con codificación posicional relativa: cada posición puede ponderar información de otras posiciones, por ejemplo un enhancer lejano al predecir un promotor. El modelo logra utilizar interacciones de hasta ~100 kb, frente a ~20 kb de Basenji2/ExPecto según los autores.

## Resultados y aportación

En expresión específica de tejido/célula, el artículo informa correlación de 0.85 frente a 0.81 para Basenji2; estima una fiabilidad experimental entre réplicas de 0.94. También supera a Basenji2 para efectos de variantes, ensayos de mutagénesis y priorización de enhancer–gen validado con CRISPRi.

No debe compararse su correlación 0.85 con los AUROC/auPRC de NCNet: predicen objetivos, ventanas y tipos de señal distintos. La contribución de Enformer es arquitectónica y biológica: usar atención para conectar regulación distal con el gen o pista de salida apropiados.

## Límite

La atención sigue siendo costosa y el modelo no observa el estado ambiental/trans completo de una célula. Una predicción alta es una estimación de señal desde secuencia, no una prueba de que el enhancer contacte causalmente un promotor. Aun así, representa el salto más claro desde clasificadores locales como NCNet hacia modelos de regulación a larga distancia.
