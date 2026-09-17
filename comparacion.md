# Comparación: de NCNet a modelos modernos de regulación genómica

## Una frase por modelo

- **NCNet:** clasifica 919 huellas regulatorias locales desde 1 kb de ADN y prueba CNN residuales + BiLSTM.
- **FactorNet:** predice unión TF en una célula nueva incorporando señales experimentales de esa célula.
- **Sei:** escala la clasificación local a 21,907 perfiles y los convierte en 40 actividades regulatorias interpretables.
- **ChromDL:** mejora directamente la tarea DeepSEA-919 con BiGRU + CNN + BiLSTM.
- **Enformer:** usa Transformers para relacionar elementos hasta ~100 kb y predecir perfiles regulatorios y expresión.
- **Borzoi:** usa atención + U-Net para predecir cobertura RNA-seq a 32 bp en ventanas de 524 kb.

## Comparación técnica

| Modelo | Año | Entrada | Datos extra además de ADN | Salida | Arquitectura dominante |
|---|---:|---:|---|---|---|
| NCNet | 2019 | 1 kb | No | 919 probabilidades binarias de perfil | ResNet 1D + BiLSTM |
| FactorNet | 2019 | ventanas genómicas | Sí: DNase, anotaciones, expresión según variante | unión TF por TF/célula | CNN-RNN multimodal |
| Sei | 2022 | 4 kb | No | 21,907 probabilidades; 40 clases derivadas | CNN residual dilatada + B-splines |
| ChromDL | 2023 | 1 kb | No | mismas 919 probabilidades de DeepSEA | BiGRU + CNN separable + BiLSTM |
| Enformer | 2021 | 196 kb | No | pistas cuantitativas a 128 bp | CNN + Transformer |
| Borzoi | 2025 | 524 kb | No en inferencia; entrenamiento multiasayo | cobertura RNA-seq y otras pistas a 32 bp | CNN + atención + U-Net |

## Resultados comparables y no comparables

La comparación numérica sólo es segura cuando la partición, las etiquetas y la métrica coinciden. De esta lista, **ChromDL vs DanQ/DeepSEA** es una comparación directa; **NCNet vs r-DanQ** es interna al artículo. Los demás cambian drásticamente objetivo o datos.

| Comparación válida reportada | Métrica | Resultado |
|---|---|---|
| NCNet-bRR vs r-DanQ | PR-AUC relativo | 114.05% de la línea base; ROC-AUC 103.64% |
| NCNet-RbR vs r-DanQ | PR-AUC relativo | 117.48%; coste de entrenamiento muy alto |
| ChromDL vs DanQ, mismas 919 etiquetas | mediana auROC / auPRC | 0.961 / 0.402 vs 0.951 / 0.372 |
| Sei, 21,907 perfiles | promedio AUROC / AUPRC | 0.972 / 0.409; protocolo distinto |
| Enformer vs Basenji2 | correlación de expresión | 0.85 vs 0.81; objetivo distinto |
| Borzoi vs Enformer | AUROC de eQTLs | 0.794 vs 0.747; datos/tarea distintos |

No se debe concluir que Borzoi “supera a ChromDL” por tener 0.794 frente a 0.961: el primero clasifica eQTLs, el segundo picos epigenómicos. Son unidades de evaluación diferentes.

## Evolución de arquitecturas

```text
motivos locales                 dependencia entre motivos            regulación distal / perfil denso
CNN ------------------------> RNN (DanQ, NCNet, ChromDL) ----------> atención (Enformer, Borzoi)
  |                                    |                                      |
motivo/TF local                   gramática en 1–4 kb                    enhancer-promotor y gen completo
```

1. **CNN:** detector local de motivos. Es eficiente y tiene sesgo inductivo apropiado: un motivo puede aparecer en cualquier posición.
2. **RNN bidireccional:** combina motivos en orden y distancia. NCNet y ChromDL exploran este camino; sus límites de memoria/coste se vuelven claros al aumentar la ventana.
3. **Convoluciones dilatadas/residuales:** Sei aumenta campo receptivo y profundidad sin una RNN secuencial completa.
4. **Transformers:** Enformer permite a posiciones distantes intercambiar información directamente, con coste mayor pero contexto largo.
5. **U-Net:** Borzoi combina contexto comprimido de larga distancia con detalle espacial a 32 bp.

## Qué aporta cada uno al problema biológico

| Necesidad | Modelo que mejor la aborda aquí | Por qué |
|---|---|---|
| Detectar perfiles regulatorios locales desde FASTA | ChromDL o NCNet | Salida directa de perfiles TF/DHS/histonas en 1 kb |
| Priorizar unión TF en un tipo celular con datos de accesibilidad | FactorNet | Usa explícitamente el contexto celular |
| Resumir una variante en actividades como enhancer/promotor de tejido | Sei | Las 40 clases hacen interpretable el gran vector de perfiles |
| Conectar elementos distales con expresión | Enformer | Atención sobre ~200 kb |
| Estudiar expresión, exones, splicing y poliadenilación | Borzoi | Predice cobertura RNA-seq de alta resolución |

## Recomendación práctica para un proyecto de ML

Si el objetivo es reproducir y entender el problema, empezar con el benchmark DeepSEA-919 y ChromDL/DanQ es razonable: la entrada y la salida son manejables y las métricas están bien definidas. Si el objetivo es priorización de variantes cerca de genes con regulación distal, conviene usar Enformer o Borzoi preentrenados antes que entrenar uno desde cero. Si se necesita un resumen regulatorio interpretable a través de muchos tejidos, Sei es una buena interfaz.

En todos los casos, tratar las predicciones como evidencia *in silico*. La verificación de causalidad requiere datos de expresión, QTL, CRISPR, MPRA u otro experimento apropiado.
