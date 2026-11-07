# ScSLINT

A scalable framework for instance matching on linked data: it finds the resources in two RDF repositories that describe the same real-world entity. Each step (property alignment, blocking, similarity matching, filtering) can run on its own or as one pipeline, and similarity functions and aggregators can be added. Written in C++ to keep large repositories within the memory of one desktop machine.

> Khai Nguyen and Ryutaro Ichise. [ScSLINT: Time and Memory Efficient Interlinking Framework for Linked Data](https://ceur-ws.org/Vol-1486/paper_34.pdf). Proceedings of the 14th International Semantic Web Conference, Posters and Demonstrations Track. CEUR-WS.org, vol. 1486, 2015.

The original release, the indexing tools and the datasets are on the [project page](https://www.ai.iee.eng.isct.ac.jp/ScSLINT/index.html). Copyright © Ichise Laboratory.

## Build

```sh
cmake -S . -B build
cmake --build build --parallel
```

This produces `build/scslint`. It needs CMake 3.20 and a C++11 compiler.

## Input

ScSLINT reads repositories in its own binary indexed format, not RDF directly. KIndexer, in `tools.zip` on the project page, converts N-Triples (`.nt`) into that format. The project page also offers a small e-commerce dataset already indexed, and the OAEI 2010 and 2012 datasets on request.

## Usage

Arguments are `key=value` pairs:

```sh
build/scslint job=default src=<source repository> trg=<target repository> name=<job name> output=<directory>
```

`job=default` runs the whole pipeline and skips any step whose output already exists in `output`: alignment (`.align`), blocking (`.block`), a default configuration (`.config.default`), matching (`.score`), then filtering.

Each step can also run alone:

| `job` | Step |
|---|---|
| `align` | align properties between `src` and `trg` |
| `block` | generate candidate pairs from an alignment (`align`); `split` and `fold` limit it to training, validation or test |
| `match` | score candidates (`block`) with a configuration (`config`) |
| `filter` | select links from scores (`score`), by stable filtering or the top `topk` per instance |
| `split` | split labeled links (`label`) into training, validation and test sets; `training`, `validation`, `folds`, `repeat` |
| `clearn` | learn a matching configuration on the training split, checked on the validation split, with `algorithm` = `naive`, `heuristic`, `exhaustive`, `infogain`, `genetic` or `acceptall` |
| `clink` | supervised linking: learn blocking and a configuration from labeled links (`align`, `label`, `split`), then link |
| `ser` | write similarity scores and labels as features (`.feature`, `.info`) for an external classifier |
| `collect` | evaluate a step's output against the labeled links |

Other parameters: `thread`, `topsim` (number of similarity functions kept), `filtering`, `temp` (working directory), `log` (log file, default `<timestamp>.txt`).
