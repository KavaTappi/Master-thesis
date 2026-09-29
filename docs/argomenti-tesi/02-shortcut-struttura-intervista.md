# Shortcut della struttura d'intervista nei modelli di screening testuale

## Titolo provvisorio

*Question-conditioned mechanistic auditing of depression-screening language models*

## Idea in breve

DAIC-WOZ è un'intervista semi-strutturata: molte domande di Ellie ricorrono tra partecipanti, ma ordine, presenza e lunghezza delle risposte possono variare. Questa struttura rende possibile un audit controllato: il modello predice dal contenuto del partecipante oppure riconosce indirettamente la domanda e la forma del protocollo?

La proposta è più focalizzata della tesi generale sui circuiti causali. Il suo contributo è un **protocollo controfattuale riproducibile** per distinguere evidenza della risposta da shortcut dell'intervistatore.

## Domanda principale

> Quanto della prestazione di un modello che stima la classe PHQ-8 deriva dal contenuto della risposta e quanto dalla domanda, dalla lunghezza e dalla posizione nel colloquio?

## Domande di ricerca

### RQ1 — Quanto predicono separatamente domanda e risposta?

Confrontare quattro condizioni:

1. `P-only`: risposta del partecipante;
2. `E-only`: domanda/intervento di Ellie;
3. `E+P`: domanda e risposta;
4. `P-matched`: risposta con distribuzioni bilanciate per domanda e lunghezza.

Se `E-only` o le sole feature strutturali ottengono una prestazione sopra il caso, esiste un segnale di protocollo che deve essere esplicitamente controllato.

### RQ2 — La decisione è invariabile a trasformazioni che preservano il contenuto?

Creare controfattuali con domanda rimossa o permutata, troncamento a uguale lunghezza, normalizzazione di filler e disfluenze e parafrasi controllate. Per trasformazioni semanticamente conservative, la predizione dovrebbe rimanere stabile.

### RQ3 — Esistono circuiti distinti per contenuto e protocollo?

Confrontare feature J-space e grafi Circuit Tracer tra input originali e controfattuali. L'ipotesi è che le feature semantiche restino più stabili, mentre quelle di shortcut diminuiscano quando il confondente viene rimosso.

### RQ4 — Il modello generalizza a tipi di domanda non osservati?

Usare una valutazione leave-question-family-out: il modello viene addestrato o calibrato senza una famiglia di domande e testato sui turni corrispondenti. Questo misura se apprende un segnale trasferibile oppure una mappa domanda-label specifica del corpus.

## Piano sperimentale quantitativo

| Blocco | Misure principali |
| --- | --- |
| Predizione | balanced accuracy, macro-F1, AUROC e AUPRC per `P-only`, `E-only`, `E+P` e `P-matched` |
| Invarianza | prediction agreement, class flip rate, variazione assoluta del logit gap e divergenza Jensen-Shannon |
| Shortcut | differenza di prestazione tra input originale e domanda permutata/rimossa; mutual information tra predizione e famiglia di domanda |
| Meccanismi | overlap pesato di feature/edge, variazione dell'attribuzione per categoria e delta-logit dopo ablation |
| Generalizzazione | degrado leave-question-family-out con intervalli bootstrap |

Le trasformazioni vanno applicate in coppia allo stesso esempio. Usare test di McNemar per i flip, bootstrap appaiato per le differenze di metrica e test di permutazione per l'associazione tra feature e condizione. La validità semantica delle parafrasi va controllata manualmente su un campione cieco.

## Modelli Hugging Face già addestrati, tutti entro 3B

| Modello | Parametri | Uso nella tesi | Perché considerarlo |
| --- | ---: | --- | --- |
| [google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b) | circa 2,6B | Audit principale del logit A/B | È il percorso più maturo per Jacobian Lens + Circuit Tracer e permette interventi sulle feature |
| [Qwen/Qwen3-1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | 1,7B | Replica architetturale | Apache-2.0, contesto lungo e disponibilità sia di lens sia di transcoders |
| [google/gemma-3-1b-it](https://huggingface.co/google/gemma-3-1b-it) | 1B | Modello leggero per la suite controfattuale | Riduce costo e memoria; lens/transcoders pubblici, da validare sulla versione esatta |
| [answerdotai/ModernBERT-large](https://huggingface.co/answerdotai/ModernBERT-large) | 395M | Baseline discriminativa long-context | Context window nativa fino a 8192 token; utile per capire se l'effetto è specifico dei decoder generativi |
| [mental/mental-roberta-base](https://huggingface.co/mental/mental-roberta-base) | circa 125M | Baseline con pretraining di dominio | Pre-addestrato su post Reddit di salute mentale; utile ma soggetto a domain shift e licenza CC-BY-NC-4.0 |

La baseline di dominio non va trattata come modello clinico: MentalRoBERTa è pre-addestrato su social media, non su interviste PHQ-8. Proprio questa differenza può essere misurata, ma non nascosta.

## Esperimento minimo

1. Raggruppare 5–10 famiglie di domande frequenti.
2. Costruire `P-only`, `E-only` ed `E+P` con split per soggetto.
3. Addestrare TF-IDF e ModernBERT; valutare Gemma 2 2B con prompt A/B congelato.
4. Applicare rimozione/permutazione della domanda e length matching.
5. Tracciare 50–100 casi con Circuit Tracer e validare alcune feature con ablation.

## Contributo atteso

- una stima quantitativa del leakage dovuto al protocollo;
- una suite di controfattuali riutilizzabile;
- una separazione tra feature semantiche e feature di shortcut supportata da interventi;
- raccomandazioni su quali rappresentazioni dell'intervista usare in futuri lavori DAIC-WOZ.

## Rischi

- Una domanda non è sempre semanticamente separabile dalla risposta: alcune risposte brevi dipendono dal contesto. Le condizioni `P-only` ed `E+P` vanno quindi interpretate insieme.
- La parafrasi automatica può alterare significato o tono. Deve essere un'analisi secondaria, con controllo umano.
- Una buona stabilità non dimostra validità clinica; dimostra soltanto che il modello non dipende da quel confondente specifico.

## Fattibilità

**Alta.** È il filone più adatto a una tesi magistrale solo testuale: domanda precisa, controlli naturali nel dataset e una parte quantitativa completa anche se l'analisi meccanicistica producesse risultati negativi.

## Risorse

- [Circuit Tracer](https://github.com/decoderesearch/circuit-tracer)
- [Jacobian Lens](https://huggingface.co/neuronpedia/jacobian-lens)
- [Inventario locale di dati e modelli DAIC](../daic-features-purpose-models.md)

