# Tecniche di explainability per le proposte 00, 01 e 02

## Scopo del documento

Questo documento raccoglie le tecniche di explainability realisticamente utilizzabili nelle tre proposte. Non è un elenco di strumenti equivalenti: ogni famiglia risponde a una domanda diversa e fornisce un livello diverso di evidenza.

Le tre domande fondamentali sono:

1. **A che cosa è sensibile il modello?** Si modificano input o esempi e si osserva l'output.
2. **Dove è rappresentata o mediata l'informazione?** Si leggono o perturbano attivazioni interne.
3. **Come viene trasformata in una decisione?** Si cercano feature e percorsi, poi si verificano con interventi.

Una spiegazione plausibile non è necessariamente fedele. Un token evidenziato, un'attenzione elevata, un probe accurato o un concetto leggibile costituiscono un'ipotesi. L'evidenza causale riguarda invece l'effetto di un intervento definito sul comportamento del modello. Anche l'intervento deve essere controllato: può creare input fuori distribuzione o danneggiare genericamente il calcolo.

## Gerarchia operativa delle evidenze

| Livello | Esempio | Cosa permette di dire | Cosa non permette di dire |
| --- | --- | --- | --- |
| Comportamentale | coppie minime, swap, mask | l'output dipende dalla variabile modificata | quale componente interna produce l'effetto |
| Attribuzione sull'input | Integrated Gradients, occlusion | quali token o gruppi ricevono un punteggio di importanza | che il ranking descriva il vero meccanismo |
| Readout rappresentazionale | probe, Logit Lens, Jacobian Lens | quale informazione è decodificabile o verbalizzabile | che il modello usi tale informazione per decidere |
| Localizzazione causale | activation patching, ablation | un sito o componente influenza l'output nell'intervento scelto | che l'intero significato attribuito al sito sia la causa |
| Circuito candidato | path patching, Circuit Tracer | un insieme di feature e percorsi può spiegare parte del calcolo | completezza o validità globale senza replica e perturbazioni |

## 1. Esperimenti comportamentali e controfattuali

### Coppie minime e controfattuali appaiati

Si costruiscono due input che differiscono per una sola proprietà: una negazione, la famiglia di prompt dell'intervistatore, l'allineamento dei landmark o la voce associata al transcript. La misura primaria è il cambiamento del logit gap.

```text
g(x) = logit(classe_B | x) - logit(classe_A | x)
effect(x, x') = g(x') - g(x)
```

**Punti di forza:** è il modo più diretto per dimostrare che il comportamento dipende da una variabile controllata; non richiede accesso alle attivazioni.

**Limiti:** il controfattuale può non essere naturale; una modifica può alterare più fattori contemporaneamente.

**Uso:** obbligatorio in 00, 01 e 02.

### Masking, occlusion e leave-one-out

Si rimuove o sostituisce un token, uno span, un turno, una domanda o un gruppo di landmark e si misura l'effetto. Varianti rilevanti:

- leave-one-token/span-out;
- leave-one-turn-out;
- leave-one-prompt-out;
- leave-one-landmark-group-out;
- maschera length-matched;
- sostituzione con un elemento matched anziché cancellazione.

**Punti di forza:** semplice, model-agnostic, produce effetti direttamente interpretabili.

**Limiti:** la cancellazione può generare input fuori distribuzione; gli effetti non sono additivi quando esistono interazioni.

**Uso:** 01 per le domande dell'intervistatore; 02 per turni e landmark; controllo secondario in 00.

### Permutation, shuffle e swap test

Si preservano alcune statistiche dell'input ma si rompe l'associazione di interesse. Esempi:

- permutare la famiglia di prompt;
- mescolare l'ordine dei landmark;
- spostare i landmark tra turni dello stesso speaker;
- scambiare i landmark fra speaker matched;
- permutare etichette in un controllo di significatività.

**Punti di forza:** distingue contenuto, ordine, allineamento e identità.

**Limiti:** uno shuffle totale può essere troppo innaturale; servono più controlli con diverso grado di perturbazione.

**Uso:** centrale in 01 e 02.

### Deletion e insertion curves

Si rimuovono o inseriscono progressivamente gli elementi seguendo il ranking prodotto da un explainer. Un buon ranking dovrebbe causare un cambiamento maggiore rispetto a ordini casuali o matched.

**Metriche:** area under deletion curve, area under insertion curve, `effect@k`, random-gap@`k`.

**Uso:** validazione di Integrated Gradients e altri ranking in 01 e 02.

## 2. Attribuzione sull'input basata sui gradienti

### Vanilla saliency e Gradient × Input

La saliency usa il gradiente del target rispetto all'embedding di input. Gradient × Input combina sensibilità locale e valore dell'input.

**Vantaggi:** economici e facili da calcolare.

**Limiti:** rumorosi, locali e sensibili alla parametrizzazione; il punteggio non equivale all'effetto di rimozione.

**Ruolo:** baseline economica, non metodo principale.

### Integrated Gradients

[Integrated Gradients](https://arxiv.org/abs/1703.01365) integra i gradienti lungo un percorso tra una baseline e l'input reale. Soddisfa proprietà formali come sensitivity e implementation invariance.

**Decisioni necessarie:** baseline degli embedding, target scalare, numero di step, aggregazione sulle dimensioni e raggruppamento dei subtoken.

**Limiti:** la baseline può essere arbitraria e il percorso può attraversare rappresentazioni non realistiche. La completezza matematica non garantisce fedeltà causale.

**Uso:** metodo post-hoc principale in 01 e 02; opzionale in 00 come confronto sull'input.

### SmoothGrad e medie su perturbazioni

Si mediano gradienti ottenuti aggiungendo piccolo rumore. Può ridurre la variabilità visiva, ma non corregge una definizione del target o una baseline inadeguata.

**Uso:** controllo di stabilità, non necessario nel nucleo.

### DeepLIFT e Layer-wise Relevance Propagation

Propagano una differenza o una rilevanza dall'output verso l'input tramite regole specifiche.

**Vantaggi:** spesso più economici dell'integrazione su molti step.

**Limiti:** richiedono scelte implementative delicate nei Transformer, LayerNorm, residual connection e attention; spiegazioni ottenute con regole diverse possono divergere.

**Uso:** alternative a Integrated Gradients solo se la libreria supporta correttamente il checkpoint.

### Strumenti

- [Captum](https://captum.ai/) per Integrated Gradients, saliency, DeepLIFT e occlusion in PyTorch;
- hook PyTorch per aggregare score su subtoken, span, turni e gruppi di landmark.

## 3. Metodi perturbativi e surrogate model

### LIME

Approssima localmente il comportamento del modello con un modello interpretabile addestrato su perturbazioni dell'input.

**Limiti:** dipende fortemente da come vengono generate e pesate le perturbazioni; sui testi lunghi può essere instabile.

**Uso:** non prioritario; può essere una baseline model-agnostic su pochi esempi.

### SHAP e Shapley values approssimati

Assegna credito agli elementi dell'input considerando coalizioni di feature. Le approssimazioni riducono il costo combinatorio.

**Limiti:** definire l'assenza di un token o turno è difficile; feature correlate e input non naturali rendono l'interpretazione problematica.

**Uso:** opzionale. Non sostituisce i controfattuali progettati specificamente per 01 e 02.

### Permutation importance

Si permuta una feature o un gruppo e si misura la perdita di performance. È particolarmente naturale per famiglie di prompt e gruppi di landmark.

**Limiti:** una permutazione non condizionata può rompere correlazioni realistiche.

**Uso:** centrale in 02 e utile in 01, preferibilmente con matching.

## 4. Attention-based explanation

### Mappe di attenzione

Visualizzano quali posizioni ricevono peso da una testa in un forward pass.

**Cosa mostrano:** routing dell'informazione all'interno di una specifica testa.

**Cosa non mostrano:** importanza causale complessiva per l'output. Il valore scritto attraverso il circuito OV, le residual connection e i layer successivi possono rendere poco informativo il solo peso di attenzione.

### Attention rollout e attention flow

Aggregano mappe attraverso layer e teste per stimare percorsi fra token.

**Limiti:** le regole di aggregazione sono semplificazioni; non incorporano automaticamente MLP, segno delle contribuzioni o non-linearità.

**Uso nelle proposte:** diagnostica visuale o generazione di ipotesi. Non deve costituire la prova principale in nessuna delle tre.

### Direct Logit Attribution

La Direct Logit Attribution proietta il contributo scritto da una testa, un MLP o un'altra componente sulla direzione del logit target o del logit gap.

**Cosa mostra:** il contributo diretto della componente al residual stream letto in output.

**Limiti:** non include gli effetti indiretti che la componente produce attraverso layer successivi e dipende dal trattamento della normalizzazione finale. È un metodo di screening, non un intervento.

**Uso:** può restringere i candidati da verificare con ablation o patching in 01 e 02; baseline interna opzionale in 00.

### Norme dei pesi e differenze di parametri

Si confrontano norme, differenze pre/post-training o grandezza media assoluta di matrici, adapter o blocchi LoRA. È il tipo di analisi utilizzato nella sezione 5 di Zhang et al.

**Cosa mostra:** dove il processo di ottimizzazione ha prodotto aggiornamenti grandi secondo la metrica scelta.

**Cosa non mostra:** quali parametri siano necessari per una singola predizione, quale informazione codifichino o quale sia il loro effetto causale. Una matrice può cambiare molto e avere effetti ridondanti; una piccola modifica può essere funzionalmente decisiva.

**Uso:** descrizione del training nella 02 e diagnostica LoRA nella 01, sempre separata da ablation e patching.

## 5. Probe e decodifica delle rappresentazioni

### Linear probes

Un classificatore lineare viene addestrato sulle attivazioni per predire una proprietà: dominio semantico, famiglia di prompt, condizione allineata/disallineata o identità del parlante.

**Interpretazione corretta:** la proprietà è decodificabile da quel sito con il probe scelto.

**Errore comune:** concludere che il modello utilizzi la proprietà. Un probe può apprendere il task o sfruttare informazione presente ma funzionalmente irrilevante. Servono probe regolarizzati, split corretti e control task; [Hewitt e Liang](https://arxiv.org/abs/1909.03368) propongono di valutarne la selettività.

**Uso:**

- 00: controllo rappresentazionale secondario;
- 01: decodificare famiglia di prompt o shortcut;
- 02: distinguere input allineati, speaker o gruppi di landmark.

### Concept Activation Vectors e TCAV

Si apprende una direzione che separa esempi positivi e negativi di un concetto e si misura la sensibilità del target lungo quella direzione.

**Limiti:** dipende dal set di esempi usato per definire il concetto; separabilità e gradiente direzionale non dimostrano uso causale.

**Uso:** opzionale per domini PHQ-8 o shortcut note, seguito da interventi direzionali.

### Analisi geometrica e similarity analysis

PCA, UMAP/t-SNE, representational similarity analysis, CKA e confronti fra centroidi descrivono la geometria delle attivazioni tra layer, classi o condizioni.

**Vantaggi:** utili per verificare se hint, shortcut o allineamento cambiano globalmente le rappresentazioni.

**Limiti:** le visualizzazioni dipendono dalla proiezione; separazione geometrica e similarità non implicano uso funzionale.

**Uso:** diagnostica opzionale in tutte le proposte, in particolare per confrontare `hint-free` e `label-hint` nella 02.

### Logit Lens

Applica l'unembedding del modello alle attivazioni intermedie per osservare come evolve la distribuzione sui token.

**Vantaggi:** nessun training aggiuntivo, economico, utile come baseline.

**Limiti:** i layer intermedi possono usare basi diverse da quella finale; un token leggibile non prova che determini la decisione.

**Uso:** baseline obbligatoria nella 00; diagnostica opzionale in 01 e 02.

### Tuned Lens

Il [Tuned Lens](https://arxiv.org/abs/2303.08112) apprende una trasformazione affine per ciascun layer prima dell'unembedding, migliorando il readout rispetto al Logit Lens.

**Limiti:** richiede fitting e un corpus appropriato; il lens può introdurre informazione o bias propri. Va validato sul checkpoint esatto.

**Uso:** alternativa secondaria al Logit Lens nella 00, se il budget consente di fittarlo senza usare il test.

### Jacobian Lens e J-space

Il [Jacobian Lens](https://transformer-circuits.pub/2026/workspace/index.html) trasporta le attivazioni intermedie verso lo spazio finale mediante un Jacobiano medio e legge contenuti verbalizzabili.

**Output:** token o concetti verbalizzabili per posizione e layer; traiettorie di comparsa e persistenza.

**Limiti:** linearizzazione, dipendenza dal corpus e dal checkpoint, difficoltà con concetti multi-token. Un alto score non dimostra che quel concetto causi la classe.

**Uso:** tecnica centrale e oggetto di valutazione nella 00. In 01 e 02 è opzionale e normalmente non trasferibile dopo LoRA, modifica degli embedding o aggiunta di token senza rifitting e nuova validazione.

## 6. Neuroni, feature sparse e dizionari

### Analisi di neuroni e activation maximization

Si cercano neuroni con attivazioni elevate su un insieme di esempi e se ne ispezionano i contesti massimi. L'activation maximization ottimizza o cerca input che attivino un'unità.

**Limiti:** i neuroni sono spesso polisemantici; massimi esempi correlazionali non dimostrano funzione causale.

**Uso:** esplorativo, seguito da ablation.

### Sparse Autoencoders

Gli SAE decompongono attivazioni dense in feature sparse. Il lavoro di [Cunningham et al.](https://arxiv.org/abs/2309.08600) mostra che tali feature possono risultare più interpretabili dei singoli neuroni.

**Output:** feature, esempi di massima attivazione, decoder direction e ricostruzione dell'attivazione.

**Limiti:** feature morte o polisemantiche, errore di ricostruzione, scelta della sparsità e costo di training. Una descrizione automatica della feature non è una validazione.

**Uso:** estensione in 01 e 02; nella 00 può fornire un confronto fra concetti J-Lens e feature sparse.

### Transcoders e cross-layer transcoders

Approssimano la trasformazione degli MLP con feature sparse e direzioni di output, facilitando la costruzione di grafi di attribuzione.

**Limiti:** il replacement model è approssimato; error nodes e ricostruzione devono essere riportati. Gli artefatti sono specifici del checkpoint.

**Uso:** prerequisito di molte pipeline Circuit Tracer, non parte del minimo delle tesi.

### Strumenti

- [SAE Lens](https://github.com/decoderesearch/SAELens) per addestramento e analisi di SAE;
- [Neuronpedia](https://www.neuronpedia.org/) per esplorare feature e artefatti pubblici;
- gli esempi pubblici non sostituiscono l'analisi locale sul checkpoint della tesi.

## 7. Spiegazioni basate sugli esempi e sui dati di training

### Nearest-neighbor e prototipi

Si recuperano esempi di training o reference set vicini all'input o alla sua rappresentazione interna. Forniscono una spiegazione per analogia: “la predizione assomiglia a questi casi”.

**Limiti:** la distanza scelta può non riflettere la funzione decisionale; esempi simili possono avere influenza nulla. Nei corpora clinici i risultati non devono esporre transcript riconoscibili.

**Uso:** controllo qualitativo in 01 e 02, soprattutto per individuare duplicati, speaker leakage o sub-dialoghi quasi identici.

### Influence functions, TracIn e training-data attribution

Stimano quali esempi di training abbiano favorito o sfavorito una predizione. Le influence functions approssimano l'effetto di ri-pesare un esempio attraverso informazioni di curvatura; TracIn usa prodotti di gradienti lungo checkpoint di training.

**Vantaggi:** possono rilevare esempi dominanti, contaminazione, memorization e dipendenza da campioni etichettati.

**Limiti:** costosi e approssimati nei modelli grandi; sensibili al percorso di ottimizzazione, ai checkpoint salvati e all'adattamento PEFT. Non spiegano direttamente il circuito di inferenza.

**Uso:** estensione della 01 per capire quali esempi insegnano la shortcut e della 02 per auditare l'effetto del label hint. Non fanno parte del minimo.

### Baseline intrinsecamente interpretabili

Regressione logistica su TF-IDF, conteggi di famiglie di prompt o frequenze dei landmark produce coefficienti globali ispezionabili. Concept bottleneck e modelli rule-based possono rendere espliciti attributi intermedi.

**Ruolo:** non spiegano il piccolo LLM, ma mostrano quale segnale è disponibile nel dataset e forniscono un riferimento contro cui giustificare la complessità del modello.

**Uso:** obbligatorio come baseline in 01 e 02; utile nel Livello B della 00.

## 8. Interventi causali interni

### Ablation di componenti

Si azzera, sostituisce con la media o modifica:

- un neurone o una feature SAE;
- una testa di attenzione;
- l'output di un MLP;
- una posizione del residual stream;
- una matrice o un adapter LoRA.

**Punti di forza:** misura un effetto funzionale diretto.

**Limiti:** l'azzeramento può essere fuori distribuzione e l'effetto può riflettere danno generale. Servono mean ablation, controlli matched, patch inverso e target di controllo.

**Uso:** 01 e 02 dopo la localizzazione; 00 come verifica di siti o direzioni candidate.

### Activation patching o causal tracing

Si eseguono una corsa source e una target, poi si sostituisce un'attivazione interna della target con quella della source. È chiamato anche interchange intervention o causal tracing.

**Output:** effetto `layer × posizione × componente` sul target scelto.

**Limiti:** risultati sensibili alla costruzione source/target, alla metrica e al tipo di patch. [Zhang e Nanda](https://arxiv.org/abs/2309.16042) mostrano che scelte diverse possono produrre localizzazioni diverse.

**Uso:**

- 00: validazione principale dei siti selezionati da J-Lens;
- 01: localizzazione della differenza causata dalla famiglia di prompt;
- 02: mediazione della differenza tra landmark allineati e controfattuali.

### Path patching

Interviene su un percorso specifico fra componenti, per esempio da una testa sorgente a una testa o MLP target, mantenendo più parti del calcolo controllate.

**Vantaggi:** maggiore specificità rispetto al patching di un intero residual stream.

**Limiti:** molte forward pass, definizione delicata del percorso e rischio di perdere interazioni parallele.

**Uso:** estensione in 01; non necessario in 00 e 02.

### Attribution patching e AtP*

Usa gradienti per approssimare l'effetto di activation patching su molte componenti, riducendo il costo dello screening. I candidati devono poi essere verificati con patching reale.

**Uso:** utile se lo spazio di siti è grande; non è necessario quando 00 limita layer e posizioni o 02 usa uno sweep ristretto.

### Causal mediation analysis

Tratta un'attivazione come possibile mediatore tra una modifica dell'input e l'output, separando effetto diretto e indiretto sotto precise assunzioni.

**Limiti:** nei modelli profondi i mediatori interagiscono, gli interventi possono essere fuori distribuzione e le assunzioni causali devono essere dichiarate. Non basta rinominare il recovery del patching come “mediazione” per ottenere una prova semantica.

**Uso:** cornice teorica utile per 01 e 02; implementazione completa opzionale.

### Steering e directional interventions

Si aggiunge o sottrae una direzione — ottenuta da differenze di medie, probe, CAV, SAE o J-space — alle attivazioni interne.

**Cosa verifica:** se modificare la coordinata produce un effetto con segno previsto.

**Limiti:** interventi grandi possono uscire dal regime locale e introdurre effetti collaterali. Servono curve dose-risposta, direzioni random e target di controllo.

**Uso:** estensione naturale della 00; opzionale in 01 e 02.

## 9. Circuit discovery

### Circuit Tracer e attribution graphs

[Circuit Tracer](https://www.transformer-circuits.pub/2025/attribution-graphs/methods.html) costruisce un grafo locale che collega token, feature sparse e target attraverso effetti diretti stimati. Il grafo viene potato per renderlo leggibile e le ipotesi devono essere validate con perturbazioni delle feature.

**Output:** nodi/feature, edge con segno, percorsi verso un logit o logit gap, error nodes e grafi interattivi.

**Limiti:** dipende da transcoders compatibili e da un replacement model approssimato; i grafi sono prompt-specifici, la potatura può eliminare percorsi e alcune componenti dell'attention non sono completamente rappresentate.

**Uso:**

- 00: estensione qualitativa dopo il confronto J-Lens/Logit Lens/patching;
- 01: utile solo se esistono transcoders compatibili con il checkpoint finale; non è un requisito;
- 02: generalmente non adatto al nucleo perché token nuovi e fine-tuning modificano il checkpoint.

### Causal scrubbing

Si formalizza un'ipotesi di circuito e si sostituiscono attivazioni con campioni che preservano le equivalenze previste dall'ipotesi. Si verifica quanto comportamento rimane spiegato.

**Vantaggi:** valuta una teoria esplicita del calcolo, non singoli componenti isolati.

**Limiti:** richiede già un circuito candidato e un dataset di resampling accuratamente progettato; è costoso e metodologicamente impegnativo.

**Uso:** fuori dal nucleo delle tre proposte, possibile estensione della 01.

## 10. Spiegazioni in linguaggio naturale

### Self-explanations, rationale e Chain-of-Thought

Si chiede al modello di spiegare la propria risposta o produrre una catena di ragionamento.

**Vantaggi:** leggibili e facili da ottenere.

**Limiti:** possono essere razionalizzazioni post-hoc. Studi sulla fedeltà delle self-explanations mostrano che affidabilità e self-consistency dipendono da modello, task e metodo di valutazione. Non offrono accesso privilegiato al meccanismo interno.

**Uso:** materiale qualitativo o confronto di plausibilità; mai evidenza causale principale nelle proposte.

### Counterfactual explanations generate dal modello

Il modello propone la modifica minima che cambierebbe la classe. La modifica va poi eseguita realmente e controllata per validità semantica.

**Uso:** generazione assistita di candidati per 01; nella 02 i controfattuali acustici devono essere costruiti dalla pipeline, non inventati testualmente dal modello.

## 11. Come valutare una spiegazione

### Fedeltà

- **Comprehensiveness:** quanto cala il target rimuovendo gli elementi indicati.
- **Sufficiency:** quanto del target resta usando soltanto gli elementi indicati.
- **Deletion/insertion:** andamento del target rimuovendo o aggiungendo elementi secondo il ranking.
- **Intervention prediction:** capacità dello score di prevedere segno e ampiezza dell'intervento.
- **Random-gap:** vantaggio rispetto a elementi o siti casuali matched.

### Stabilità e robustezza

- overlap dei top-`k` fra parafrasi, seed e split;
- correlazione dei ranking;
- sensibilità a baseline, tokenizzazione e prompt format;
- replica su template o partecipanti tenuti fuori.

### Specificità e selettività

- effetto sul target principale meno effetto su un target linguistico non correlato;
- confronto con componenti dello stesso layer e attivazione simile;
- patch inverso e interventi su esempi neutri;
- controllo del danno generale tramite perplexity o task ausiliario.

### Plausibilità

Accordo con annotatori o conoscenza di dominio. È utile per la leggibilità, ma non sostituisce la fedeltà. La survey di [Lyu et al.](https://aclanthology.org/2024.cl-2.6/) organizza i metodi di spiegazione NLP proprio attorno a questa distinzione.

### Unità statistica

Le misure devono rispettare il disegno dei dati:

- partecipante per DAIC-WOZ/E-DAIC;
- template o famiglia di generazione per benchmark sintetici;
- coppia controfattuale per gli effetti appaiati;
- prompt, e non singolo token, per i grafi locali aggregati.

## 12. Matrice di scelta per le tre proposte

| Tecnica | 00 — J-Lens | 01 — Shortcut | 02 — Landmarks | Priorità |
| --- | --- | --- | --- | --- |
| Coppie minime/controfattuali | sì | sì | sì | obbligatoria |
| Leave-one-out/occlusion | controllo | sì | sì | alta in 01–02 |
| Integrated Gradients | baseline opzionale | sì | sì | alta in 01–02 |
| Permutation/swap | controllo | sì | centrale | obbligatoria dove applicabile |
| Attention maps | diagnostica | diagnostica | diagnostica | bassa |
| Direct Logit Attribution | opzionale | screening | screening | media |
| Norme/differenze dei pesi | no | diagnostica LoRA | confronto con Zhang | solo descrittiva |
| Linear probes | opzionale | opzionale | opzionale | media |
| Similarity/geometry | opzionale | opzionale | confronto hint | bassa |
| Logit Lens | baseline centrale | opzionale | opzionale | alta in 00 |
| Tuned Lens | estensione | raramente | raramente | bassa |
| Jacobian Lens | oggetto centrale | solo se compatibile | normalmente no dopo nuovi token | alta solo in 00 |
| SAE/feature analysis | estensione | estensione | estensione | bassa/media |
| Training-data attribution | no | estensione | estensione hint | bassa |
| Component ablation | verifica | sì | sì | alta dopo localizzazione |
| Activation patching | validazione centrale | nucleo interno | nucleo interno limitato | alta |
| Path patching | estensione | estensione | non necessario | bassa |
| Attribution patching | screening opzionale | screening opzionale | non necessario | media se molti siti |
| Steering direzionale | estensione naturale | opzionale | opzionale | media in 00 |
| Circuit Tracer | qualitativo opzionale | solo checkpoint compatibile | non consigliato nel nucleo | bassa |
| Self-explanation/CoT | solo confronto | solo confronto | solo confronto | non usare come prova |

## 13. Stack consigliato

| Esigenza | Strumenti possibili | Nota |
| --- | --- | --- |
| Modelli e fine-tuning | PyTorch, Hugging Face Transformers, PEFT | salvare checkpoint, tokenizer e adapter esatti |
| Gradient attribution | Captum | verificare baseline, convergenza e aggregazione subtoken |
| Hook e patching | hook PyTorch, TransformerLens, NNsight | la compatibilità dipende dall'architettura |
| Interventi strutturati | pyvene o implementazione controllata | validare prima su un caso sintetico noto |
| Probe e baseline | scikit-learn | split e regolarizzazione fissati sul development |
| SAE | SAE Lens | addestrare da zero solo se necessario e con budget adeguato |
| J-Lens | repository Jacobian Lens e artefatti Neuronpedia | checkpoint-specifico |
| Circuiti | Circuit Tracer e transcoders compatibili | sempre verificare reconstruction error e interventi |
| Analisi statistica | pandas, scipy/statsmodels, bootstrap appaiato | unità statistica coerente con partecipanti/template |

## 14. Regole pratiche comuni

1. Definire un target scalare, preferibilmente un logit gap fra token di classe singoli.
2. Verificare prima che il modello svolga il task e che esista l'effetto da spiegare.
3. Separare dati per partecipante o template prima di generare varianti.
4. Selezionare siti, `k`, baseline e soglie sul development; il test verifica, non guida.
5. Confrontare metodi applicando lo stesso tipo di intervento e lo stesso budget.
6. Usare controlli random **matched**, non soltanto casuali senza vincoli.
7. Riportare risultati nulli, copertura ed esempi esclusi.
8. Non trasferire Lens, SAE o transcoders fra checkpoint diversi senza rivalidazione.
9. Non chiamare “causale” un ranking di gradienti, un probe o la grandezza dei pesi.
10. Limitare ogni conclusione al comportamento del modello: nessuna tecnica dimostra uno stato clinico della persona.

## Riferimenti essenziali

- [Lyu et al. (2024), *Towards Faithful Model Explanation in NLP*](https://aclanthology.org/2024.cl-2.6/).
- [Sundararajan et al. (2017), *Axiomatic Attribution for Deep Networks*](https://arxiv.org/abs/1703.01365).
- [Hewitt e Liang (2019), *Designing and Interpreting Probes with Control Tasks*](https://arxiv.org/abs/1909.03368).
- [Belrose et al. (2023), *Eliciting Latent Predictions from Transformers with the Tuned Lens*](https://arxiv.org/abs/2303.08112).
- [Cunningham et al. (2023), *Sparse Autoencoders Find Highly Interpretable Features in Language Models*](https://arxiv.org/abs/2309.08600).
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching*](https://arxiv.org/abs/2309.16042).
- [Ameisen et al. (2025), *Circuit Tracing: Revealing Computational Graphs in Language Models*](https://www.transformer-circuits.pub/2025/attribution-graphs/methods.html).
- [Gurnee et al. (2026), *Verbalizable Representations Form a Global Workspace in Language Models*](https://transformer-circuits.pub/2026/workspace/index.html).
