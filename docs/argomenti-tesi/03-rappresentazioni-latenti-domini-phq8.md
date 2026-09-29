# Rappresentazioni latenti interpretabili dei domini PHQ-8

## Titolo provvisorio

*Disentangling symptom-related concepts in small language-model representations of clinical interviews*

## Idea in breve

Invece di comprimere l'intera intervista in un unico punteggio, la tesi studia se le rappresentazioni interne separano concetti riconducibili agli otto domini del PHQ-8: interesse/piacere, umore, sonno, energia, appetito, autovalutazione, concentrazione e rallentamento/agitazione.

La domanda riguarda il **modello**, non la persona: una direzione latente associata al sonno indica che il modello usa una regolarità testuale di quel tipo, non che abbia identificato un sintomo clinico.

## Prerequisito decisivo

Verificare che la distribuzione disponibile di DAIC-WOZ contenga gli otto item PHQ-8 per train/dev. Se gli item non fossero accessibili, le alternative accettabili sono:

- annotare un piccolo campione di turni con un protocollo e annotatori indipendenti;
- limitarsi a concetti linguistici osservabili, evitando pseudo-label cliniche prodotte automaticamente.

Senza item né annotazioni affidabili, questo argomento perde gran parte della sua forza.

## Domanda principale

> Le rappresentazioni di un piccolo language model contengono feature distinte, interpretabili e causalmente rilevanti per diversi domini PHQ-8, oppure il modello usa un unico segnale generico di negatività/distress?

## Domande di ricerca

### RQ1 — Gli item sono predicibili separatamente?

Formulare un problema multi-task: predire contemporaneamente score totale e otto item ordinali. Confrontare un encoder multi-head, un decoder con output strutturato e baseline lessicali. La performance per item stabilisce quali domini hanno un segnale testuale sufficiente.

### RQ2 — Le feature latenti sono specifiche o condivise?

Estrarre direzioni con probing, concept activation vectors, sparse autoencoder o transcoders già disponibili. Misurare quanto le feature associate a un item siano separabili da quelle degli altri item e da concetti generici come sentiment, lunghezza o negazione.

### RQ3 — Le feature sono necessarie e sufficienti per il comportamento del modello?

Eseguire ablation, insertion e patching. Una feature candidata deve modificare selettivamente il logit dell'item previsto più dei logits degli altri item e di un target linguistico di controllo.

### RQ4 — Le rappresentazioni meccanicistiche superano le spiegazioni post-hoc?

Confrontare capacità predittiva degli interventi con attention, gradient saliency e rationale generati. Il criterio è la fedeltà: quale metodo anticipa meglio l'effetto reale di una modifica interna?

## Piano quantitativo

- **Score totale:** MAE, RMSE e Spearman.
- **Item ordinali:** macro-F1, quadratic weighted kappa, ordinal MAE e log-loss.
- **Separazione dei concetti:** AUROC dei probe su split tenuti separati, similarità coseno tra direzioni e matrice di confusione tra item.
- **Fedeltà causale:** delta-logit per item, class flip rate, necessità e sufficienza dopo ablation/insertion.
- **Selettività:** effetto sul target corretto meno effetto medio sugli altri item e su un task di controllo.
- **Stabilità:** overlap delle feature su seed/split e correlazione degli effetti di intervento.
- **Annotazione:** Cohen's kappa o Krippendorff's alpha, con valutatori ciechi rispetto al label e alla predizione.

Usare correzione per confronti multipli quando si testano molti item/feature e intervalli bootstrap gerarchici, campionando prima partecipanti e poi turni.

## Modelli Hugging Face già addestrati, tutti entro 3B

| Modello | Parametri | Ruolo | Nota metodologica |
| --- | ---: | --- | --- |
| [google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b) | circa 2,6B | Decoder principale per J-space, circuiti e interventi | Toolchain meccanicistica più matura; usare output a token singolo per item/classi |
| [Qwen/Qwen3-1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | 1,7B | Replica dei risultati su un'altra architettura | Lens e transcoders pubblici; licenza Apache-2.0 |
| [google/gemma-3-1b-pt](https://huggingface.co/google/gemma-3-1b-pt) | 1B | Alternativa base, utile se si vuole evitare l'effetto dell'instruction tuning | Più leggero ma con percorso interpretativo meno consolidato di Gemma 2 |
| [answerdotai/ModernBERT-large](https://huggingface.co/answerdotai/ModernBERT-large) | 395M | Encoder multi-task per interviste lunghe | 8192 token e 395M parametri; richiede classifier heads supervisionate |
| [mental/mental-roberta-base](https://huggingface.co/mental/mental-roberta-base) | circa 125M | Baseline di pretraining nel dominio mental-health | Pretraining su Reddit, quindi utile anche per misurare il domain mismatch |
| [rafalposwiata/deproberta-large-v1](https://huggingface.co/rafalposwiata/deproberta-large-v1) | circa 0,4B | Baseline domain-adapted | Pre-addestrato su post depressivi Reddit; non è un predittore PHQ-8 e può enfatizzare keyword social |

Per la tesi principale sono preferibili modelli generali con target DAIC ben controllato. I modelli addestrati su social media servono come baseline di dominio, non come ground truth né come scorciatoia per evitare la validazione.

## Esperimento minimo

1. Scegliere 2–3 item con sufficiente variabilità e plausibile segnale testuale, oltre allo score totale.
2. Addestrare un baseline ModernBERT multi-head e valutare un decoder congelato con prompt strutturato.
3. Estrarre feature candidate su un campione predefinito.
4. Far annotare i massimi esempi di attivazione da due valutatori ciechi.
5. Testare causalmente un numero limitato di feature per item.

## Contributo atteso

- passaggio da un singolo score opaco a un profilo di concetti ispezionabile;
- protocollo per separare feature specifiche di item da sentiment/distress generico;
- valutazione causale e non soltanto correlazionale delle spiegazioni interne.

## Rischi e contenimento

- **Label a livello di sessione:** non attribuire automaticamente un item a uno specifico turno.
- **Correlazione tra item:** usare modelli ordinali/multi-task e riportare la matrice di correlazione dei target.
- **Circolarità nell'annotazione:** annotatori ciechi e ontologia fissata prima di osservare i risultati.
- **Numero eccessivo di test:** preregistrare pochi item e poche famiglie di feature.

## Fattibilità

**Media.** Scientificamente forte, ma dipende dalla disponibilità degli item e da una fase di annotazione. È più rischiosa della proposta sugli shortcut, ma può produrre un contributo più originale.

## Risorse

- [Jacobian Lens pubblici](https://huggingface.co/neuronpedia/jacobian-lens)
- [Circuit Tracer](https://github.com/decoderesearch/circuit-tracer)
- [Dati e target DAIC descritti localmente](../daic-features-purpose-models.md)

