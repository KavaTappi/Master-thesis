# 02 — Causal stress-testing dell'integrazione testo-voce mediante acoustic landmarks

## Titolo provvisorio

*Do small language models use acoustic landmarks for the right reasons? A causal stress test of text-speech integration*

## Idea e perimetro

La tesi non propone un nuovo metodo per inserire gli **acoustic landmarks** in un LLM. Questo passaggio è già stato studiato da [Zhang et al. (EMNLP 2024)](https://aclanthology.org/2024.emnlp-main.8/), che estraggono landmark dalla voce, li convertono in token discreti e addestrano Llama 2 per la depression detection su DAIC-WOZ.

La proposta studia il passaggio successivo, ancora necessario per interpretare quel risultato:

> **Quando i landmark modificano la predizione, l'effetto dipende dal loro contenuto acustico e dal corretto abbinamento con il testo, oppure da lunghezza, ordine dei token, identità del parlante, preprocessing o supervisione introdotta durante il training?**

Il contributo è quindi un **audit causale e di fedeltà delle spiegazioni**. Lo stesso transcript viene valutato con versioni controllate dei landmark; successivamente si verifica se le spiegazioni individuano token, turni e siti interni che anticipano gli effetti di tali interventi.

Il task applicativo principale è la classificazione binaria session-level `PHQ-8 < 10` contro `PHQ-8 >= 10`. La regressione del punteggio è secondaria. Il sistema è un modello sperimentale di ricerca e non uno strumento diagnostico.

## Che cosa aggiunge rispetto a Zhang et al.

Zhang et al. mostrano che una rappresentazione discreta della voce può essere incorporata in Llama 2 e confrontano `text-only`, `landmark-only` e `text+landmark`. Nella sezione 5 analizzano soprattutto il training: misurano la grandezza media assoluta delle matrici LoRA, osservano contributi maggiori nelle componenti feed-forward e nei primi layer, confrontano LoRA e P-tuning e disattivano alcune matrici con contributo basso.

Questa analisi non separa però diverse possibili spiegazioni del guadagno multimodale. In particolare, non verifica se il modello richieda:

- i landmark della persona corretta;
- il corretto ordine temporale dei landmark;
- l'allineamento tra un turno testuale e il segmento vocale corrispondente;
- il contenuto dei landmark, anziché il semplice numero di token aggiunti;
- il **label hint** usato da Zhang et al. durante il cross-modal instruction fine-tuning, che specifica se l'esempio proviene da una persona classificata come depressa o sana.

Il label hint non costituisce automaticamente leakage se viene applicato soltanto ai dati di training dopo uno split corretto. È però una forma aggiuntiva di supervisione che può modificare ciò che il modello impara a chiamare “informazione dei landmark”; per questo viene isolata sperimentalmente.

La tesi aggiunge quattro elementi:

1. **Decomposizione causale del guadagno multimodale.** Interventi appaiati mantengono fisso il testo e modificano una proprietà dei landmark per volta.
2. **Audit del procedimento di training.** Si confrontano configurazioni con e senza label hint, senza chiamare automaticamente “informazione vocale” ciò che potrebbe dipendere dalla supervisione della classe.
3. **Valutazione della fedeltà delle spiegazioni.** Le attribuzioni devono prevedere gli effetti di rimozione o sostituzione, non soltanto produrre una heatmap plausibile.
4. **Localizzazione interna limitata.** L'activation patching verifica se alcune attivazioni mediano in modo riproducibile la differenza tra input allineati e controfattuali.

Usare un LLM sotto i 3B rende l'audit più accessibile, ma **non è da solo la novità scientifica**. La novità è il protocollo che distingue uso del contenuto acustico, allineamento e artefatti.

## Domanda principale e domande di ricerca

> **Gli acoustic landmarks influenzano la decisione di un piccolo LLM attraverso informazione acustica specifica e correttamente allineata al testo, e le tecniche di explainability identificano fedelmente tale influenza?**

### RQ1 — Valore oltre il testo

Un modello `text+landmarks` supera in modo stabile il migliore sistema unimodale e una fusione tardiva semplice, con split e incertezza calcolati a livello di partecipante?

### RQ2 — Contenuto, ordine, allineamento o identità

Quanto cambia la decisione quando si conserva il transcript ma si mascherano, permutano, disallineano o scambiano i landmark? Le diverse perturbazioni permettono di distinguere:

- presenza generica di token acustici;
- distribuzione globale dei landmark;
- ordine temporale;
- allineamento locale testo-voce;
- caratteristiche globali del parlante o del canale.

### RQ3 — Dipendenza dal label hint

La prestazione e la sensibilità ai landmark persistono quando il pretraining cross-modale non contiene l'etichetta `depressed/healthy` nel prompt? Il label hint modifica soltanto l'ottimizzazione oppure anche il tipo di informazione usato dal classificatore?

### RQ4 — Fedeltà delle spiegazioni

Integrated Gradients e perturbation importance selezionano gruppi di landmark e turni la cui sostituzione produce effetti maggiori di controlli casuali matched? I siti interni selezionati mediano parte dell'effetto tra la versione allineata e quella controfattuale?

RQ1 è un prerequisito descrittivo; RQ2–RQ4 costituiscono il contributo rispetto al lavoro precedente. Un risultato nullo è informativo: può mostrare che il guadagno non richiede l'allineamento, che dipende dal label hint oppure che le spiegazioni non sono fedeli.

## Dati e unità di analisi

### Corpus

- **DAIC-WOZ** è il riferimento per la confrontabilità con Zhang et al.
- **E-DAIC** può essere usato come release principale o analisi di sensibilità, ma contiene sessioni di DAIC-WOZ e non costituisce automaticamente un test esterno indipendente.
- Un eventuale terzo corpus audio-testuale con label compatibili è un'estensione, non un requisito.

Si utilizzano soltanto i turni del partecipante. Audio, transcript e label devono essere accessibili secondo licenza. Frame scrubbed, bleed-over dell'intervistatore, segmenti non vocali e qualità del canale vengono registrati e controllati. L'unità statistica è il **partecipante**, non il sub-dialogo aumentato.

### Acoustic landmarks

Gli acoustic landmarks sono eventi discreti estratti dalla waveform: inizio/fine di vibrazione glottale, periodicità, fricazione, rumore turbolento e altri cambiamenti acustici bruschi. Non sono etichette cliniche.

Per mantenere il confronto con Zhang et al., il piano principale usa lo stesso insieme documentato di landmark e confronta token singoli e bigrammi soltanto sul development. L'estrattore, le soglie, la frequenza di campionamento e la politica di smoothing devono essere versionati. I landmark vengono allineati ai turni mediante timestamp dell'audio e del transcript; l'errore di allineamento viene misurato, non ignorato.

Esempio di input:

```text
[TURN 01]
TEXT: I have been very tired lately.
LANDMARKS: g+ p+ p- ...

[TURN 02]
TEXT: I do not sleep much.
LANDMARKS: ...

TASK: A = PHQ8 < 10; B = PHQ8 >= 10.
ANSWER:
```

## Modello e rappresentazione

Il modello rimane un decoder testuale: l'audio viene trasformato prima in landmark discreti. La tesi non interpreta direttamente la waveform e non richiede un Audio LLM nativo.

| Priorità | Modello | Uso e cautela |
| --- | --- | --- |
| 1 | [Qwen3 1.7B Base](https://huggingface.co/Qwen/Qwen3-1.7B-Base) | candidato principale per LoRA e contesto lungo; richiede addestramento degli embedding se si aggiungono token dedicati |
| 2 | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) | replica o fallback; contesto più corto e licenza Gemma |
| 3 | [Gemma 3 1B base](https://huggingface.co/google/gemma-3-1b-pt) | pilota leggero; può non superare le baseline e va sottoposto allo stesso gate |
| Controlli | TF-IDF/lineare sul testo; classificatore regolarizzato sui landmark; fusione tardiva | stabiliscono se la complessità del decoder è giustificata |

Il piano minimo usa **un solo LLM**. I nuovi token rendono il checkpoint diverso dal modello base: Jacobian Lens, SAE o transcoders pre-fittati non vengono considerati automaticamente validi. Il nucleo usa attribution sugli input, hook standard, patching e ablation sul checkpoint finale effettivamente valutato.

## Condizioni controfattuali

Ogni intervento parte dallo stesso transcript e modifica soltanto la rappresentazione acustica dopo lo split dei partecipanti.

| Condizione | Proprietà preservata | Proprietà distrutta | Domanda a cui risponde |
| --- | --- | --- | --- |
| `aligned` | testo, speaker, ordine e timing | nessuna | riferimento multimodale |
| `masked` | testo e lunghezza del blocco, se la maschera è length-matched | contenuto dei landmark | i landmark hanno un effetto complessivo? |
| `random-token control` | numero di token, struttura del prompt e frequenze marginali campionate dal train | identità e sequenza reale dei landmark | basta aggiungere token con statistiche plausibili? |
| `within-turn shuffled` | speaker, turno e istogramma dei token | ordine locale | contano le sequenze o solo le frequenze? |
| `turn-shifted within speaker` | testo, speaker e statistiche globali | allineamento con il turno corretto | conta il rapporto locale tra cosa e come viene detto? |
| `matched speaker swap` | testo, durata, qualità/canale e lunghezza approssimativa | voce associata al partecipante | il modello dipende dai landmark della persona corretta? |

Lo `shuffle` non basta da solo: può produrre una sequenza fuori distribuzione. Per questo viene affiancato allo spostamento fra turni dello stesso speaker e allo swap fra partecipanti matched. Lo swap viene stratificato in donatori della **stessa classe**, per isolare l'associazione partecipante-voce, e della **classe opposta**, come test secondario del segnale acustico associato al target. I criteri di matching — durata, numero di landmark, qualità audio ed eventuale sito di registrazione — vengono fissati sul development; la classe del donatore non viene usata per il training.

## Esperimenti minimi

### Esperimento 0 — Gate tecnico e baseline

**Obiettivo.** Stabilire che la pipeline audio sia riproducibile e che il task non sia dominato da errori di preprocessing.

**Configurazioni.**

1. `text-only` con TF-IDF + regressione logistica e con il piccolo LLM;
2. `landmark-only` con classificatore lineare/regolarizzato;
3. fusione tardiva delle migliori baseline unimodali;
4. piccolo LLM `text+landmarks`.

**Controlli.** Split per partecipante; preprocessing fittato solo sul train; nessuna selezione su test; confronto sia con singolo modello sia, se replicato, con l'ensemble di Zhang senza confondere i due risultati.

**Gate.** La tesi procede comunque se il multimodale non migliora: in quel caso l'audit stabilisce se il modello ignora i landmark o se ne è sensibile in modo non utile. Le analisi interne approfondite vengono eseguite solo se esiste un effetto comportamentale misurabile.

### Esperimento 1 — Decomposizione causale dell'informazione acustica

**Obiettivo.** Distinguere contenuto, ordine, allineamento e speaker.

Per ogni sessione del test si calcolano il logit gap e un margine orientato verso il label osservato:

```text
g(x) = logit(B | x) - logit(A | x)
s_y = +1 se y = B, altrimenti -1
m_y(x) = s_y * g(x)
damage_c(x) = m_y(x_aligned) - m_y(x_c)
```

dove `c` è una delle condizioni controfattuali. Un `damage_c` positivo indica che il controfattuale riduce l'evidenza per la classe osservata. Si riportano anche il `delta-logit` non orientato per singola coppia, prediction agreement, class-flip rate e perdita di performance aggregata. Il test statistico primario confronta gli effetti appaiati per partecipante con intervalli bootstrap.

**Interpretazione.**

- l'allineato ha margine orientato migliore di random/masked ma non dello shuffle: evidenza per presenza o frequenza dei token, non per il loro ordine;
- il turn-shift riduce il margine: evidenza di uso dell'allineamento locale;
- lo speaker-swap riduce il margine: evidenza di dipendenza da informazione associata allo speaker;
- nessuna differenza: il guadagno non è attribuibile al contenuto/allineamento nel protocollo adottato.

### Esperimento 2 — Audit del label hint

**Obiettivo.** Verificare se la procedura di Zhang orienti l'apprendimento dei landmark attraverso l'etichetta di classe.

Si addestrano due configurazioni identiche per dati, seed, budget e architettura:

1. `hint-free`: il pretraining cross-modale associa testo e landmark senza dichiarare la classe;
2. `label-hint`: replica il prompt che segnala `depressed` o `healthy` nel training cross-modale.

Per entrambe si ripetono baseline e controfattuali dell'Esperimento 1. Il confronto importante non è soltanto la F1: si misura se il label hint aumenta la sensibilità a landmark reali oppure anche a landmark disallineati e controlli artificiali.

Se due training completi sono troppo costosi, l'esperimento usa un singolo seed per screening e replica solo il contrasto principale con più seed. Il test finale rimane separato dalla scelta della configurazione.

### Esperimento 3 — Fedeltà delle spiegazioni sull'input

**Obiettivo.** Verificare se un ranking di importanza anticipi gli effetti degli interventi.

Tecniche minime:

- Integrated Gradients sul logit gap per token o gruppi di landmark;
- leave-one-turn-out;
- leave-one-landmark-group-out, raggruppando eventi contigui o famiglie fonetiche prespecificate.

Procedura:

1. si calcola il ranking senza osservare gli effetti di rimozione del test;
2. si sostituiscono i top-`k` gruppi con controlli matched;
3. si ripete su gruppi casuali con stesso numero, posizione e frequenza;
4. si confrontano effetto top-`k`, curva di deletion, correlazione score-effetto e class flip.

Integrated Gradients non è considerato causale da solo. La spiegazione è utile soltanto se i gruppi selezionati producono effetti maggiori e più specifici dei controlli.

### Esperimento 4 — Patching interno limitato

**Obiettivo.** Superare l'analisi della sola grandezza dei pesi LoRA e localizzare dove viene mediata la differenza tra input allineato e controfattuale.

Si selezionano in anticipo due contrasti, preferibilmente `aligned ↔ turn-shifted` e `aligned ↔ matched speaker swap`. Sullo stesso checkpoint si sostituisce il residual stream della corsa allineata con quello della corsa controfattuale:

- obbligatoriamente alla posizione di output finale, per tutti i layer;
- solo come estensione, nelle posizioni dei turni più influenti, quando source e target hanno una corrispondenza posizionale definita senza forzarla.

Si confrontano siti candidati e siti casuali matched per layer e posizione. La quantità primaria è la quota dell'effetto di input trasferita dal patch:

```text
recovery(s) = [g(x patched at s) - g(x_aligned)]
              / [g(x_counterfactual) - g(x_aligned)]
```

Il patching dell'intero residual stream localizza una differenza causalmente influente ma non assegna automaticamente un significato semantico preciso. Head ablation, path patching, Jacobian Lens e Circuit Tracer sono estensioni successive, non requisiti.

Il recovery normalizzato viene calcolato solo per coppie il cui effetto totale supera una soglia fissata sul development; vicino a un denominatore nullo è instabile. Il `delta-logit` non normalizzato viene sempre riportato.

## Protocollo minimo realizzabile

Il minimo che distingue chiaramente la tesi dal paper comprende:

1. un estrattore di landmark verificato e uno split per partecipante;
2. quattro baseline: testo, landmark, fusione tardiva e piccolo LLM multimodale;
3. quattro controfattuali obbligatori: `masked`, `random-token`, `turn-shifted` e `matched speaker swap`; lo shuffle è un controllo aggiuntivo economico;
4. confronto `hint-free` contro `label-hint`;
5. Integrated Gradients e perturbation importance con top-`k` contro random matched;
6. activation patching su due contrasti, posizione finale e sweep dei layer;
7. bootstrap appaiato a livello di partecipante e report degli effetti nulli.

Non sono necessari nel nucleo: un secondo LLM, un Audio LLM nativo, Circuit Tracer, Jacobian Lens, SAE addestrati da zero, regressione continua o un terzo corpus.

## Metriche e criteri di successo

| Livello | Metriche |
| --- | --- |
| Predizione | macro-F1, balanced accuracy, AUROC/AUPRC, Brier score |
| Valore multimodale | differenza rispetto al migliore unimodale e alla fusione tardiva, con bootstrap appaiato |
| Controfattuali | `delta-logit`, agreement, class-flip rate, perdita di performance |
| Fedeltà attribution | effect@`k`, random-gap@`k`, deletion curve, correlazione score-effetto |
| Patching | recovery dell'effetto, accuratezza del segno, specificità contro siti random |
| Robustezza | replica fra seed, stabilità del matching e sensibilità alle soglie dell'estrattore |

Il successo non richiede che `text+landmarks` vinca. Sono esiti scientificamente interpretabili anche:

- il miglioramento scompare quando si rimuove il label hint;
- il modello reagisce allo swap ma non al disallineamento locale;
- il multimodale migliora, ma le spiegazioni non predicono gli interventi;
- il piccolo LLM non utilizza i landmark meglio di una fusione tardiva semplice.

## Stack tecnologico

| Livello | Tecnologie e tecniche |
| --- | --- |
| Audio e dati | Python, `numpy`, `scipy`, `librosa`/`torchaudio`, estrattore landmark riproducibile, timestamp e forced alignment quando necessario |
| Modelli | PyTorch, Hugging Face Transformers, PEFT/LoRA, scikit-learn per baseline |
| Attribution | Captum per Integrated Gradients; occlusion e perturbation importance implementate sugli stessi gruppi |
| Interventi interni | hook PyTorch, TransformerLens o NNsight se compatibili; activation patching sul checkpoint finale |
| Analisi | `pandas`, bootstrap appaiato, test di permutazione, configurazioni e seed versionati |



## Claim consentiti

| Risultato | Conclusione proporzionata |
| --- | --- |
| L'allineato supera mask, turn-shift e swap | il modello usa informazione acustica associata al testo e allo speaker nel corpus studiato |
| Solo lo swap produce un effetto | il modello usa proprietà globali associate allo speaker, non è dimostrata integrazione locale testo-voce |
| Shuffle e shift non cambiano la decisione | non è dimostrato l'uso dell'ordine o dell'allineamento |
| Il vantaggio compare soltanto con label hint | il risultato dipende dalla procedura supervisionata e non prova un apprendimento generale dei landmark |
| Attribution top-`k` supera random matched | il metodo anticipa quali gruppi di input influenzano maggiormente la decisione |
| Patching recupera parte dell'effetto | il sito media una differenza interna tra le due condizioni; non identifica da solo un concetto clinico |

## Riferimenti essenziali

- [Zhang et al. (2024), *When LLMs Meet Acoustic Landmarks*](https://aclanthology.org/2024.emnlp-main.8/) — metodo di partenza, DAIC-WOZ, integrazione landmark-LLM e analisi LoRA.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching*](https://arxiv.org/abs/2309.16042) — sensibilità del patching a metriche e costruzione dei controfattuali.
- [Sundararajan et al. (2017), *Axiomatic Attribution for Deep Networks*](https://arxiv.org/abs/1703.01365) — Integrated Gradients.
- [Lyu et al. (2024), *Towards Faithful Model Explanation in NLP*](https://aclanthology.org/2024.cl-2.6/) — distinzione tra plausibilità e fedeltà delle spiegazioni.
