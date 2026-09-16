# eccDNA FCGR Siamese Network

Progetto di Bioinformatica per l’analisi di sequenze di **DNA extracromosomico circolare (eccDNA)** tramite **Frequency Chaos Game Representation (FCGR)** e reti neurali.

L’obiettivo è verificare se la composizione delle sequenze eccDNA contenga un segnale utile per distinguere o mettere in relazione campioni associati a diverse condizioni biologiche.

## Pipeline

```text
Sequenza eccDNA
      ↓
Estrazione k-mer
      ↓
FCGR
      ↓
CNN Encoder
      ↓
Rappresentazione 128D
      ↓
 ┌───────────────────┬───────────────────┐
 │ Siamese Network   │ CNN Multiclass    │
 │ Metric Learning   │ Classificazione   │
 │ Similarity Ranking│ diretta 12 classi │
 └───────────────────┴───────────────────┘
```

La configurazione finale utilizza:

- `k = 6`
- FCGR `64 × 64`
- 12 classi: **11 tumorali + Healthy**

I dati derivano principalmente da **CircleBase V2** ed **eccDNABase**.

## Modelli

### Siamese Network

La rete Siamese utilizza un encoder CNN condiviso e apprende uno spazio latente in cui campioni della stessa classe vengono avvicinati e campioni di classi differenti vengono allontanati.

Il training utilizza:

- Contrastive Loss con distanza Euclidea
- Dynamic Semi-Hard Negative Mining
- embedding 128D normalizzati L2

In fase di inferenza, la classificazione viene ottenuta confrontando il nuovo embedding con una reference bank tramite **cosine similarity**.

### CNN Multiclass

La CNN diretta utilizza lo stesso backbone della Siamese, ma la rappresentazione 128D viene utilizzata direttamente per la classificazione nelle 12 classi.

Questo permette di confrontare il metric learning con una classificazione supervisionata standard.

## Risultati principali

| Modello | Top-1 | Macro-F1 | Top-3 | Top-5 |
|---|---:|---:|---:|---:|
| Siamese Downsampled | 31.58% | 27.84% | 65.83% | 82.53% |
| Siamese Full-Train | 30.68% | 26.88% | 66.48% | 83.34% |
| **CNN Diretta** | **32.83%** | **28.58%** | **67.59%** | **84.09%** |

La CNN diretta ottiene le migliori prestazioni complessive nella classificazione multiclass.

La Siamese rimane utile per analisi di **similarità, retrieval e ranking** dei campioni.

## Struttura del progetto

```text
eccdna_fcgr_siamese/
├── data/
├── notebooks/
├── src/
├── artifacts/
├── environment.yml
└── README.md
```

I file di grandi dimensioni, come cache FCGR e checkpoint, non vengono tracciati da Git.

## Ambiente

Creazione dell’ambiente:

```bash
conda env create -f environment.yml
```

Attivazione:

```bash
conda activate eccdna_fcgr_clean
```

## Conclusione

Le FCGR mostrano un segnale composizionale associato alla condizione biologica, ma diverse classi rimangono fortemente sovrapposte.

Il metric learning risulta utile per organizzare i campioni in uno spazio di similarità, mentre la CNN diretta è più efficace per la classificazione multiclass.
