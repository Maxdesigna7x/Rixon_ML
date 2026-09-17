# NCNet (Zhang et al., 2019)

Fuente local: [`ncnet-2019.pdf`](../papers/ncnet-2019.pdf). DOI: [10.3389/fgene.2019.00432](https://doi.org/10.3389/fgene.2019.00432).

## La pregunta

NCNet intenta responder: **mirando sólo 1,000 letras de ADN, ¿qué huellas experimentales de regulación esperaríamos ver en el centro de esa ventana?** Es una tarea de genómica regulatoria, no de predicción de genes, enfermedades ni proteínas.

La motivación es que mucha variación asociada a enfermedad está en ADN no codificante. Una mutación puede romper o crear un motivo corto reconocido por un factor de transcripción (TF), cambiar el acceso de la cromatina y, de forma indirecta, alterar la expresión de un gen.

## Entrada, etiqueta y salida

La entrada es una secuencia humana GRCh37 de 1 kb, representada en *one-hot* como una matriz `1000 x 4`. Cada fila codifica A, C, G o T. La red devuelve 919 probabilidades sigmoid, una por ensayo/celda/señal regulatoria.

| Grupo de etiquetas | Número | Qué aproxima |
|---|---:|---|
| TFBS | 690 | Unión de un TF medida por ChIP-seq |
| DHS | 125 | Cromatina accesible medida por DNase-seq |
| Marcas de histonas | 104 | Estado epigenético asociado a promotores, enhancers o represión |

La etiqueta binaria se obtiene por solapamiento con un pico experimental. Un 0 significa “no se anotó un pico en ese ensayo”, no “el evento es biológicamente imposible”. El artículo llama a las 919 salidas “TF bindings”, pero el vector mezcla TFBS, accesibilidad y marcas de histonas.

El conjunto, heredado de DeepSEA/DanQ, contiene 4.4 M ejemplos de entrenamiento, 8,000 de validación y 455,024 de prueba. Las clases positivas son raras, de modo que accuracy no es una métrica fiable; PR-AUC es especialmente informativa.

## Arquitectura y razonamiento

DanQ, la línea base, usa:

```text
ADN one-hot -> CNN -> max-pooling -> BiLSTM -> dense -> 919 sigmoides
```

La CNN funciona como un banco aprendido de detectores de motivos: aprende patrones como `TGACTCA`, sin que haya que suministrar PWMs. La BiLSTM modela la “gramática regulatoria”: combinaciones, orientación y distancia entre motivos. La parte `bi` permite utilizar contexto a ambos lados.

NCNet modifica esa arquitectura en tres formas:

| Variante | Cambio | Intuición de ML |
|---|---|---|
| NCNet-RR | dos bloques CNN residuales antes de la BiLSTM | entrenar una CNN más profunda mediante atajos identidad |
| NCNet-bRR | ocho bloques residuales *bottleneck* antes de la BiLSTM | usar `1x1 -> conv -> 1x1` para reducir coste y permitir profundidad |
| NCNet-RbR | BiLSTM sobre ADN crudo y después CNN residual | capturar primero dependencias de toda la ventana y luego patrones locales |

Un bloque residual calcula aproximadamente `y = F(x) + x`. El atajo deja pasar información sin transformarla y hace más estable la optimización de redes profundas. En un *bottleneck*, las capas 1x1 reducen temporalmente el número de canales, por lo que la convolución central es barata.

## Entrenamiento y evaluación

Los autores usan RMSprop, lotes de 100, hasta 60 épocas y parada temprana tras cinco épocas sin mejora de validación. Para cada una de las 919 salidas calculan métricas binarias; informan promedios ponderados por prevalencia. Esto es importante: los resultados porcentuales de la tabla son **relativos a su propia reimplementación de DanQ**, no valores absolutos ni una garantía de mejora en otra implementación.

| Modelo | ROC-AUC relativo a r-DanQ | PR-AUC relativo | Tamaño | Tiempo/época |
|---|---:|---:|---:|---:|
| r-DanQ | 100% | 100% | 375.4 MB | ~4 h |
| NCNet-RR | 101.28% | 102.60% | 465.5 MB | ~19 h |
| NCNet-bRR | 103.64% | 114.05% | 69.5 MB | ~2 h |
| NCNet-RbR | 104.37% | 117.48% | 18.1 MB | ~42 h |

**Conclusión del artículo:** bRR es el mejor compromiso práctico. RbR consigue el PR-AUC relativo mayor, pero una LSTM sobre los 1,000 nucleótidos resulta muy lenta.

## Cómo interpretar una variante

Se predice la secuencia de referencia y la secuencia con el alelo alternativo. Para cada salida puede calcularse `delta_i = p_i(alternativo) - p_i(referencia)`. Un delta grande sugiere que la mutación puede alterar esa señal. Es una hipótesis funcional; no identifica automáticamente el gen afectado ni demuestra causalidad.

## Límites

- Sólo observa 1 kb: no puede modelar bien enhancers que regulan un promotor a 50–100 kb.
- Las salidas son picos de ensayos, no expresión génica ni fenotipo clínico.
- No aporta por defecto una explicación causal; se requieren atribuciones, mutagénesis *in silico* y validación experimental.
- La ganancia se comparó contra una reimplementación con Keras/Theano de 2019, por lo que conviene reproducir con software actual antes de adoptar la arquitectura.
