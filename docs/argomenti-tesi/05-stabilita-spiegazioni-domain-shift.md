# Stabilità delle spiegazioni sotto shift demografico, di protocollo e di dominio

## Titolo provvisorio

*Mechanistic explanation stability under demographic and interview-to-social-media shift*

## Idea in breve

La tesi verifica se le spiegazioni interne individuate su DAIC-WOZ restano valide quando cambiano sottogruppo, struttura dell'intervista o dominio testuale. L'estensione naturale è SWMH/Reddit, usato non per “validare” il PHQ-8, ma per studiare quali meccanismi sopravvivono quando scompaiono Ellie, le domande semi-strutturate e il formato dialogico.

DAIC e SWMH hanno label diverse: `PHQ8 >= 10` e appartenenza a una community non sono equivalenti. Il confronto riguarda la **trasferibilità del comportamento del modello e delle sue feature**, non l'equivalenza clinica dei target.

## Domanda principale

> Le feature e i circuiti associati al linguaggio depression-related sono stabili sotto cambio di sottogruppo, protocollo e dominio, oppure il modello ricostruisce la decisione usando shortcut specifiche della fonte dei dati?

## Domande di ricerca

### RQ1 — La prestazione e la calibrazione degradano sotto shift?

Addestrare/calibrare su un dominio o sottogruppo e valutare sull'altro. Misurare separatamente discriminazione, calibrazione e selective risk; non limitarsi alla sola accuracy.

### RQ2 — I concetti J-space si trasferiscono?

Definire prima un'ontologia comune: sonno/energia, umore, autosvalutazione, supporto sociale, protocollo, auto-diagnosi, stile e contenuto neutro. Confrontare frequenza, persistenza per layer e associazione con il logit gap su DAIC, SWMH raw e SWMH keyword-controlled.

### RQ3 — I circuiti causali vengono riusati?

Confrontare overlap pesato di feature/edge e contributo semantico dei circuiti. Eseguire ablation delle feature selezionate senza guardare il dominio di test. Una feature trasferibile deve conservare un effetto selettivo, anche se ridotto.

### RQ4 — Possiamo distinguere shortcut di intervista e shortcut di community?

Applicare controlli paralleli:

| DAIC-WOZ | SWMH/Reddit |
| --- | --- |
| rimuovere/permutare la domanda di Ellie | mascherare subreddit, community e auto-diagnosi esplicite |
| length matching | length matching |
| `P-only` contro `E+P` | post senza metadata contro post originale |
| normalizzazione filler/disfluenze | parafrasi/stile controllato |

## Piano quantitativo

- performance in-domain e out-of-domain: balanced accuracy, macro-F1, AUROC/AUPRC;
- calibrazione: Brier score, ECE e reliability diagram;
- generalization gap assoluto e relativo;
- risk-coverage curve e quota di astensione necessaria per mantenere un rischio target;
- overlap pesato di feature e edge, similarità tra sottografi e stabilità delle categorie semantiche;
- delta-logit, flip rate e selettività dopo ablation in-domain e cross-domain;
- risultati per sottogruppo con intervalli bootstrap, evitando conclusioni se la numerosità è insufficiente.

Per il domain shift usare un modello a effetti misti o bootstrap gerarchico con dominio e trasformazione come fattori. Per i sottogruppi riportare soprattutto stime e intervalli, non solo p-value. Nessun hyperparameter va scelto sul dominio di test.

## Modelli Hugging Face già addestrati, tutti entro 3B

| Modello | Parametri | Ruolo | Cautela |
| --- | ---: | --- | --- |
| [google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b) | circa 2,6B | Backbone comune per il confronto meccanicistico tra domini | Ottimo supporto J-space/Circuit Tracer; principalmente inglese |
| [Qwen/Qwen3-1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | 1,7B | Replica su un modello multilingue e Apache-2.0 | Toolchain recente; controllare versioni di lens/transcoders |
| [answerdotai/ModernBERT-large](https://huggingface.co/answerdotai/ModernBERT-large) | 395M | Baseline generalista long-context | Fine-tuning supervisionato necessario |
| [mental/mental-roberta-base](https://huggingface.co/mental/mental-roberta-base) | circa 125M | Baseline pre-addestrata su Reddit mental-health | Può essere favorita su SWMH per somiglianza di dominio; licenza CC-BY-NC-4.0 e accesso gated |
| [rafalposwiata/deproberta-large-v1](https://huggingface.co/rafalposwiata/deproberta-large-v1) | circa 0,4B | Encoder pre-addestrato su post depressivi Reddit | Ideale per misurare il vantaggio/shortcut di dominio, non per equivalenza con PHQ-8 |
| [rafalposwiata/deproberta-large-depression](https://huggingface.co/rafalposwiata/deproberta-large-depression) | circa 0,4B | Baseline già fine-tuned su livelli di depressione da social media | Label moderate/severe da shared task social ≠ score clinico; usare solo come baseline esterna congelata |

La presenza di modelli social-media-specifici rende l'esperimento interessante: ci si aspetta che abbiano buone prestazioni in-domain ma siano più vulnerabili a keyword e stile di community. Questa è un'ipotesi da testare, non un presupposto.

## Esperimento minimo

1. DAIC `P-only` e SWMH depression/control con backbone congelato.
2. Tre versioni SWMH: raw, keyword-controlled e length/topic-matched.
3. Valutazione predittiva e di calibrazione separata per dominio.
4. Analisi J-space/Circuit Tracer su un campione matched.
5. Ablation di poche feature selezionate nel dominio sorgente e valutate nel dominio target.
6. Failure table per shortcut, dominio e sottogruppo.

## Contributo atteso

- definizione operativa della stabilità di una spiegazione meccanicistica sotto shift;
- distinzione tra circuiti semantici, circuiti di protocollo e circuiti di community;
- valutazione delle spiegazioni come strumenti di audit del bias di dominio;
- protocollo riutilizzabile per evitare di confondere accuratezza in-domain e spiegazione trasferibile.

## Limiti etici e di validità

- I label SWMH sono proxy di community, non diagnosi né misure individuali.
- Non pubblicare post, ID o esempi riconoscibili; rispettare accesso controllato e licenza.
- Non usare suicidal ideation come funzionalità operativa di screening.
- Le differenze demografiche possono essere non identificabili con la numerosità disponibile; una conclusione “nessuna differenza” richiede potenza adeguata.
- Un circuito stabile tra i domini descrive il modello, non dimostra un meccanismo psicologico universale.

## Fattibilità

**Media come tesi autonoma; alta come capitolo finale della proposta sugli shortcut.** Il costo principale è ottenere SWMH e costruire confronti validi tra target non equivalenti. Senza accesso a SWMH, la stessa struttura può essere applicata a shift tra famiglie di domanda e sottogruppi DAIC, con conclusioni più limitate.

## Risorse

- [Jacobian Lens](https://huggingface.co/neuronpedia/jacobian-lens)
- [Circuit Tracer e lista transcoders](https://github.com/decoderesearch/circuit-tracer)
- [Descrizione locale dell'estensione SWMH](../../misc/brainstorming.md)

