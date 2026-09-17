# ChromDL (Hill, Hudaiberdiev y Ovcharenko, 2023)

Artículo: [10.1093/bioinformatics/btad217](https://doi.org/10.1093/bioinformatics/btad217). Código: [chrishil1/ChromDL](https://github.com/chrishil1/ChromDL).

## Por qué es el sucesor más comparable

ChromDL utiliza exactamente el conjunto DeepSEA clásico que usó NCNet: ventanas de 1 kb alrededor de bins de 200 bp de GRCh37, con 919 etiquetas: 690 TFBS, 125 DHS y 104 marcas de histonas. Por ello es la comparación posterior más limpia para preguntar si una arquitectura diferente predice mejor el mismo vector de perfiles.

## Arquitectura

```text
1000 x 4 one-hot
 -> BiGRU (128 unidades)
 -> conv separable 1D (750 filtros, kernel 16)
 -> conv 1D (360 filtros, kernel 8) + max-pooling
 -> BiLSTM (128) + batch norm + average-pooling
 -> BiLSTM (128)
 -> flatten -> densa sigmoid de 919 salidas
```

Una GRU y una LSTM son variantes de RNN con compuertas. La GRU inicial forma una lectura contextual de bases antes de detectar motivos con CNN; las BiLSTM posteriores vuelven a combinar patrones de rango mayor. Una convolución separable reduce parámetros: en vez de aprender simultáneamente mezcla de canales y posición, separa ambas operaciones.

## Qué predice y cómo se evalúa

Devuelve las mismas 919 probabilidades que NCNet. Entrenó con Adam y lotes de 500, hasta 100 épocas o estancamiento de la pérdida de validación. Evalúa por etiqueta, una práctica esencial porque una media puede ocultar que el método sirve sólo para algunos TFs.

| Mediana en las 919 etiquetas | ChromDL | DanQ | DeepSEA |
|---|---:|---:|---:|
| auROC | 0.961 | 0.951 | 0.944 |
| auPRC | 0.402 | 0.372 | 0.344 |

ChromDL tuvo auROC mayor que DanQ en 812 de las 919 etiquetas (88.35%). Para TFBS específicamente, fue superior en 640 de 690. También reporta mejor detección de picos ChIP-seq débiles.

## Aporte y precaución

La aportación principal es una búsqueda extensa de arquitecturas híbridas y evidencia de que una recurrente temprana (BiGRU) más CNN y BiLSTM apiladas mejora el benchmark local. Es todavía un modelo de 1 kb, por lo que no resuelve interacción enhancer-promotor de larga distancia. Su mejora contra DanQ es directamente informativa para NCNet, pero no hay una tabla publicada que compare ChromDL contra NCNet-bRR bajo la misma receta exacta.
