# Rixon_ML

Material de investigación y documentación técnica sobre modelos de *deep learning* que predicen función regulatoria a partir de ADN no codificante.

## Investigación disponible

La colección principal está en [research/noncoding-dna](research/noncoding-dna/README.md). Está pensada para lectores con conocimientos de ML que están entrando en genómica regulatoria.

Incluye:

- Un [resumen completo de NCNet](research/noncoding-dna/summaries/00-ncnet.md): tarea, dominio, entradas, salidas, arquitectura, evaluación y límites.
- Resúmenes de sus sucesores relevantes: [FactorNet](research/noncoding-dna/summaries/01-factornet.md), [Sei](research/noncoding-dna/summaries/02-sei.md), [ChromDL](research/noncoding-dna/summaries/03-chromdl.md), [Enformer](research/noncoding-dna/summaries/04-enformer.md) y [Borzoi](research/noncoding-dna/summaries/05-borzoi.md).
- Una [comparación transversal](research/noncoding-dna/comparacion.md) de arquitecturas, tareas, entradas, salidas, métricas reportadas y aportaciones.
- Copias de los artículos en PDF en [research/noncoding-dna/papers](research/noncoding-dna/papers), incluidos preprints abiertos cuando la versión editorial no está disponible libremente.

## Alcance

Estos modelos no diagnostican enfermedades directamente. Predicen señales regulatorias, expresión o cobertura RNA-seq desde la secuencia de ADN; sus resultados deben tratarse como evidencia *in silico* y validarse con datos experimentales cuando se requiera una conclusión causal.
