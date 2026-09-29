# Circuiti causali per la stima della severità PHQ-8 da trascrizioni

## Titolo provvisorio

*What does a small language model use to estimate PHQ-8 severity? A causal mechanistic analysis on DAIC-WOZ interviews*

## Idea in breve

La tesi costruisce un predittore testuale del punteggio PHQ-8 o della soglia `PHQ-8 >= 10`, ma il contributo principale non è massimizzare l'accuratezza. L'obiettivo è stabilire **quali componenti interne del modello causano la decisione** e se esse rappresentano contenuto linguisticamente pertinente oppure scorciatoie del dataset, come lunghezza, disfluenze, domanda dell'intervistatore o struttura del colloquio.

Il sistema va presentato come supporto sperimentale allo screening e come audit di un modello, non come diagnosi automatica né come scoperta di biomarcatori clinici.

## Domanda principale

> Un piccolo modello linguistico open-weight usa rappresentazioni interne riconducibili al contenuto delle risposte per stimare la severità PHQ-8, oppure basa la decisione soprattutto su artefatti del protocollo d'intervista?

## Domande di ricerca e ipotesi

### RQ1 — Esiste un segnale predittivo testuale riproducibile?

Confrontare majority baseline, TF-IDF + regressione logistica/Ridge, encoder Transformer e decoder generativo con output binario. L'ipotesi è che un modello neurale possa superare la baseline lessicale, ma il confronto deve essere svolto con split per partecipante e senza usare il test set per selezionare modello o soglie.

### RQ2 — Quali feature e circuiti contribuiscono al logit di classe?

Usare Jacobian Lens/J-space per osservare i contenuti verbalizzabili nei layer e Circuit Tracer per costruire grafi di attribuzione del contrasto:

```text
g(x) = logit(at_or_above_10) - logit(below_10)
```

L'ipotesi è che almeno una parte del contrasto sia mediata da feature ricorrenti associate a temi presenti nelle risposte, ma va testata contro feature relative alla domanda, alla lunghezza e allo stile.

### RQ3 — Le feature individuate sono causalmente rilevanti?

Eseguire feature ablation e activation/feature patching. Una feature è una spiegazione candidata soltanto se l'intervento produce un cambiamento del logit nella direzione prevista e un effetto maggiore di controlli casuali o abbinati per layer e livello di attivazione.

## Dati e unità di analisi

- Dataset principale: DAIC-WOZ.
- Unità statistica: partecipante/sessione, non singolo turno.
- Target primario consigliato: `PHQ8_Binary`, perché permette un contrasto di logits pulito.
- Target secondario: `PHQ8_Score`, valutato con regressione ma non necessariamente usato per Circuit Tracer.
- Input primario: soli turni del partecipante (`P-only`).
- Controlli: `E-only`, `E+P`, lunghezza, numero di turni e tipo di domanda.

## Piano sperimentale quantitativo

1. Fissare split per partecipante e ripeterli con pochi seed predefiniti oppure usare nested cross-validation compatibile con il numero di sessioni.
2. Valutare classificazione con balanced accuracy, macro-F1, AUROC, AUPRC e Brier score; valutare regressione con MAE, RMSE e correlazione di Spearman.
3. Selezionare prima dell'analisi 50–100 casi bilanciati, includendo veri positivi, veri negativi ed errori.
4. Estrarre feature/circuiti con soglie di pruning dichiarate; misurare overlap pesato dei nodi e degli edge tra casi e split.
5. Per ogni feature candidata calcolare `delta-logit`, variazione di probabilità e class flip rate dopo ablation o patching.
6. Confrontare gli effetti con feature casuali e feature abbinate; usare intervalli bootstrap al 95% e test di permutazione.
7. Controllare la selettività: un intervento deve modificare il target PHQ-8 più di un target linguistico non correlato.

## Modelli Hugging Face già addestrati, tutti entro 3B

Verifica dei repository effettuata il 29 settembre 2026. “Già addestrato” significa che i pesi di base sono disponibili; non significa che il modello sia già validato per PHQ-8 o DAIC-WOZ.

| Modello | Parametri | Ruolo proposto | Compatibilità meccanicistica | Limite |
| --- | ---: | --- | --- | --- |
| [google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b) | circa 2,6B effettivi | Modello principale, backbone congelato e classificazione generativa A/B | Jacobian Lens pubblico; PLT/CLT e demo mature in Circuit Tracer | Accesso soggetto alla licenza Gemma; contesto e memoria delle attivazioni vanno verificati |
| [Qwen/Qwen3-1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | 1,7B | Replica con architettura diversa | Jacobian Lens e transcoders Circuit Tracer pubblici | Toolchain più recente; fissare commit e checkpoint esatti |
| [google/gemma-3-1b-it](https://huggingface.co/google/gemma-3-1b-it) | 1B | Alternativa leggera instruction-tuned | Lens e transcoders pubblici per la famiglia Gemma 3 | Meno maturo del percorso Gemma 2 2B; licenza Gemma |
| [meta-llama/Llama-3.2-1B](https://huggingface.co/meta-llama/Llama-3.2-1B) | circa 1,2B | Controllo generativo leggero | Transcoders e demo Circuit Tracer disponibili | Licenza Llama; al momento non è la scelta più completa per Jacobian Lens |
| [answerdotai/ModernBERT-large](https://huggingface.co/answerdotai/ModernBERT-large) | 395M | Baseline encoder con classifier head | Probing e ablation standard, non la pipeline J-space/Circuit Tracer prevista | Richiede fine-tuning supervisionato; non genera logits A/B nello stesso formato del decoder |

La scelta iniziale consigliata è **Gemma 2 2B** perché minimizza il rischio tecnico dell'interpretabilità. **Qwen3 1.7B** è la replica più utile per verificare che i risultati non siano specifici di una sola famiglia.

## Risultato minimo pubblicabile

- baseline predittive riproducibili;
- un protocollo per aggregare circuiti su più casi;
- almeno una famiglia di feature sottoposta ad ablation/patching con controlli;
- una failure table che mostri quando il circuito non è stabile o è dominato da artefatti.

Un risultato negativo è valido: se i circuiti non sono replicabili o gli interventi non sono selettivi, la tesi dimostra un limite concreto dell'uso di spiegazioni meccanicistiche su un dataset piccolo e sensibile.

## Rischi e contenimento

- **Campione ridotto:** congelare il backbone, limitare i gradi di libertà e usare baseline regolarizzate.
- **Lens/transcoders non validi dopo LoRA:** iniziare senza fine-tuning; un eventuale LoRA richiede una rivalidazione separata.
- **Grafi troppo grandi:** limitare lunghezza e numero di casi, salvare il grafo completo e dichiarare la potatura.
- **Interpretazione clinica eccessiva:** descrivere feature del modello, non stati mentali del partecipante.

## Fattibilità

**Alta-medio alta.** È la proposta più coerente con l'obiettivo di explainability meccanicistica e dispone già di una toolchain pubblica per modelli sotto 3B. La difficoltà principale è la rigorosa aggregazione statistica dei risultati, non l'addestramento del modello.

## Risorse tecniche

- [Circuit Tracer e transcoders disponibili](https://github.com/decoderesearch/circuit-tracer)
- [Jacobian Lens pubblici su Hugging Face](https://huggingface.co/neuronpedia/jacobian-lens)
- [Descrizione locale di Circuit Tracer](../circuit-tracer.md)
- [Descrizione locale di J-space e Jacobian Lens](../jspace-jacobian-lens.md)

