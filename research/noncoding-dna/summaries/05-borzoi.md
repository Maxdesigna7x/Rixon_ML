# Borzoi (Linder et al., 2025)

Artículo: [10.1038/s41588-024-02053-6](https://doi.org/10.1038/s41588-024-02053-6). Código/modelos: [calico/borzoi](https://github.com/calico/borzoi).

## Problema

Borzoi amplía Enformer hacia una salida más detallada: **predecir cobertura de RNA-seq, específica de tejido/célula, desde ADN**. Una cobertura RNA-seq contiene información de expresión, exones, intrones, splicing y poliadenilación. Así conecta más directamente una variante no codificante con consecuencias transcriptómicas.

## Entrada y salida

Entrada: 524,288 bp (524 kb) de ADN. Salida: cobertura predicha en bins de 32 bp a lo largo de la ventana, para miles de experimentos RNA-seq y pistas regulatorias. El modelo también se entrena con CAGE, DNase-seq, ATAC-seq y ChIP-seq para mejorar la representación regulatoria.

La salida ya no es un vector de 919 booleanos. Para una pista RNA-seq, cada bin contiene una intensidad de cobertura. Al agregar los bins de exones se obtiene una estimación de expresión; al comparar exón/intrón o uniones se puede estudiar procesamiento del RNA.

## Arquitectura

```text
524 kb ADN -> torre CNN + downsampling
          -> self-attention a resolución de 128 bp
          -> U-Net: upsampling y conexiones skip
          -> cobertura multicanal a resolución de 32 bp
```

La autoatención es el núcleo de Enformer: comunica posiciones distantes. El problema es que no es viable aplicarla a resolución de una base sobre 524 kb. Borzoi la hace operar a 128 bp y usa una U-Net para recuperar 32 bp. Las conexiones *skip* reutilizan detalles locales de la torre CNN mientras la representación profunda aporta contexto de larga distancia.

## Resultados

En secuencias humanas retenidas, el artículo reporta correlación de Pearson media 0.74 para cobertura por bins y 0.87 al agregar a nivel de gen. Para distinguir eQTLs finamente mapeados de negativos emparejados por distancia, informa AUROC medio 0.794 con ensemble, frente a 0.747 para Enformer; para la correlación con tamaños de efecto eQTL informa 0.334 frente a 0.227.

Son resultados contra Enformer en sus propios datos, no contra NCNet. La comparación útil es conceptual: Borzoi cubre más de 500 veces la ventana de NCNet y genera una señal transcripcional espacialmente densa.

## Aporte y límites

El aporte es unificar transcripción, splicing y poliadenilación en un modelo de secuencia a cobertura, con contexto suficientemente largo para genes grandes y regulación distante. Requiere recursos considerables, datos masivos y técnicas de atribución para interpretar una variante. También sigue siendo una predicción desde ADN: efectos trans, ambiente, dosis de TFs y estado celular no quedan completamente determinados por la secuencia.
