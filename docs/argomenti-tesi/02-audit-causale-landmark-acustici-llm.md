# Audit causale dell'integrazione testo-voce in piccoli LLM mediante acoustic landmarks

## Titolo provvisorio

*Are acoustic landmarks causally used by small language models for interview-based PHQ-8 screening?*

## Idea in breve

Un piccolo LLM open-weight (entro 3B parametri) riceve le risposte del partecipante e una sequenza di **acoustic landmarks** discreti, derivati dalla sua voce e allineati ai turni. La tesi non assume che il miglioramento audio-testuale sia reale solo perché la metrica aumenta: verifica se il modello usa davvero la sequenza acustica, se tale uso dipende dall'allineamento con il testo e se le spiegazioni individuano componenti causalmente influenti.

Il punto di partenza e' Zhang et al. (EMNLP 2024), che integra acoustic landmarks in Llama 2 da 7B/13B e riporta risultati competitivi su DAIC-WOZ. Quel lavoro non e' pero' un'analisi di fedelta' delle spiegazioni e non rispetta il vincolo di un modello entro 3B. La tesi studia quindi una domanda diversa, su un modello piccolo e ispezionabile: **il guadagno attribuito alla voce e' un uso causale del segnale acustico o un effetto di rappresentazione, lunghezza, training o correlazioni spurie?**

Il task resta sperimentale: classificazione `PHQ8 < 10` contro `PHQ8 >= 10` e, come secondario, regressione del punteggio. Non e' diagnosi clinica.

## Domanda principale

> In un piccolo LLM, gli acoustic landmarks aggiungono informazione causale oltre alle risposte testuali nella stima del PHQ-8, e i metodi di explainability prevedono gli effetti degli interventi su tale informazione?

## Perche' e' una proposta piu' solida della sola ricerca di shortcut

Qui il fenomeno da testare e' costruito attorno a input che hanno una corrispondenza diretta: testo del partecipante e traccia acustica dello stesso turno. Non bisogna prima trovare una shortcut naturale in una particolare architettura per poter definire gli interventi. Anche un risultato nullo e' sostanziale: puo' mostrare che il modello non usa la voce, che il presunto guadagno non e' robusto, oppure che un metodo di spiegazione non e' fedele.

Il lavoro EMNLP fornisce inoltre un controllo positivo di fattibilita' per l'idea audio-testuale, ma non una garanzia: impiega Llama 2 da 7B/13B, una procedura di fine-tuning diversa e sub-dialoghi aumentati. Le sue conclusioni non si trasferiscono automaticamente a Gemma 2 2B o Qwen3 1.7B.

## Input e modelli

### Input principale

Solo turni del partecipante, per evitare che le domande dell'intervistatore diventino una fonte alternativa di segnale:

```text
[TURN 01]
TEXT: I have been very tired lately.
LANDMARKS: g+ p+ ...

[TURN 02]
TEXT: I do not sleep much.
LANDMARKS: ...
```

Gli acoustic landmarks possono essere token singoli o 2-grammi discreti, estratti dall'audio. Il loro significato e' fonetico/acustico (onset/offset di periodicita', vibrazione glottale, fricazione ecc.), non un'etichetta clinica. La sequenza deve essere ottenuta esclusivamente dall'audio e i nuovi token vanno aggiunti al vocabolario in modo tracciabile.

### Condizioni sperimentali

| Condizione | Scopo |
| --- | --- |
| `text-only` | baseline: che cosa si ottiene dalle risposte? |
| `landmark-only` | i landmark da soli contengono segnale? |
| `aligned text+landmarks` | il modello puo' integrare le due modalita' allineate? |
| `shuffled landmarks` | stesso numero e distribuzione di token, ma ordine/allineamento rotti |
| `matched swapped landmarks` | landmark di un altro partecipante abbinato per durata/qualita'; test esplicito di dipendenza dalla voce |
| `masked landmarks` | ablazione della modalita' con maschera di uguale lunghezza |

Le condizioni controfattuali non affermano che la nuova intervista sia clinicamente realistica: sono interventi controllati sul modello. Tutte le trasformazioni vengono applicate dopo lo split, senza spostare materiale tra partecipanti.

### Modello sotto audit

Gemma 2 2B oppure Qwen3 1.7B sono candidati entro il vincolo. Il decoder produce due token-etichetta fissi; la quantita' tracciata e' `logit(PHQ8>=10) - logit(PHQ8<10)`. Una baseline TF-IDF/ModernBERT e una baseline audio regolarizzata sono obbligatorie: l'LLM non deve essere usato solo perche' e' piu' complesso.

Un adattamento LoRA e' ammissibile se necessario per il task, ma gli strumenti pre-addestrati di interpretabilita' non vanno assunti automaticamente validi sul checkpoint adattato. La validazione causale avviene sempre sul modello effettivamente valutato.

## Domande di ricerca

### RQ1 — Il piccolo LLM trae un guadagno stabile dalla voce?

Confrontare `text-only`, `landmark-only` e `aligned text+landmarks` su DAIC-WOZ ed E-DAIC, con split per partecipante e intervalli bootstrap. Il valore multimodale e' il miglioramento rispetto al migliore sistema unimodale, non una sola F1 favorevole.

### RQ2 — Il modello usa l'allineamento audio-testuale o un artefatto della sequenza?

Confrontare l'input allineato con sequenze permutate, mascherate e scambiate. Se l'effetto sparisce con la permutazione o con lo scambio, pur mantenendo lunghezza e distribuzione simili, c'e' evidenza che la modalita' acustica contribuisca alla decisione. Se rimane invariato, non si puo' attribuire il risultato al contenuto acustico.

### RQ3 — Le spiegazioni sono fedeli all'uso effettivo dei landmark?

Confrontare almeno tre famiglie di spiegazioni:

1. **post-hoc sugli input:** integrated gradients e leave-one-turn/leave-one-landmark-out;
2. **rappresentazionali:** probe lineari per distinguere input allineati e non allineati, logit lens/Jacobian lens solo se compatibile con il checkpoint;
3. **causali interni:** activation patching tra la versione allineata e quella permutata, ablation di head/MLP o di feature candidate, con controlli casuali matched per layer e budget.

Il criterio non e' la plausibilita' di una visualizzazione: una classifica di token o componenti e' utile solo se anticipa `delta-logit`, class flip e specificita' osservati dopo l'intervento.

## Disegno dati: DAIC-WOZ ed E-DAIC

- Usare soltanto voce e trascrizioni del partecipante; mascherare frame scrubbed e possibili segmenti di bleed-over.
- Verificare la versione effettiva di E-DAIC: i transcript dell'intervistatore non sono completi, ma qui non sono richiesti.
- Correggere/annotare eventuali incoerenze fra score PHQ-8 e label binaria prima di creare gli split, seguendo una regola documentata.
- E-DAIC comprende DAIC-WOZ: i due non sono quindi train/test indipendenti. L'uso consigliato e' E-DAIC come corpus principale e DAIC-WOZ solo come replica di confrontabilita' sulle sessioni comuni o analisi di sensibilita' alla release; per un vero trasferimento occorre un terzo corpus non sovrapposto.

## Metriche e decisioni

| Evidenza | Misura |
| --- | --- |
| Predizione | macro-F1, balanced accuracy, AUROC/AUPRC, Brier score |
| Valore della modalita' | differenza rispetto al migliore unimodale, con bootstrap appaiato |
| Sensibilita' ai controfattuali | `delta-logit`, prediction agreement, class-flip rate |
| Fedelta' delle spiegazioni | `effect@k`, `random-gap@k`, correlazione score-effetto, specificita' |
| Trasferimento | stessa batteria di interventi sull'altro corpus, senza scegliere nuovamente le soglie |

## Stack tecnologico e di explainability

Lo stack e' intenzionalmente separato in quattro livelli. I landmark non sono una spiegazione: sono la rappresentazione acustica su cui l'audit puo' intervenire in modo controllato.

| Livello | Tecnologie / tecniche | Cosa si valuta |
| --- | --- | --- |
| Dati e allineamento | E-DAIC/DAIC-WOZ; Python, `pandas`, `librosa`/`torchaudio`; diarizzazione gia' disponibile o verificata; forced alignment testo-audio (es. Montreal Forced Aligner) | qualita', durata e corrispondenza fra ciascun turno e la sua traccia vocale; gli errori di alignment diventano una variabile di audit |
| Rappresentazione e baseline | estrattore di acoustic landmarks riproducibile; TF-IDF + regressione logistica per testo; classificatore audio regolarizzato e fusione tardiva; PyTorch e Hugging Face Transformers | se il segnale nasce da testo, da landmark o dal loro incontro, prima di attribuirlo al piccolo LLM |
| Explainability post-hoc | Captum/integrated gradients, gradient x input, leave-one-turn-out, leave-one-landmark-group-out e permutation importance | ranking di turni, gruppi di landmark e posizioni candidate; baseline di plausibilita', non prova causale |
| Audit causale e meccanicistico | mask, shuffle e swap matched per durata/qualita'; hook sulle attivazioni con PyTorch/NNsight o equivalente; activation/path patching; ablation di head, MLP o feature; probe lineari; Circuit Tracer/Jacobian Lens **solo se compatibili** con il checkpoint effettivo | se il candidato predice `delta-logit`, class flip e specificita' meglio di componenti casuali e delle attribuzioni post-hoc |
| Statistica e riproducibilita' | split per partecipante, configurazioni versionate, seed fissati; bootstrap appaiato e test di permutazione; report aggregati senza audio o transcript riconoscibili | incertezza dell'effetto e possibilita' di replicare l'audit senza esporre dati sensibili |

Il nucleo sufficiente per la tesi e': baseline unimodali e fusione tardiva, tre controfattuali (`mask`, `shuffle`, `swap`), integrated gradients + perturbation importance, quindi patching/ablation con set casuali matched. Circuit Tracer, Jacobian Lens e sparse autoencoder sono estensioni, non prerequisiti.

## Esiti interpretabili

| Risultato | Claim consentito |
| --- | --- |
| Testo+landmark migliora e i landmark allineati superano i controlli | Il piccolo LLM usa informazione acustica associata al testo, nel corpus valutato |
| Il guadagno resta dopo permutazione/scambio | Non e' dimostrato che il guadagno sia dovuto all'allineamento o al contenuto vocale |
| Il modello non migliora sul testo | I landmark non aggiungono valore nel setup scelto; non si conclude che la voce sia irrilevante in generale |
| Le attribuzioni selezionano elementi senza effetto causale | Le spiegazioni considerate non sono fedeli, anche se paiono clinicamente plausibili |
| Il patching supera post-hoc e controlli random | Il metodo meccanicistico localizza componenti causalmente influenti per l'integrazione audio-testuale nel modello studiato |

## Esperimento minimo realizzabile

1. Baseline testo, landmark e fusione tardiva semplice.
2. Gemma 2 2B o Qwen3 1.7B con `text-only` e `text+landmarks`.
3. Permutazione, mascheramento e scambio matched della sequenza landmark.
4. Integrated gradients + perturbation importance sugli input.
5. Activation patching e ablation sulle prime componenti interne, confrontate con controlli random.
6. Replica dell'effetto principale su E-DAIC o DAIC-WOZ, a seconda del corpus scelto per lo sviluppo.

## Rischi e confini

- Il corpus e' piccolo e le label sono session-level: evitare claim su singole parole o singoli frame come "sintomi".
- Il lavoro di Zhang et al. usa modelli superiori a 3B e sub-dialoghi aumentati; non va riprodotto come se fosse direttamente comparabile. In particolare, va separata con attenzione ogni fase che usa informazioni sul label dal training della rappresentazione audio.
- Questa proposta spiega landmark tokenizzati, non l'intera waveform. Un encoder audio end-to-end sarebbe un'estensione distinta e molto piu' costosa da interpretare.
- Il risultato e' un audit di un modello sperimentale di stima PHQ-8, non validazione diagnostica.

## Casi d'uso e decisioni a cui puo' servire

L'utilita' non e' stimare una diagnosi dalla voce, ma verificare che un sistema sperimentale non attribuisca alla componente vocale un valore che non possiede.

| Evidenza dell'audit | Decisione o caso d'uso appropriato |
| --- | --- |
| I landmark allineati aggiungono valore e l'effetto supera i controlli | Progettare uno strumento di **ricerca** multimodale per il pre-screening remoto, mantenendo una modalita' text-only di fallback e riportando esplicitamente la dipendenza dalla qualita' audio |
| Il modello cambia decisione solo con landmark allineati in pochi turni | Audit di robustezza di un assistente conversazionale: definire quali segmenti richiedono una registrazione sufficiente e quali no, senza trasformare quei segmenti in indicatori clinici individuali |
| Il guadagno sopravvive a `shuffle` o `swap` | Rilevare un possibile artefatto di durata, canale, speaker o preprocessing; rimuovere/ribilanciare la sorgente prima di considerare la voce una modalita' utile |
| Le attribuzioni post-hoc non anticipano patching/ablation | Scegliere metriche di validazione e model card che non presentino heatmap o token salienti come spiegazioni affidabili |
| Il protocollo e' stabile su una nuova raccolta audio | Valutare il riuso per studi longitudinali o telemedicina di ricerca, ma ripetendo l'audit per microfono, ambiente, popolazione e protocollo differenti |

## Riferimenti chiave

- [Zhang et al. (EMNLP 2024), *When LLMs Meet Acoustic Landmarks*](https://aclanthology.org/2024.emnlp-main.8/) — precedente diretto audio-testuale su DAIC-WOZ; Llama 2 7B/13B, non explainability causale.
- [Gratch et al. (2014), *The Distress Analysis Interview Corpus*](https://aclanthology.org/L14-1421/) — origine di DAIC e delle sue modalita'.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching*](https://arxiv.org/abs/2309.16042) — cautela metodologica per metriche, corruption e controlli nel patching.
- [Lyu, Apidianaki e Callison-Burch (2024), *Towards Faithful Model Explanation in NLP*](https://direct.mit.edu/coli/article/50/2/657/119158/Towards-Faithful-Model-Explanation-in-NLP-A-Survey) — distinzione fra attribuzione plausibile e fedele.
