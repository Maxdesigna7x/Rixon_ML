# Modelos de secuencia a función regulatoria

Esta carpeta contiene una guía de lectura en español para una persona con base de ML y poca biología. El punto de partida es NCNet (2019) y los trabajos posteriores se separan en dos grupos:

- **Comparación directa:** conserva la entrada de 1 kb y las 919 etiquetas del benchmark DeepSEA (ChromDL).
- **Extensiones del mismo problema:** siguen prediciendo función regulatoria desde ADN, pero aumentan contexto, número de ensayos, resolución o incorporan datos de contexto celular.

## Contenido

- [Resumen de NCNet](summaries/00-ncnet.md)
- [FactorNet](summaries/01-factornet.md)
- [Sei](summaries/02-sei.md)
- [ChromDL](summaries/03-chromdl.md)
- [Enformer](summaries/04-enformer.md)
- [Borzoi](summaries/05-borzoi.md)
- [Comparación transversal](comparacion.md)

## PDFs descargados

Los siguientes archivos se verificaron como PDFs legibles. FactorNet, ChromDL y Borzoi se guardan como preprints abiertos; los resúmenes indican además el DOI de la versión revisada por pares cuando existe.

| Archivo | Versión |
|---|---|
| `papers/factornet-2017-preprint.pdf` | Preprint que antecede a FactorNet (2019) |
| `papers/sei-2022.pdf` | Artículo publicado en *Nature Genetics* |
| `papers/chromdl-2023-preprint.pdf` | Preprint que antecede a ChromDL (2023) |
| `papers/enformer-2021.pdf` | Artículo publicado en *Nature Methods* |
| `papers/borzoi-2023-preprint.pdf` | Preprint que antecede a Borzoi (2025) |

Los resúmenes incluyen DOI, código y enlace al artículo para localizar la versión editorial correspondiente.

## Convención de lectura

Un valor de salida alto significa: “según los experimentos usados para entrenar, esta secuencia parece compatible con esta señal regulatoria”. No prueba por sí mismo causalidad, actividad en todos los tejidos ni patogenicidad clínica.
