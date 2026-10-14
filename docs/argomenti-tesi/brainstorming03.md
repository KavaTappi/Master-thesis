# Brainstorming 03 — nuove direzioni di explainability degli LLM e matching con Mark J. Carman

**Ricognizione bibliografica: 14 ottobre 2026.** Questo documento propone quattro argomenti diversi dai nuclei di brainstorming01 e brainstorming02: qui l'oggetto è l'adattamento del modello, l'influenza degli esempi nel prompt, l'affidabilità di una rappresentazione a concetti e l'efficienza della valutazione delle spiegazioni. DAIC-WOZ serve come fonte di interviste ed etichette; E-DAIC può servire per una verifica esterna, previa verifica di sovrapposizioni, trascrizioni e label.

Le tecniche generali sotto elencate esistono già. Il contributo candidato è un confronto sperimentale specifico, indicato in ogni scheda. La ricerca ha escluso alcune rivendicazioni troppo ampie; non certifica un primato assoluto. Le proposte richiedono un controllo bibliografico finale e un pilot prima della scelta con il relatore.

## 1. Che cosa spiega il cambiamento di un LLM dopo il fine-tuning?

**Titolo possibile:** *Dal rango all'effetto: verifica causale dell'adattamento LoRA di un LLM sulle interviste DAIC-WOZ*.

### Domanda e differenza rispetto ai brainstorming precedenti

Il fine-tuning distribuisce gli aggiornamenti fra diversi moduli del modello. I moduli cui viene assegnata più capacità sono anche quelli che spiegano maggiormente il cambiamento delle predizioni? Oppure rango e dimensione dei pesi sono indicatori poco affidabili del contributo effettivo?

Qui si spiega la **differenza fra modello iniziale e modello adattato**, mantenendo fissa l'intervista. Non occorre trovare prima una shortcut, modificare una negazione o analizzare le citazioni del modello.

### Tecniche di explainability e disegno

1. Partire da un LLM aperto e confrontare LoRA a rango fisso e, se sostenibile, una distribuzione adattiva del rango. Congelare prompt e formato di risposta; mantenere uguale il budget di parametri e documentare quello di addestramento.
2. **Ablazione degli adapter:** disattivare singoli gruppi di adapter a inferenza e misurare quanto si perde del cambiamento prodotto dal fine-tuning. La disattivazione di un adapter rimuove l'aggiornamento mantenendo il modulo base.
3. **Ripristino di sottoinsiemi:** partire dal modello base e riattivare gruppi di adapter selezionati. Confrontare ranking per rango, norma dell'aggiornamento, effetto di ablazione e selezione casuale con pari numero di parametri.
4. Misurare interazioni fra pochi gruppi: due adapter possono essere ridondanti o utili insieme. Non sommare automaticamente effetti di ablazione singola.
5. Solo come estensione, effettuare patching tra checkpoint compatibili o analisi di percorsi circoscritti. La differenza di attivazione è un indizio descrittivo; l'intervento verifica l'effetto sul modello.

### Valutazioni quantitative

| Misura | Come calcolarla e che cosa risponde |
|---|---|
| Prestazione e stabilità | Macro-F1, AUROC e log-loss su partecipanti disgiunti, con più seed. Verifica che il cambiamento studiato non sia soltanto instabilità dell'addestramento. |
| Cambiamento da spiegare | `d(x) = s_adattato(x) − s_base(x)`, con score di classe definito in modo identico. Riportare anche casi con differenza piccola. |
| Contributo del gruppo g | `A_g(x) = s_adattato(x) − s_senza_g(x)`, insieme alla variazione di log-loss. Un grande effetto sullo score può essere utile oppure dannoso. |
| Valore dei proxy | Correlazione di Spearman fra rango/norma e contributo causale aggregato; riportare sia valori grezzi sia confronto a parità di parametri. |
| Fedeltà del sottoinsieme | Errore medio `abs(s_adattato(x) − s_solo_S(x))` al crescere dei parametri riattivati; confrontare la curva con gruppi casuali e ranking semplici. |
| Interazione e replica | `I(g,h) = A_{g,h} − A_g − A_h`; replica di segno, ranking e recupero su seed e partecipanti non usati per selezionare i gruppi. |

Usare rapporti di recupero normalizzati soltanto quando `|d(x)|` supera una soglia prefissata; riportare sempre anche l'effetto non normalizzato. Gli interventi spiegano l'adattamento, non le cause cliniche della depressione.

### Paper vicini e revisione dell'idea

- [Singh et al., L1RA: Dynamic Rank Assignment in LoRA Fine-Tuning, 2025](https://aclanthology.org/2025.icnlsp-1.35/) — distribuisce il rango e analizza quali moduli vengono adattati. **Carman è coautore.** Fare soltanto una mappa dei ranghi replicherebbe un'analisi già proposta.
- [Wang et al., Towards Understanding Fine-Tuning Mechanisms of LLMs via Circuit Analysis, 2025](https://arxiv.org/abs/2502.11812) — studia il cambiamento dei circuiti durante l'adattamento su task matematici. «Studiare i circuiti prima e dopo» non è quindi una novità generale.
- [DepressLLM, 2025](https://arxiv.org/abs/2508.08591) — precedente di adattamento di LLM per la depressione e valutazione su ulteriori dati, compreso DAIC-WOZ. Il solo fine-tuning nel dominio non basta.
- [Brunello et al., IPE: Isolating Path Effects for Improving Latent Circuit Identification, 2025](https://aclanthology.org/2025.blackboxnlp-1.30/) — possibile riferimento per un'estensione su percorsi interni; **Carman è coautore**. Il costo della ricerca dei percorsi richiede un pilot, specialmente su input lunghi.

### Che cosa aggiunge allo stato attuale

**Testare se gli indicatori usati per descrivere l'adattamento, soprattutto il rango assegnato, predicono davvero il contributo degli adapter e consentono di ricostruire il cambiamento di decisione con pochi moduli.** DAIC fornisce un caso realistico con pochi dati e label a livello di intervista. Per sostenere un contributo metodologico oltre il dominio, aggiungere un piccolo task testuale di controllo, se il budget lo permette.

**Fattibilità:** medio-alta complessità; servono fine-tuning ripetibili e risorse per più seed. Se il rango risulta poco informativo, il confronto causale resta un risultato utile. Se il fine-tuning non produce un cambiamento stabile, la domanda non è studiabile con quella configurazione.

## 2. Quali esempi nel prompt guidano la decisione su un nuovo partecipante?

**Titolo possibile:** *Spiegazioni basate sugli esempi: attribuzione dell'influenza delle dimostrazioni in-context nella predizione su DAIC-WOZ*.

### Domanda e differenza rispetto ai brainstorming precedenti

Un LLM riceve nel prompt alcune interviste etichettate e poi una nuova intervista. Quali esempi spostano la sua decisione? L'esempio semanticamente più simile è anche il più influente? Un esempio può essere influente pur peggiorando la predizione?

L'unità della spiegazione è **un caso dimostrativo esterno**, non una frase del partecipante da valutare. L'LLM resta congelato; si studia l'apprendimento in-context.

### Tecniche di explainability e disegno

1. Creare un archivio di dimostrazioni esclusivamente dal training, con estratti selezionati da una regola fissata sullo sviluppo. Un estratto conserva la label dell'intervista come informazione dell'esempio, ma non diventa una frase clinicamente etichettata.
2. Confrontare selezione casuale bilanciata e recupero per similarità. Usare pochi esempi, per esempio quattro, per rendere possibile un'analisi esaustiva dei sottoinsiemi.
3. **Attribuzione per esempio:** calcolare contributi marginali e valori di Shapley sui sottoinsiemi di dimostrazioni; confrontare un'approssimazione come DETAIL. Il valore attribuito dipende dalla funzione scelta, per esempio lo score di classe.
4. **Sostituzione controllata:** sostituire un esempio con uno della stessa classe e lunghezza simile, mantenendo costanti numero, ordine e bilanciamento delle dimostrazioni. Questo aiuta a distinguere l'informazione del caso dagli effetti di lunghezza o prevalenza delle label.
5. Verificare se gli esempi più influenti sono anche utili: l'effetto sulla decisione e il miglioramento rispetto alla label di test sono quantità diverse. Non fornire al selettore le label dei partecipanti di test.

### Valutazioni quantitative

| Misura | Come calcolarla e che cosa risponde |
|---|---|
| Fedeltà dell'attribuzione | Correlazione fra ranking stimato e riferimento Shapley esaustivo sui quattro esempi; errore medio dei contributi e accordo del segno. Con quattro esempi servono 16 coalizioni per configurazione. |
| Similarità e influenza | Correlazione fra similarità di retrieval e contributo stimato. Una correlazione bassa limita l'uso del «caso più simile» come spiegazione del modello. |
| Utilità dell'esempio | Variazione di log-loss della query dopo sostituzione/rimozione, oltre a `Δs`; quota di esempi molto influenti ma dannosi rispetto alla label osservata. |
| Interazioni | Differenza tra effetto congiunto di due esempi e somma degli effetti individuali, rispetto allo stesso insieme di riferimento. |
| Valore pratico | Log-loss, Macro-F1 e costo di una selezione basata sull'attribuzione contro retrieval e casuale bilanciato, scegliendo qualsiasi regola solo sullo sviluppo. |

### Paper vicini e revisione dell'idea

- [Zhou et al., DETAIL: Task DEmonsTration Attribution for Interpretable In-context Learning, 2024](https://arxiv.org/abs/2405.14899) — propone già l'attribuzione delle dimostrazioni: è il metodo da confrontare, non una tecnica da rivendicare come nuova.
- [Explaining Large Language Models Decisions Using Shapley Values, 2024](https://arxiv.org/abs/2404.01332) — attribuisce contributi alle componenti del prompt; definire Shapley sul prompt non basta come originalità.
- [Feng et al., Affect Recognition in Conversations Using Large Language Models, 2024](https://aclanthology.org/2024.sigdial-1.23/) — include DAIC-WOZ e confronti zero-shot, few-shot e fine-tuning. Il solo confronto tra numeri di esempi è già coperto.
- [Yapa et al., How Interpretable Are LLMs? A Multi-Metric Framework for Evaluating Synthetic Explanations in Digital Mental Health, 2026](https://www.scitepress.org/PublishedPapers/2026/146449/) — studia spiegazioni e il loro uso negli esempi few-shot su SMHD; evitare di proporre soltanto «aggiungere spiegazioni alle dimostrazioni».

### Che cosa aggiunge allo stato attuale

**Verificare se il retrieval di casi simili offre anche una spiegazione affidabile della decisione e quando similarità, influenza e utilità divergono.** Il confronto con attribuzioni esaustive su piccoli prompt rende l'esito verificabile; le sostituzioni a classe e lunghezza controllate evitano di confondere la qualità dell'esempio con il semplice formato del prompt. L'eventuale novità è questo audit nelle interviste cliniche, non l'invenzione della demonstration attribution.

**Fattibilità:** media; non richiede fine-tuning, ma molte inferenze. Se tutti gli esempi hanno effetti trascurabili, occorre riportare il limite dell'ICL testato; non si può produrre una graduatoria informativa a partire da puro rumore.

## 3. I concetti leggibili permettono davvero di correggere il modello?

**Titolo possibile:** *Correggibilità e informazione nascosta nei modelli LLM a concetti per l'analisi di interviste DAIC*.

### Domanda e differenza rispetto ai brainstorming precedenti

Un sistema estrae dal testo concetti leggibili, per esempio evidenza di difficoltà nel sonno o perdita di interesse, e li usa per predire la label. Se una persona corregge un concetto estratto male, la decisione migliora? Oppure i valori numerici dei concetti trasportano informazioni ulteriori, rendendo la loro interpretazione ingannevole?

Si studia una **architettura interpretabile per costruzione**, `intervista → LLM → concetti → classificatore`. Questo spiega il nuovo sistema a concetti; non prova di spiegare il funzionamento di un LLM che predice direttamente dal testo.

### Tecniche di explainability e disegno

1. Definire pochi concetti osservabili nella trascrizione e distinguere «evidenza presente», «esplicitamente assente» e «non determinabile». Non usare la mancata menzione come assenza del sintomo. Le annotazioni testuali restano distinte dagli item PHQ-8 autoriferiti.
2. Confrontare un bottleneck discreto con uno continuo, per esempio probabilità delle categorie, seguito dallo stesso tipo di classificatore semplice. Rendere esplicita l'assenza di un percorso che passi direttamente gli embedding dell'intervista al classificatore.
3. **Spiegazioni per concetto:** mostrare contributi del classificatore e origine dei concetti estratti. Valutare qualità dell'estrazione separatamente dalla trasparenza della testa predittiva.
4. **Interventi sui concetti:** sostituire progressivamente concetti errati con annotazioni umane, confrontando ordine casuale e ordine basato su errore stimato o importanza. Usare soltanto errori annotati, non correzioni scelte per far concordare il risultato con la label.
5. **Audit del leakage semantico:** verificare se i valori continui contengono informazione sul target oltre le categorie annotate. La possibilità di predire informazione residua è un indizio, da controllare per incompletezza delle annotazioni e capacità dei modelli, non una prova automatica di leakage.

### Valutazioni quantitative

| Misura | Come calcolarla e che cosa risponde |
|---|---|
| Qualità dei concetti | Macro-F1 per categoria e accordo fra annotatori; matrice di confusione tra assente e non determinabile. |
| Trasparenza/prestazione | Macro-F1 e log-loss per bottleneck discreto, continuo e LLM diretto. Quest'ultimo è un confronto predittivo, non il sistema spiegato dal bottleneck. |
| Beneficio delle correzioni | Curva di log-loss e accuratezza dopo 0, 1, 2, … correzioni corrette; area sotto la curva e numero di correzioni necessario per un miglioramento prefissato. |
| Correzioni dannose | Percentuale di casi in cui una correzione semanticamente corretta aumenta la loss; riportare che anche concetti corretti non determinano perfettamente il PHQ-8. |
| Informazione residua | Miglioramento di log-loss fuori campione aggiungendo valori continui alle categorie annotate, a capacità del predittore controllata; affiancare misure pubblicate di leakage solo se stimabili con il campione disponibile. |

### Paper vicini e revisione dell'idea

- [Sun et al., Concept Bottleneck Large Language Models, 2024](https://arxiv.org/abs/2412.07992) — i CB-LLM esistono già.
- [Interpretable-by-Design Text Understanding with Iteratively Generated Concept Bottleneck](https://arxiv.org/abs/2310.19660) — precedente di concetti testuali generati con LLM.
- [Lee et al., Interpretable depression assessment using a large language model, 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12885269/) — fattori estratti con LLM e regressione su DAIC-WOZ/E-DAIC: «usare fattori interpretabili per predire» sarebbe troppo vicino.
- [Havasi et al., Addressing Leakage in Concept Bottleneck Models, 2022](https://openreview.net/forum?id=tglniD_fn9) e [Parisini et al., Leakage and Interpretability in Concept-Based Models, 2026](https://www.jmlr.org/papers/v27/25-1121.html) — formalizzano problemi e misure di leakage: non è un fenomeno nuovo.
- [Are Concept Bottleneck Models Effective as Decision-Support Systems?, 2026](https://arxiv.org/abs/2608.25581) — precedente recente sulla valutazione delle correzioni da parte degli utenti; il semplice coinvolgimento umano non basta.

### Che cosa aggiunge allo stato attuale

**Valutare con annotazioni indipendenti se il vantaggio predittivo dei concetti continui si paga con minore correggibilità, soprattutto quando le interviste non documentano tutti i sintomi.** Il contributo è il legame misurato fra rappresentazione dei concetti, informazione residua e utilità delle correzioni. Non si propone un'altra pipeline di estrazione dei fattori come novità autonoma.

**Fattibilità:** media-alta soprattutto per il lavoro di annotazione; senza abbastanza casi revisionati l'audit diventa debole. Si può misurare la correggibilità con correzioni umane già annotate; non serve promettere uno studio di utilizzo clinico.

## 4. Quanto costa stabilire quale metodo di explainability è più affidabile?

**Titolo possibile:** *Valutazione adattiva dei metodi di explainability sotto un budget di inferenza: uno studio su LLM e interviste DAIC*.

### Domanda e differenza rispetto ai brainstorming precedenti

Valutare una spiegazione può richiedere decine di perturbazioni per intervista. Si può scegliere in modo adattivo **quali confronti eseguire** per identificare il metodo più fedele con meno inferenze, senza favorire i casi facili o dichiarare vincitori inesistenti?

L'oggetto è il **metodo di valutazione e il suo costo**, non uno specifico tipo di shortcut. DAIC è un banco di prova con testi lunghi, costo variabile e poche persone indipendenti.

### Tecniche di explainability e disegno

1. Congelare tre spiegatori comparabili, per esempio attribuzione per gradiente aggregata a turno, surrogate locale e attribuzione per perturbazioni. Definire la stessa granularità e il medesimo budget di spiegazione.
2. Definire prima del campionamento una misura di fedeltà primaria: ad esempio capacità del ranking di importanza di anticipare gli effetti su uno **specifico insieme di interventi validati**. La misura resta relativa a quell'insieme, senza essere una verità assoluta sul modello.
3. Su un sottoinsieme sostenibile, calcolare una valutazione estesa di riferimento. Simulare budget ridotti rivelando progressivamente gli esiti all'algoritmo di selezione; gli esiti non ancora acquistati devono restare nascosti.
4. Confrontare campionamento uniforme, stratificato per partecipante/lunghezza e una strategia di identificazione del miglior metodo basata sull'incertezza. Usare confronti appaiati sugli stessi casi dove possibile e includere l'opzione «evidenza insufficiente per scegliere».
5. Confermare la politica di selezione su nuove interviste. Contabilizzare anche il costo di produrre le spiegazioni, le perturbazioni e la validazione, distinguendo costi condivisi e costi marginali.

### Valutazioni quantitative

| Misura | Come calcolarla e che cosa risponde |
|---|---|
| Identificazione corretta | Frequenza con cui la scelta a budget limitato concorda con il riferimento esteso; gestire esplicitamente metodi quasi equivalenti. |
| Regret della scelta | Fedeltà del migliore nel riferimento meno fedeltà del metodo scelto, al variare del budget; zero indica la scelta ottimale. |
| Costo | Inferenze, token processati e tempo GPU per raggiungere un errore di selezione prefissato; riportare anche il costo iniziale dell'audit esteso. |
| Calibrazione della decisione | Frequenza di false dichiarazioni di superiorità in simulazioni con metodi equivalenti; non usare intervalli ordinari ignorando selezione adattiva e arresto anticipato. |
| Copertura | Distribuzione dei casi acquistati per persona, classe e lunghezza; errore della stima finale rispetto alla distribuzione bersaglio fissata prima della selezione. |

### Paper vicini e revisione dell'idea

- [Haffari, Tran e Carman, Efficient Benchmarking of NLP APIs using Multi-armed Bandits, 2017](https://aclanthology.org/E17-1039/) — precedente diretto di benchmarking efficiente, anche se l'oggetto è la prestazione dei sistemi NLP.
- [Neuron Shapley: Discovering the Responsible Neurons, NeurIPS 2020](https://proceedings.neurips.cc/paper/2020/file/41c542dfe6e4fc3deb251d64cf6ed2e4-Paper.pdf) — usa già campionamento adattivo per attribuzioni; non rivendicare come nuova l'unione generica di bandit e XAI.
- [Cost-Aware Multi-Objective Bandits: Theory and Application to Budgeted LLM Configuration Evaluation, 2026](https://arxiv.org/abs/2608.04333) — mostra che anche la valutazione a costo limitato degli LLM ha precedenti recenti.
- [Kamahi e Yaghoobzadeh, Counterfactuals As a Means for Evaluating Faithfulness of Attribution Methods in Autoregressive Language Models, 2024](https://aclanthology.org/2024.blackboxnlp-1.28/) — riferimento per il disegno delle perturbazioni linguistiche da utilizzare nell'audit.

### Che cosa aggiunge allo stato attuale

**Un protocollo di selezione e arresto per confrontare spiegatori su documenti lunghi, tenendo insieme costo variabile, dipendenza tra interventi della stessa persona e conferma su persone nuove.** La sola applicazione di Thompson Sampling a DAIC sarebbe debole. Per sostenere una novità metodologica occorre mostrare un vantaggio rispetto a campionamento stratificato e metodi esistenti, e preferibilmente replicarlo su un secondo task testuale. Tra queste proposte, è quella la cui originalità richiede la verifica bibliografica più attenta.

**Fattibilità:** media-alta; richiede competenze statistiche e un audit esteso iniziale. Il risparmio va dimostrato per audit futuri, senza nascondere il costo del riferimento usato nella ricerca.

## 5. Scelta pratica e requisiti comuni

| Proposta | Oggetto spiegato | Risorsa critica | Se l'ipotesi non regge |
|---|---|---|---|
| 1. Adapter e fine-tuning | Cambiamento del modello dopo l'addestramento | GPU e repliche con più seed | Si delimita il valore interpretativo di rango e norme, purché l'adattamento produca effetti misurabili. |
| 2. Esempi nel prompt | Influenza dei casi dimostrativi | Numero di inferenze e retrieval controllato | Si mostra quando similarità o attribuzioni non aiutano a selezionare esempi; effetti nulli vanno stimati con precisione. |
| 3. Correzione dei concetti | Affidabilità della spiegazione incorporata nella pipeline | Annotazioni indipendenti | Si documenta quando la trasparenza non rende utile la correzione, senza forzare conclusioni cliniche. |
| 4. Audit adattivo | Affidabilità ed efficienza della valutazione XAI | Statistica e confronto esteso di riferimento | Si identifica quando l'adattività non ripaga il costo o peggiora la selezione. |

Per tutte: split per partecipante, selezione di prompt/soglie sul development, target PHQ-8 esplicitato, incertezza a livello di persona e seed. Lo score del modello non è automaticamente una probabilità calibrata. Se si riportano log-loss o Brier score, usare probabilità definite e valutarne la calibrazione separatamente. Le varianti sintetiche non ereditano automaticamente la correttezza della label originale.

Il lessico non è un requisito di queste proposte. Può servire ad annotare o campionare, ma non determina il contributo. Non è necessario svolgere tutte e quattro le tesi: ogni scheda definisce una direzione alternativa.

## 6. Il professor Mark J. Carman sarebbe un relatore adatto?

**Valutazione: sì, il matching scientifico è forte, soprattutto per la proposta 1; è concreto anche per 2 e 4.** È una valutazione basata sui lavori pubblici, non una conoscenza della sua disponibilità, delle risorse attuali o del suo stile di supervisione.

Il [profilo Scholar indicato](https://scholar.google.com/citations?hl=en&user=fcPONTQAAAAJ&view_op=list_works&sortby=pubdate) non è stato accessibile dal motore di consultazione. Ho quindi verificato identità, interessi e coautorie tramite [profilo istituzionale Polimi/AIRIC](https://airic.polimi.it/it/expert-panel/), [scheda docente](https://onlineservices.polimi.it/manifesti/manifesti/controller/ricerche/RicercaPerDocentiPublic.do?evn_CARICHE_ISTITUZIONALI=evento&k_doc=690519&lang=IT), [ACL Anthology](https://aclanthology.org/people/mark-carman/) e pagine originali dei paper. La lista seguente è mirata al tema, non l'intera produzione del docente.

### Evidenze del matching

| Lavoro con Carman fra gli autori | Competenza documentata | Legame con le proposte |
|---|---|---|
| [L1RA, 2025](https://aclanthology.org/2025.icnlsp-1.35/) | Distribuzione del rango degli adapter e diagnostica dell'adattamento | **Diretto per 1:** verificare con interventi quanto la distribuzione del rango anticipi il contributo effettivo. |
| [IPE, BlackboxNLP 2025](https://aclanthology.org/2025.blackboxnlp-1.30/) | Identificazione di percorsi di calcolo e loro ablazione/sostituzione | **Diretto per interpretabilità meccanicistica:** utile per un'estensione di 1 e per brainstorming02, proposta 4. Il paper documenta anche limiti di scalabilità. |
| [InTraVisTo, 2025](https://arxiv.org/abs/2507.13858) | Visualizzazione degli stati interni e dei flussi tra componenti di Transformer | Supporto all'esplorazione del modello e alla presentazione dei risultati; una visualizzazione resta distinta dalla verifica causale. |
| [Medical Information Extraction with Large Language Models, 2024](https://aclanthology.org/2024.icnlsp-1.47/) | LLM aperti di piccola scala, prompting ed estrazione da testi medici con riferimenti alle righe di origine | **Buon legame per 2 e 3:** esperienza con input clinici e output strutturati; il lavoro riguarda farmaci e timeline, non dimostra specializzazione su DAIC o depressione. |
| [Efficient Benchmarking of NLP APIs using Multi-armed Bandits, 2017](https://aclanthology.org/E17-1039/) | Valutazione NLP con meno interrogazioni dei sistemi | **Diretto sul metodo per 4:** adattare la selezione efficiente a misure di fedeltà delle spiegazioni richiede un nuovo disegno. |

La [pagina istituzionale](https://airic.polimi.it/it/expert-panel/) riporta inoltre interessi in information retrieval, machine learning, crowdsourcing e text analytics. Perciò il matching non dipende soltanto da una generica esperienza con l'AI.

### Matching per argomento

| Argomento | Matching scientifico stimato | Motivazione e integrazione utile |
|---|---|---|
| 03–1: spiegare l'adattamento LoRA | **Molto alto** | L1RA è un punto di partenza diretto; IPE offre competenze sugli interventi. Concordare un perimetro sostenibile sul modello scelto. |
| 03–2: influenza degli esempi in-context | **Alto** | Collegamento fra retrieval, prompting medico e interpretabilità. Non ho identificato nei lavori consultati un suo metodo specifico di demonstration attribution. |
| 03–3: concetti e correggibilità | **Buono, meno diretto** | Vicino all'estrazione medica e alla XAI; per validare i concetti è utile un collaboratore clinico. I paper consultati non provano una specializzazione sua nel concept bottleneck leakage. |
| 03–4: audit adattivo degli spiegatori | **Alto sul metodo** | Precedente esplicito sui bandit per benchmarking NLP. Da verificare interesse attuale e solidità della nuova domanda XAI. |
| 02–4: meccanismi interni sintomo-contesto | **Molto alto sul metodo** | IPE e InTraVisTo sono riferimenti pertinenti; annotazione e interpretazione clinica rimangono competenze da integrare. |
| 02–1 e 02–2: citazioni e protocollo | **Alto sul metodo** | Buon raccordo fra estrazione trasparente e interventi interni; nessuna evidenza consultata permette di dare per acquisita esperienza specifica sul dataset. |

### Quale proposta portargli per prima

Porterei **03–1**, formulata come domanda aperta: *«La capacità assegnata agli adapter è un indicatore affidabile di ciò che il fine-tuning cambia nelle decisioni? Vorrei verificarlo con ablazioni e ripristino di sottoinsiemi, usando interviste DAIC come caso di studio.»* Il collegamento a L1RA è preciso e il contributo è valutabile anche se il rango si rivela poco informativo.

Come alternativa con minori esigenze di addestramento porterei **03–2**. La proposta **03–4** è adatta se si preferisce una tesi metodologica sulla valutazione, ma va ristretta con lui per evitare una semplice applicazione dei bandit. Per **03–3** cercherei fin dall'inizio supporto sulle annotazioni cliniche.

Per decidere davvero se sia il relatore giusto, restano da chiarire con lui: interesse per la domanda specifica, possibilità di supervisione regolare, accesso a calcolo e dati, eventuali collaboratori del gruppo, disponibilità di una competenza clinica quando necessaria. Le pubblicazioni sostengono l'adeguatezza scientifica; non permettono di garantire questi aspetti pratici.

## 7. Portata della verifica bibliografica

Le ricerche hanno incrociato termini quali `DAIC LoRA interpretability adapter`, `DAIC demonstration attribution`, `concept bottleneck leakage depression`, `budgeted explainability evaluation bandits`, oltre ai titoli dei lavori di Carman. Ho consultato record e abstract originali; per IPE e Medical Information Extraction anche sezioni del PDF, e per il confronto sul fine-tuning il testo HTML di Wang et al. I lavori più vicini devono ancora essere confrontati integralmente con il protocollo definitivo, inclusi supplementi e codice.

**Rivendicazioni eliminate dopo la ricerca:** primo audit del fine-tuning, prima attribuzione degli esempi ICL, primo modello a concetti per testo, prima combinazione fra bandit e explainability. Le rivendicazioni residue riguardano le verifiche specifiche descritte nelle quattro schede. Se un precedente copre già domanda, confronto e metrica proposti, cambiare il confronto sperimentale; il solo passaggio a DAIC non garantisce originalità.
