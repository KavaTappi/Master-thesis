# Spiegazioni multimodali causali che collegano testo e voce

## Titolo provvisorio

*Causal cross-modal explanations for distress prediction from interview transcripts and speech*

## Idea in breve

La tesi confronta informazione linguistica e vocale nelle interviste DAIC-WOZ. Prima stabilisce se l'audio aggiunge davvero segnale rispetto al testo; poi verifica causalmente se il modello di fusione usa le due modalità in modo complementare o se una modalità domina la decisione.

L'approccio raccomandato è progressivo: feature acustiche interpretabili e fusione tardiva prima di introdurre encoder audio neurali. In questo modo un eventuale guadagno può essere attribuito a pause, F0, intensità, qualità vocale o embedding audio senza confondere il risultato con la sola capacità del modello.

## Domanda principale

> Il guadagno multimodale nella stima del PHQ-8 proviene da informazione complementare tra testo e voce, e tale contributo può essere verificato mediante interventi sulle rappresentazioni delle due modalità?

## Domande di ricerca

### RQ1 — L'audio aggiunge valore predittivo oltre al testo?

Confrontare testo, audio interpretabile, embedding audio e fusione tardiva usando gli stessi split per partecipante. La fusione è giustificata soltanto se il miglioramento è stabile e supera l'incertezza statistica.

### RQ2 — Quale modalità guida la decisione?

Eseguire modality ablation, mascheramento di segmenti e sostituzione delle rappresentazioni audio/testo tra coppie abbinate. Misurare quanto cambia il logit PHQ-8 e se il modello compensa l'assenza di una modalità.

### RQ3 — Le interazioni cross-modali sono interpretabili e causali?

Due disegni possibili:

1. **Percorso consigliato:** convertire feature acustiche aggregate in token trasparenti, come `PAUSE_HIGH` o `F0_VARIABILITY_LOW`, e analizzare con Circuit Tracer come il decoder le integra al testo.
2. **Percorso avanzato:** usare un encoder audio e un piccolo modulo di fusione, quindi applicare probing, gradient/activation patching e ablation separatamente sui due rami.

Il primo percorso spiega come il language model usa proxy acustici; non spiega l'onda audio grezza. Il secondo è più completo ma anche più fragile.

## Dati e preprocessing

- Transcript con timestamp e separazione Ellie/partecipante.
- Audio a 16 kHz, segmentato sui turni del partecipante.
- COVAREP e formanti già forniti dal dataset come baseline primaria.
- Maschere per frame non voiced, segmenti scrubbed e possibili bleed-over dell'intervistatore.
- Aggregazioni per turno/sessione: media, deviazione, quantili, slope, pause, voiced ratio e durata.

## Piano sperimentale quantitativo

| Sistema | Scopo |
| --- | --- |
| TF-IDF / ModernBERT | baseline testuale |
| COVAREP/Formanti + Elastic Net o XGBoost leggero | baseline audio interpretabile |
| embedding WavLM + classifier regolarizzato | baseline audio neurale |
| fusione tardiva calibrata | test di complementarità |
| decoder con testo + token acustici | audit meccanicistico condiviso |

Metriche:

- balanced accuracy, macro-F1, AUROC e AUPRC per il target binario;
- MAE/RMSE per score continuo;
- delta di prestazione e delta-logit dopo modality ablation;
- conditional permutation importance delle feature acustiche;
- interaction gain: performance della fusione meno migliore unimodale;
- calibrazione con Brier score ed ECE;
- stabilità dell'effetto su split, seed e famiglie di domanda.

Usare bootstrap gerarchico per partecipante. Per confrontare sistemi sul target binario usare McNemar e intervalli bootstrap delle differenze; per le ablation usare test appaiati/permutazione. Correggere per la selezione di molte feature acustiche.

## Modelli Hugging Face già addestrati, tutti entro 3B

### Ramo testuale

| Modello | Parametri | Uso |
| --- | ---: | --- |
| [google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b) | circa 2,6B | Decoder principale per testo + token acustici; supporto maturo a J-space/Circuit Tracer |
| [Qwen/Qwen3-1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | 1,7B | Replica più permissiva come licenza e architettura |
| [answerdotai/ModernBERT-large](https://huggingface.co/answerdotai/ModernBERT-large) | 395M | Baseline encoder sulle trascrizioni lunghe |

### Ramo audio

| Modello | Parametri | Stato dei pesi | Uso e cautela |
| --- | ---: | --- | --- |
| [microsoft/wavlm-base-plus](https://huggingface.co/microsoft/wavlm-base-plus) | circa 95M | Pre-addestrato self-supervised su 94k ore di parlato | Estrattore di rappresentazioni; richiede classifier head sul task e non è un modello di depressione |
| [superb/wav2vec2-base-superb-er](https://huggingface.co/superb/wav2vec2-base-superb-er) | circa 95M | Fine-tuned per emotion recognition su IEMOCAP | Baseline paralinguistica; le quattro emozioni IEMOCAP non equivalgono al PHQ-8 |
| [openai/whisper-small](https://huggingface.co/openai/whisper-small) | 244M | Pre-addestrato per ASR/translation su 680k ore | Controllo sulla qualità delle trascrizioni o feature dell'encoder; la model card sconsiglia inferenze soggettive dirette dall'audio |

Whisper va usato per trascrizione/controllo ASR, non come classificatore diretto di depressione. WavLM è più appropriato come encoder sperimentale, ma le feature COVAREP restano la baseline più interpretabile.

## Esperimento minimo

1. Testo `P-only` con baseline lineare/ModernBERT.
2. COVAREP/Formanti aggregati con modello regolarizzato.
3. Fusione tardiva degli score, valutata su split identici.
4. Solo se la fusione migliora: testo + 5–10 token acustici predefiniti in Gemma 2 2B.
5. Tracciamento e ablation dei token/feature acustici rispetto al logit gap.

## Contributo atteso

- quantificazione del valore incrementale della voce;
- confronto tra segnali acustici interpretabili ed embedding neurali;
- spiegazioni multimodali sottoposte a modality ablation e patching;
- indicazioni chiare su quando la fusione è reale e quando maschera dominanza o leakage.

## Rischi

- allineamento temporale imperfetto e segmenti scrubbed;
- forte rischio di overfitting con 189 sessioni;
- feature audio correlate a genere, microfono o durata;
- assenza di transcoders per l'intero sistema audio-testo: non chiamare “Circuit Tracer multimodale” un'analisi limitata al decoder testuale.

## Fattibilità

**Media-bassa per la versione completa; medio-alta per testo + feature acustiche interpretabili.** Va scelta solo se gli asset audio sono accessibili e la baseline unimodale mostra un segnale stabile.

## Risorse

- [Inventario locale degli asset DAIC](../daic-features-purpose-models.md)
- [Circuit Tracer](https://github.com/decoderesearch/circuit-tracer)
- [WavLM Base Plus](https://huggingface.co/microsoft/wavlm-base-plus)
- [Whisper Small](https://huggingface.co/openai/whisper-small)

