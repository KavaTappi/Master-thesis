---
title: Piani di tesi — proposte 00 e 01
permalink: /
description: Due proposte per la discussione con il relatore, dalla scoperta di circuiti guidata da lessici all’audit degli indizi d’intervista.
---

<a id="english"></a>

**Language / Lingua:** [English](#english) · [Italiano](#italiano)

# Thesis plans for discussion with the supervisor

## 1. Introduction

### 1.1 Proposed title

**The value of lexical guidance in LLM circuit discovery: cost, faithfulness, and generalization on DAIC-WOZ interviews.**

### 1.2 Main research question

Given the same resources and circuit size, does a lexicon help identify circuits that more faithfully reproduce an LLM’s PHQ-8 predictions, while preserving this faithfulness on unseen participants and paraphrases?
(Compared with global, random, and other forms of guidance.)

### 1.3 Objectives and expected contribution

- Design a lexicon-based prioritization strategy for circuit search and compare it with global, random, and salience-based guidance.
- **Is a lexicon a better guide for starting the search for circuits?**
- Measure the added value of the lexicon in terms of cost, faithfulness, and transfer in the discovered circuits, compared with those examined initially.
- Produce a reproducible protocol that also documents conditions under which the guidance is ineffective, as well as its limitations.

## 2. Testable hypotheses and research questions

| Question/Hypothesis | Main area | Experiment | Positive outcome | Negative outcome |
|---|---|---|---|---|
| **H0 — Can the system be studied with the available resources?** | Feasibility | E0–E1: data, lexicon, model, and intervention checks. | Usable inputs and categories. | Not feasible. |
| **H1 — Does lexical guidance improve the cost–faithfulness trade-off?** | Efficiency | E2: comparison of error curves with circuit size controlled; E5: robustness. For example, set a time budget and check whether circuits discovered using lexical guidance have lower error than those discovered using other selection rationales. The focus is therefore primarily on performance. | Lower error at the same cost, or lower cost at the same tolerance, including screening. | No practical advantage, or additional costs that exceed the savings. |
| **H2 — Does the advantage depend on lexical information?** | Specificity | E0, E2, and E5: comparison with **matched random masks** (randomly selected tokens) and **token salience** (tokens to which the output is most sensitive), with comparable coverage. Essentially, this assesses how useful the lexicon is relative to the other baselines. | Lexical guidance outperforms the controls according to a predefined measure. | The controls reproduce the advantage, or it is explained by the frequency, position, and number of selected tokens. This is still useful information. |
| **H3 — Do the circuits remain faithful beyond the discovery examples?** | Generalization | E3: unseen participants and paraphrases; E6: optional external replication. | The increase in error remains within a tolerance set on the development set, and the advantage is preserved. This would be a strong result: a mechanism that generalizes has been identified. | Faithfulness or the advantage deteriorates, especially beyond the initial vocabulary. |
| **H4 — Does the circuit reproduce the model’s contextual sensitivity?** | Context | E4: negation, temporality, and the person whose experience is being described. | The circuit reproduces changes in the full model’s score. | The circuit loses contextual distinctions present in the model. |

**Interpretation rule:** wide intervals yield an inconclusive result, not evidence of no effect. H1 is the primary hypothesis; H2 and H3 define the scope of its interpretation. H4 is a secondary investigation.

## 3. Technical stack

### 3.1 Dataset

- **DAIC-WOZ:** English transcripts, speaker roles, and the official binary label associated with PHQ-8, verified in the available release. [Official source](https://dcapswoz.ict.usc.edu/).
- Split any training data, discovery data, development data, and final evaluation data by participant. All variants from one person remain in the same split; if the official test set cannot be used, explicitly report that the evaluation is internal.
- The model receives the same dialogue across all search strategies. The lexicon initially marks the participant’s responses.
- If the full context is too costly, use deterministic excerpts distributed throughout the dialogue, selected independently of the lexicon and label. In that case, the explanations concern predictions made on those excerpts.
- Create reviewed paraphrases that avoid the entries used for guidance. Where possible, keep families of expressions separate between discovery and validation.
- Optional external replication on a simple text task or on compatible, disjoint data. E-DAIC requires checks for overlap and protocol compatibility.

### 3.2 Model task and environment

**Main task:** prediction of the binary PHQ-8 class using a small LLM with accessible weights and activations. Compare the base model and, if necessary, a LoRA adaptation trained only on the training split; then freeze the checkpoint, prompt, and threshold. Predictive baselines: majority class and TF-IDF with logistic regression.

Use two class symbols verified to be single tokens. Define the score as the difference between their logits and the probability as a softmax restricted to the two options. This probability is not automatically calibrated. Predictive metrics: macro-F1, AUROC, and log-loss.

**Planned environment:** Python, PyTorch, Transformers, PEFT for LoRA, tools for accessing activations that are compatible with the checkpoint, the [IPE/MIB](https://github.com/nepp1d0/MIB-circuit-track-with-paths) code, scikit-learn, and pandas. Compatibility must be verified during the pilot.

### 3.3 Explainability, lexicon, and comparisons

#### 3.3.1 Lexicon development

Start from a documented English resource, such as selected categories from [NRC EmoLex](https://www.saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm). Record the source, version, categories, terms of use, matching rules, and token offsets. Any extension with expressions related to sleep, energy, or interest must be reviewed by people and developed without using the test set.

Evaluate the matcher’s precision/recall and inter-annotator agreement on an independent sample. The categories are analytical tools: they do not assign diagnoses or PHQ-8 labels to sentences.

#### 3.3.2 Search procedure

1. **EAP-IG screening:** estimate edge importance and, where possible, retain contributions by position. [Hanna et al., 2024](https://arxiv.org/abs/2403.17806).
2. **Lexical guidance:** combine a global ranking with a ranking based on positions covered by the lexicon and their context; select the weighting on the development set. The guidance changes candidate priorities while maintaining exploration beyond the lexicon.
3. **Path interventions:** evaluate candidates using IPE, with a shared maximum compute budget, depth, and continuation rule. [Brunello et al., 2025](https://aclanthology.org/2025.blackboxnlp-1.30/).
4. **Joint validation:** build circuits of comparable sizes and measure their behavior by intervening on contributions from outside the circuits. The effects of individual paths are not automatically additive.
5. **Freezing:** evaluate the circuits on new data without rediscovering them on the test set.

The original model’s input text and weights remain fixed. Runs with interventions are separate experiments: replacing activations changes the internal computation without necessarily changing the input tokens. Reference pairs must be consistent and aligned at the positions where interventions are performed.

#### 3.3.3 Planned comparisons

| Strategy | Role in the comparison |
|---|---|
| EAP-IG with standard edge selection | Baseline to assess whether additional path search is useful. |
| Search with global prioritization | The same proposed procedure, without the lexical contribution. |
| Search with lexical prioritization | The variant to be evaluated. |
| Search with matched random masks | Control for the number, position, frequency, and length of spans. |
| Search guided by token salience | Control to assess the advantage over information derived directly from the model. |
| Original IPE and the position-aware method | Methodological references evaluated on a manageable subset with a comparable protocol. |

[Position-aware Automatic Circuit Discovery](https://aclanthology.org/2025.acl-long.141/) already considers positions and semantic spans. The proposal must demonstrate the incremental value of the lexicon: combining existing techniques with DAIC alone does not guarantee originality. The salience control is not equivalent to reproducing the position-aware method.

### 3.4 Quantitative analyses

**Reference for faithfulness: the full model’s output on the same interview, not the observed PHQ-8.** Let `p_M(x)` denote the original output and `p_C(x; r)` the output obtained from the circuit under a fixed intervention reference:

`E_fid(C) = mean_over_participants [abs(p_M(x) − p_C(x; r))]`

The full graph must have near-zero error. It represents the theoretical maximum in faithfulness, but retains the entire model; therefore, compare circuits of equal size. Add an empty circuit and random subgraphs as controls; report when the reference produces a contrast too small to be informative.

- **Efficiency:** error–GPU-time curve, area over a common interval, and cost to reach a tolerance set on the development set. Include screening, preparation of the guidance, and search; report annotation and final evaluation separately.
- **Faithfulness and compactness:** error in probabilities and logits, class agreement, and number of edges/nodes.
- **Generalization:** increase in error on paraphrases and unseen participants; distinguish instability in the original model from circuit unfaithfulness.
- **Context:** distance between the changes in the model’s and circuit’s scores, `abs(Δs_M − Δs_C)`.
- **Uncertainty:** paired comparisons, participant-level bootstrap, and repeated discovery runs. Set thresholds and minimum differences of interest before testing.

Savings are a hypothesis: changing only the exploration order does not necessarily reduce costs if the same candidates are ultimately examined.

### 3.5 Scope and fallback plan

The core study comprises one model, one task, one lexicon, matched controls, and transfer to paraphrases. If IPE is too costly, switch to EAP-IG edge selection with validation through patching: the question becomes the **size–faithfulness** trade-off, without assuming savings in screening. If the PHQ-8 predictor still cannot be meaningfully studied after limited adaptation, transfer the methodological question to a simpler text task and update the title. Incorrect interventions must be repaired or replaced with validated tools before drawing causal conclusions.

## 4. Planned experiments

| Experiment | Hypothesis/Question addressed |
|---|---|
| **E0 — Data and lexicon preparation:** splits, coverage, agreement, and matcher quality. | H0, H2: reliability of the resource and comparisons. |
| **E1 — Model and intervention pilot:** performance, technical replication, full/empty graph, memory, and runtime. | H0: conditions for proceeding and selection of a feasible scope. |
| **E2 — Search with a controlled budget:** cost–faithfulness curves across strategies and circuit sizes. | H1, H2: efficiency and the specific contribution of the lexicon. |
| **E3 — Transfer to unseen participants and paraphrases:** frozen circuits and distance from the original outputs. | H3: generalization beyond the discovery data and vocabulary. |
| **E4 — Contextual variations:** negation, time, and person, with review of the pairs. | H4: reproduction of the model’s contextual sensitivity. |
| **E5 — Robustness of the result:** repeated runs, alternative references, and coverage/size controls. | H1, H2: verification that the advantage is not an artifact of selection or patching. |
| **E6 — Optional external replication:** a second configuration or compatible task. | H3: scope of the conclusions beyond the first experiment. |

## 5. Titles based on experimental outcomes

| Observed outcomes | Possible final title |
|---|---|
| E2 shows an advantage, E3 confirms its transfer, and E5 supports its robustness. | **Efficient and generalizable LLM circuit discovery through lexical guidance: a study on DAIC-WOZ.** |
| E2 shows an advantage, but E3 reveals deterioration on paraphrases. | **Efficiency and generalization limits of lexical guidance in LLM circuit discovery.** |
| E2 and E5 show negligible benefits over the controls, with sufficiently narrow intervals. | **Value and limitations of lexical guidance in circuit discovery: a controlled evaluation on DAIC-WOZ.** |
| Differences remain uncertain due to variability or limited sample size. | **An experimental evaluation of lexical guidance in LLM circuit search on DAIC-WOZ.** |
| The edge-based fallback plan is adopted, with faithfulness and transfer checks. | **Lexical guidance for circuit selection in LLMs: compactness, faithfulness, and generalization.** |

---

<a id="italiano"></a>

**Language / Lingua:** [English](#english) · [Italiano](#italiano)

# Piani di tesi per la discussione con il relatore

## 1. Introduzione

### 1.1 Titolo proposto

**Il valore della guida lessicale nella scoperta di circuiti degli LLM: costo, fedeltà e generalizzazione su interviste DAIC-WOZ.**

### 1.2 Domanda centrale

A parità di risorse e di dimensione del circuito, un lessico permette di trovare circuiti che riproducono più fedelmente le predizioni di un LLM sul PHQ-8, mantenendo questa fedeltà su partecipanti nuovi e parafrasi?
(rispetto a guide globali, casuali, ...)

### 1.3 Obiettivi e contributo atteso

- Progettare una priorità lessicale per la ricerca di circuiti e confrontarla con guide globali, casuali e basate sulla salienza.
- **Il lessico è una migliore linea guida per iniziare a cercare circuiti?**
- 
- Misurare il valore aggiunto del lessico in termini di costo, fedeltà e trasferimento, nei circuiti trovati, rispetto a quelli visti all'inizio.
- 
- Produrre un protocollo riproducibile che documenti anche condizioni di inefficacia e limiti della guida.


## 2. Ipotesi verificabili e domande di ricerca

| Domanda/Ipotesi | Macro area | Esperimento | Esito positivo | Esito negativo |
|---|---|---|---|---|
| **H0  Il sistema è studiabile con le risorse disponibili?** | Fattibilità | E0–E1: dati, lessico, modello e controlli sugli interventi. | Input e categorie utilizzabili | Nonfattibile  |
| **H1  La guida lessicale migliora il compromesso costo–fedeltà?** | Efficienza | E2: confronto delle curve di errore a dimensione del circuito controllata; E5: robustezza.                                                                                 Per esempio definisco un budget di tempo e guardo se i circuiti creati partendo dal lessico fanno un errore inferiori che partendo da altre razionali, quindi è più una questione di perfomances     | Errore inferiore allo stesso costo, oppure costo inferiore alla stessa tolleranza, includendo lo screening.                                                        | Nessun vantaggio pratico o costo aggiuntivo superiore al risparmio. |
| **H2  Il vantaggio dipende dall’informazione lessicale?** | Specificità            | E0, E2 ed E5: confronto con **maschere casuali abbinate** (token scelti a random) e **salienza del token** (token per il quali l'output è più sensibile), a copertura comparabile.quindi sostanzialmente quanto il lessico è valido rispetto alle eltre baseline           | La guida lessicale supera i controlli secondo una misura prefissata. | Il vantaggio è riprodotto dai controlli o spiegato da frequenza, posizione e numero dei token selezionati.   comunque informazione utile     |
| **H3  I circuiti restano fedeli oltre gli esempi di scoperta?** | Generalizzazione | E3: nuove persone e parafrasi; E6: eventuale replica esterna. | L’incremento di errore resta entro una tolleranza fissata sul development e il vantaggio si conserva.   Molto buono abbiamo trovato un meccanismo valido          | La fedeltà o il vantaggio si deteriorano, soprattutto fuori dal vocabolario iniziale. |
| **H4  Il circuito riproduce la sensibilità contestuale del modello?** | Contesto | E4: negazione, temporalità e persona cui è riferita l’esperienza. | Il circuito riproduce le variazioni dello score del modello completo. | Il circuito perde differenze contestuali presenti nel modello. |

**Regola di interpretazione:** intervalli ampi producono un risultato inconcludente, non una dimostrazione di assenza di effetto. H1 è primaria; H2 e H3 ne delimitano il significato. H4 è un approfondimento secondario.

## 3. Stack tecnico

### 3.1 Dataset

- **DAIC-WOZ:** trascrizioni in inglese, ruoli dei parlanti ed etichetta binaria ufficiale associata al PHQ-8, verificata nella release disponibile. [Fonte ufficiale](https://dcapswoz.ict.usc.edu/).
- Separare addestramento eventuale, scoperta, development e valutazione finale per partecipante. Tutte le varianti di una persona rimangono nello stesso split; se il test ufficiale non è utilizzabile, dichiarare la valutazione interna.
- Il modello riceve lo stesso dialogo in tutte le strategie di ricerca. Il lessico marca inizialmente le risposte del partecipante.
- Se il contesto completo è troppo costoso, usare estratti deterministici distribuiti nel dialogo, indipendenti da lessico ed etichetta. In tal caso si spiegano le predizioni sugli estratti.
- Costruire parafrasi revisionate che evitino le voci usate nella guida. Separare, ove possibile, famiglie di espressioni fra scoperta e verifica.
- Replica esterna facoltativa su un task testuale semplice o su dati compatibili e disgiunti. E-DAIC richiede controlli di sovrapposizione e protocollo.

### 3.2 Task del modello e ambiente

**Task principale:** predizione della classe binaria PHQ-8 mediante un LLM piccolo con pesi e attivazioni accessibili. Confrontare il modello base e, se necessario, un adattamento LoRA solo sul training; congelare poi checkpoint, prompt e soglia. Baseline predittive: classe maggioritaria e TF-IDF con regressione logistica.

Usare due simboli di classe verificati come singoli token. Definire lo score come differenza fra i loro logits e la probabilità come softmax limitata alle due opzioni. Quest’ultima non è automaticamente calibrata. Metriche predittive: macro-F1, AUROC e log-loss.

**Ambiente previsto:** Python, PyTorch, Transformers, PEFT per LoRA, strumenti di accesso alle attivazioni compatibili con il checkpoint, codice [IPE/MIB](https://github.com/nepp1d0/MIB-circuit-track-with-paths), scikit-learn e pandas. La compatibilità va verificata nel pilot.

### 3.3 Explainability, lessico e confronti

#### 3.3.1 Sviluppo del lessico

Partire da una risorsa inglese documentata, per esempio categorie selezionate di [NRC EmoLex](https://www.saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm). Registrare fonte, versione, categorie, condizioni d’uso, regole di matching e offset dei token. Un’eventuale estensione con espressioni di sonno, energia o interesse deve essere revisionata da persone e costruita senza utilizzare il test.

Valutare precisione/recall del matcher e accordo fra annotatori su un campione indipendente. Le categorie sono strumenti di analisi: non assegnano diagnosi o label PHQ-8 alle frasi.

#### 3.3.2 Procedura di ricerca

1. **Screening EAP-IG:** stimare l’importanza degli archi e conservare, ove possibile, i contributi per posizione. [Hanna et al., 2024](https://arxiv.org/abs/2403.17806).
2. **Guida lessicale:** combinare una graduatoria globale con una graduatoria relativa alle posizioni coperte dal lessico e al loro contesto; scegliere il peso sul development. La guida cambia la priorità dei candidati, mantenendo esplorazione anche fuori dal lessico.
3. **Interventi sui percorsi:** verificare candidati mediante IPE, con budget massimo di calcolo, profondità e regola di prosecuzione comuni. [Brunello et al., 2025](https://aclanthology.org/2025.blackboxnlp-1.30/).
4. **Verifica congiunta:** costruire circuiti di dimensioni comparabili e misurarne il comportamento intervenendo sui contributi esterni. Gli effetti dei singoli percorsi non sono automaticamente additivi.
5. **Congelamento:** valutare i circuiti su nuovi dati senza riscoprirli sul test.

Il testo e i pesi del modello originale rimangono fissi. Le esecuzioni con interventi sono esperimenti separati: sostituire attivazioni modifica il calcolo interno, senza necessariamente modificare i token di input. Le coppie di riferimento devono essere coerenti e allineate nelle posizioni su cui si interviene.

#### 3.3.3 Confronti da realizzare

| Strategia | Funzione nel confronto |
|---|---|
| EAP-IG con selezione standard degli archi | Baseline che verifica se l’ulteriore ricerca dei percorsi è utile. |
| Ricerca con priorità globale | Stessa procedura proposta, senza contributo lessicale. |
| Ricerca con priorità lessicale | Variante da valutare. |
| Ricerca con maschere casuali abbinate | Controllo di numero, posizione, frequenza e lunghezza degli span. |
| Ricerca guidata dalla salienza dei token | Controllo del vantaggio rispetto a informazione ricavata direttamente dal modello. |
| IPE originale e metodo position-aware | Riferimenti metodologici su un sottoinsieme sostenibile e con protocollo comparabile. |

[Position-aware Automatic Circuit Discovery](https://aclanthology.org/2025.acl-long.141/) considera già posizioni e span semantici. La proposta deve dimostrare il valore incrementale del lessico: la sola combinazione di tecniche esistenti e DAIC non garantisce originalità. Il controllo con salienza non equivale a riprodurre il metodo position-aware.

### 3.4 Analisi quantitative

**Riferimento della fedeltà:
** **l*’output del modello completo sulla stessa intervista, non il PHQ-8 osservato.*** Indicando con `p_M(x)` l’output originale e con `p_C(x; r)` quello ottenuto dal circuito sotto un riferimento di intervento fissato:

`E_fid(C) = media_per_partecipante [abs(p_M(x) − p_C(x; r))]`

Il grafo completo deve avere errore prossimo a zero. È il massimo teorico, ma conserva tutto il modello: confrontare quindi circuiti di pari dimensione. Aggiungere circuito vuoto e sottografi casuali come controlli; dichiarare quando il riferimento produce un contrasto troppo piccolo per essere informativo.

- **Efficienza:** curva errore–tempo GPU, area su un intervallo comune e costo per raggiungere una tolleranza fissata sul development. Includere screening, preparazione della guida e ricerca; riportare separatamente annotazione e valutazione finale.
- **Fedeltà e compattezza:** errore su probabilità e logits, accordo delle classi, numero di archi/nodi.
- **Generalizzazione:** aumento dell’errore su parafrasi e persone nuove; distinguere instabilità del modello originale e infedeltà del circuito.
- **Contesto:** distanza fra cambiamento dello score del modello e del circuito, `abs(Δs_M − Δs_C)`.
- **Incertezza:** confronti appaiati, bootstrap per partecipante e repliche della scoperta. Stabilire soglie e differenze minime di interesse prima del test.

Il risparmio è un’ipotesi: cambiare soltanto l’ordine di esplorazione non riduce necessariamente il costo se si esaminano infine gli stessi candidati.

### 3.5 Perimetro e piano alternativo

Il nucleo comprende un modello, un task, un lessico, controlli abbinati e trasferimento alle parafrasi. Se IPE è troppo costoso, passare alla selezione di archi EAP-IG con verifica mediante patching: la domanda diventa il compromesso **dimensione–fedeltà**, senza presumere un risparmio nello screening. Se il predittore PHQ-8 resta non studiabile dopo un adattamento limitato, trasferire la domanda metodologica a un task testuale più semplice, aggiornando il titolo. Interventi non corretti devono essere riparati o sostituiti con strumenti validati prima di formulare conclusioni causali.

## 4. Esperimenti da realizzare

| Nome esperimento | A quale Ipotesi/Domanda risponde |
|---|---|
| **E0  Preparazione di dati e lessico:** split, copertura, accordo e qualità del matcher. | H0, H2: affidabilità della risorsa e dei confronti. |
| **E1  Pilot del modello e degli interventi:** prestazioni, replica tecnica, grafo completo/vuoto, memoria e tempi. | H0: condizioni per proseguire e scelta del perimetro sostenibile. |
| **E2  Ricerca a budget controllato:** curve costo–fedeltà per strategie e dimensioni del circuito. | H1, H2: efficienza e contributo specifico del lessico. |
| **E3  Trasferimento a nuove persone e parafrasi:** circuiti congelati e distanza dagli output originali. | H3: generalizzazione oltre dati e vocabolario di scoperta. |
| **E4  Variazioni contestuali:** negazione, tempo e persona, con revisione delle coppie. | H4: riproduzione della sensibilità contestuale del modello. |
| **E5 Robustezza del risultato:** repliche, riferimenti alternativi e controlli di copertura/dimensione. | H1, H2: verifica che il vantaggio non sia un artefatto della selezione o del patching. |
| **E6  Replica esterna, opzionale:** seconda configurazione o task compatibile. | H3: portata delle conclusioni oltre il primo esperimento. |

## 5. Titoli in base all’output degli esperimenti

| Esiti osservati | Titolo finale possibile |
|---|---|
| E2 mostra un vantaggio, E3 ne conferma il trasferimento ed E5 ne sostiene la robustezza. | **Scoperta efficiente e generalizzabile di circuiti negli LLM mediante guida lessicale: uno studio su DAIC-WOZ.** |
| E2 mostra un vantaggio, ma E3 rileva un deterioramento sulle parafrasi. | **Efficienza e limiti di generalizzazione della guida lessicale nella scoperta di circuiti degli LLM.** |
| E2 ed E5 mostrano un beneficio trascurabile rispetto ai controlli, con intervalli sufficientemente stretti. | **Valore e limiti della guida lessicale nella scoperta di circuiti: una valutazione controllata su DAIC-WOZ.** |
| Le differenze restano incerte per variabilità o campione limitato. | **Valutazione sperimentale della guida lessicale nella ricerca di circuiti degli LLM su DAIC-WOZ.** |
| Si adotta il piano alternativo sugli archi, con verifiche di fedeltà e trasferimento. | **Guida lessicale alla selezione di circuiti negli LLM: compattezza, fedeltà e generalizzazione.** |

