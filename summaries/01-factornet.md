# FactorNet (Quang y Xie, 2019)

Artículo: [10.1016/j.ymeth.2019.03.020](https://doi.org/10.1016/j.ymeth.2019.03.020). Código: [uci-cbcl/FactorNet](https://github.com/uci-cbcl/FactorNet).

## Qué intenta resolver

FactorNet trata una versión más difícil y concreta de la tarea: **predecir dónde se unirá un TF en un tipo celular para el cual no tenemos ChIP-seq de ese TF**. Esto se denomina imputación *cross-cell-type*. No basta con saber que un motivo de TF existe: la misma secuencia puede ser inaccesible en una célula y activa en otra.

## Por qué no es exactamente NCNet

NCNet es *sequence-only*: la misma entrada de ADN debe producir una firma regulatoria aprendida de los contextos de entrenamiento. FactorNet añade información del contexto celular, por ejemplo señal DNase-seq, expresión génica y anotaciones. Esto hace que responda una pregunta más específica de célula, pero también significa que no puede usarse con sólo un FASTA sin esos datos auxiliares.

## Cómo lo hace

Es una arquitectura híbrida CNN-RNN derivada de DanQ:

```text
secuencia + perfiles de accesibilidad/anotaciones
  -> CNN: motivos y patrones locales
  -> RNN bidireccional: dependencias entre motivos
  -> clasificador de unión TF
```

La CNN busca el “candado” (motivo); DNase-seq indica si el candado es accesible; la RNN combina el contexto y la organización de motivos. FactorNet usó además datos de hebras directa y complemento inverso, un detalle necesario porque el ADN puede leerse en cualquiera de las dos orientaciones.

## Salida y evaluación

La salida es una puntuación/probabilidad de unión para una pareja TF–tipo celular y posición genómica. Se evaluó en el desafío ENCODE-DREAM de predicción de unión TF; el artículo reporta primer lugar en 6 de 13 pares TF/célula de la ronda final. No se compara numéricamente con NCNet en la misma partición de 919 etiquetas, por lo que no es válido afirmar que “gana a NCNet”.

## Aporte y límite principal

El aporte es reconocer que el ADN fija las posibilidades regulatorias, pero la **accesibilidad y el estado celular** deciden cuáles se realizan. Su límite es precisamente esa dependencia de perfiles experimentales del nuevo tipo celular. En la evolución del campo es un puente entre modelos de secuencia pura y modelos multimodales específicos de contexto.
