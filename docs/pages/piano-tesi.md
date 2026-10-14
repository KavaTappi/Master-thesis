---
title: Piani di tesi — proposte 00 e 01
permalink: /piano-tesi-new/
description: Due proposte per la discussione con il relatore, dalla scoperta di circuiti guidata da lessici all’audit degli indizi d’intervista.
---

# Piani di tesi per la discussione con il relatore

[Proposta 00 — lessici e circuiti](#thesis-plan00) · [Proposta 01 — audit dell’intervista](#thesis-plan01)

<a id="thesis-plan00"></a>

# Thesis-plan00 — Guida lessicale alla scoperta dei circuiti

**Proposta prioritaria per la discussione con il relatore.** Sintesi di `proposta-carman00.md`, aggiornata con i chiarimenti su fedeltà, costo e piani alternativi. Le ipotesi e i titoli finali non anticipano risultati sperimentali.

## 1. Titolo

### 1.1 Titolo proposto

**Il valore della guida lessicale nella scoperta di circuiti degli LLM: costo, fedeltà e generalizzazione su interviste DAIC-WOZ.**

### 1.2 Domanda centrale

A parità di risorse e di dimensione del circuito, un lessico permette di trovare circuiti che riproducono più fedelmente le predizioni di un LLM sul PHQ-8, mantenendo questa fedeltà su partecipanti nuovi e parafrasi?

### 1.3 Obiettivi e contributo atteso

- Progettare una priorità lessicale per la ricerca di circuiti e confrontarla con guide globali, casuali e basate sulla salienza.
- Misurare il valore aggiunto del lessico in termini di costo, fedeltà e trasferimento.
- Produrre un protocollo riproducibile che documenti anche condizioni di inefficacia e limiti della guida.

### 1.4 Possibili casi d’uso

- **Audit con risorse limitate:** scegliere una strategia di ricerca che raggiunga una fedeltà prefissata con meno calcolo.
- **Verifica delle spiegazioni interne:** controllare se un circuito resta valido quando cambia il modo di esprimere lo stesso contenuto.
- **Progettazione di strumenti di interpretabilità:** stabilire quando convenga introdurre conoscenza lessicale esterna.

## 2. Ipotesi verificabili e domande di ricerca

| Domanda/Ipotesi | Macro area | Esperimento | Esito positivo | Esito negativo |
|---|---|---|---|---|
| **H0 — Il sistema è studiabile con le risorse disponibili?** | Fattibilità | E0–E1: dati, lessico, modello e controlli sugli interventi. | Input e categorie utilizzabili, comportamento non degenere e interventi corretti entro il budget. | Dati insufficienti, output non informativi o interventi non validabili: attivare il piano alternativo prima dell’audit esteso. |
| **H1 — La guida lessicale migliora il compromesso costo–fedeltà?** | Efficienza | E2: confronto delle curve di errore a dimensione del circuito controllata; E5: robustezza. | Errore inferiore allo stesso costo, oppure costo inferiore alla stessa tolleranza, includendo lo screening. | Nessun vantaggio pratico o costo aggiuntivo superiore al risparmio. |
| **H2 — Il vantaggio dipende dall’informazione lessicale?** | Specificità | E0, E2 ed E5: confronto con maschere casuali abbinate e salienza, a copertura comparabile. | La guida lessicale supera i controlli secondo una misura prefissata. | Il vantaggio è riprodotto dai controlli o spiegato da frequenza, posizione e numero dei token selezionati. |
| **H3 — I circuiti restano fedeli oltre gli esempi di scoperta?** | Generalizzazione | E3: nuove persone e parafrasi; E6: eventuale replica esterna. | L’incremento di errore resta entro una tolleranza fissata sul development e il vantaggio si conserva. | La fedeltà o il vantaggio si deteriorano, soprattutto fuori dal vocabolario iniziale. |
| **H4 — Il circuito riproduce la sensibilità contestuale del modello?** | Contesto | E4: negazione, temporalità e persona cui è riferita l’esperienza. | Il circuito riproduce le variazioni dello score del modello completo. | Il circuito perde differenze contestuali presenti nel modello. |

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

Una GPU Colab Pro può servire per il pilot; la configurazione effettiva determina i budget. Partire da un modello intorno a 0,5B e input di 256–512 token come prova tecnica, senza assumere che questa configurazione sia già sufficiente per il task finale.

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

**Riferimento della fedeltà:** l’output del modello completo sulla stessa intervista, non il PHQ-8 osservato. Indicando con `p_M(x)` l’output originale e con `p_C(x; r)` quello ottenuto dal circuito sotto un riferimento di intervento fissato:

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
| **E0 — Preparazione di dati e lessico:** split, copertura, accordo e qualità del matcher. | H0, H2: affidabilità della risorsa e dei confronti. |
| **E1 — Pilot del modello e degli interventi:** prestazioni, replica tecnica, grafo completo/vuoto, memoria e tempi. | H0: condizioni per proseguire e scelta del perimetro sostenibile. |
| **E2 — Ricerca a budget controllato:** curve costo–fedeltà per strategie e dimensioni del circuito. | H1, H2: efficienza e contributo specifico del lessico. |
| **E3 — Trasferimento a nuove persone e parafrasi:** circuiti congelati e distanza dagli output originali. | H3: generalizzazione oltre dati e vocabolario di scoperta. |
| **E4 — Variazioni contestuali:** negazione, tempo e persona, con revisione delle coppie. | H4: riproduzione della sensibilità contestuale del modello. |
| **E5 — Robustezza del risultato:** repliche, riferimenti alternativi e controlli di copertura/dimensione. | H1, H2: verifica che il vantaggio non sia un artefatto della selezione o del patching. |
| **E6 — Replica esterna, opzionale:** seconda configurazione o task compatibile. | H3: portata delle conclusioni oltre il primo esperimento. |

## 5. Titoli in base all’output degli esperimenti

| Esiti osservati | Titolo finale possibile |
|---|---|
| E2 mostra un vantaggio, E3 ne conferma il trasferimento ed E5 ne sostiene la robustezza. | **Scoperta efficiente e generalizzabile di circuiti negli LLM mediante guida lessicale: uno studio su DAIC-WOZ.** |
| E2 mostra un vantaggio, ma E3 rileva un deterioramento sulle parafrasi. | **Efficienza e limiti di generalizzazione della guida lessicale nella scoperta di circuiti degli LLM.** |
| E2 ed E5 mostrano un beneficio trascurabile rispetto ai controlli, con intervalli sufficientemente stretti. | **Valore e limiti della guida lessicale nella scoperta di circuiti: una valutazione controllata su DAIC-WOZ.** |
| Le differenze restano incerte per variabilità o campione limitato. | **Valutazione sperimentale della guida lessicale nella ricerca di circuiti degli LLM su DAIC-WOZ.** |
| Si adotta il piano alternativo sugli archi, con verifiche di fedeltà e trasferimento. | **Guida lessicale alla selezione di circuiti negli LLM: compattezza, fedeltà e generalizzazione.** |
| Il task PHQ-8 non supera il pilot e si completa lo studio su un task diverso. | **Il valore della guida lessicale nella scoperta di circuiti per [task effettivamente studiato].** |

I titoli devono riportare il corpus e il comportamento effettivamente analizzati; se si usano estratti, precisarlo nel sottotitolo o nell’abstract. Un pilot non riuscito, da solo, non costituisce il completamento della tesi.

---

<a id="thesis-plan01"></a>

# Thesis-plan01 — Audit degli indizi d’intervista

**Proposta alternativa per la discussione con il relatore.** Sintesi dell’attuale `piano-tesi.md`. L’oggetto è la dipendenza delle predizioni da evidenze del partecipante e indizi del protocollo, con verifica delle componenti interne coinvolte.

## 1. Titolo

### 1.1 Titolo proposto

**Evidenze del partecipante e indizi d’intervista nelle predizioni degli LLM: un audit meccanicistico guidato da lessici su DAIC-WOZ.**

### 1.2 Domanda centrale

In quale misura un LLM usa espressioni del partecipante e indizi del protocollo per predire l’etichetta PHQ-8, quali componenti interne contribuiscono alle dipendenze osservate e quanto il lessico aiuta a individuarle?

### 1.3 Obiettivi e contributo atteso

- Distinguere segnale disponibile nelle domande, suo uso effettivo nel dialogo e contributi delle componenti interne.
- Valutare se il lessico permette di selezionare interventi più informativi dei controlli.
- Delimitare la trasferibilità degli effetti fra persone e, se i dati lo consentono, protocolli diversi.

### 1.4 Possibili casi d’uso

- **Revisione di sistemi che analizzano interviste:** individuare formulazioni delle domande cui le predizioni risultano sensibili.
- **Passaggio a un diverso protocollo:** verificare quali dipendenze rimangano stabili fra interviste guidate e autonome.
- **Analisi degli errori:** distinguere gli effetti delle risposte del partecipante da quelli della cornice d’intervista.

I risultati supportano un audit sperimentale; non certificano idoneità diagnostica.

## 2. Ipotesi verificabili e domande di ricerca

| Domanda/Ipotesi | Macro area | Esperimento | Esito positivo | Esito negativo |
|---|---|---|---|---|
| **H0 — Dati, lessico e modello permettono un audit informativo?** | Fattibilità | E0–E1: qualità dei dati, annotazioni, prestazioni e controlli sensibili. | Categorie affidabili, score non degenere e pipeline verificabile. | Dati insufficienti o modello non studiabile: ridimensionare il task prima delle conclusioni. |
| **H1 — Le domande contengono segnale predittivo autonomo?** | Segnale nel corpus | E2: Ellie-only, participant-only e baseline semplici su persone separate. | Le domande consentono predizioni superiori ai controlli. | Nessuna evidenza di segnale autonomo; non esclude il loro uso come contesto. |
| **H2 — Il modello sul dialogo completo usa gli indizi manipolati?** | Dipendenza dal protocollo | E3: modifiche coerenti delle domande, risposte invariate e controlli abbinati. | Effetto sullo score replicabile e superiore ai controlli. | Effetto trascurabile entro la precisione prefissata; con intervalli ampi l’esito è inconcludente. |
| **H3 — Alcuni siti interni contribuiscono alla dipendenza osservata?** | Meccanismi interni | E4: patching bidirezionale e ablazioni, su siti selezionati fuori dal test. | Effetti direzionali superiori ai controlli, replicati su nuove persone. | Nessun sito confermato; se E3 è privo di contrasto, la verifica è non applicabile a quelle coppie. |
| **H4 — Il lessico migliora la selezione di casi e siti da verificare?** | Utilità del lessico | E5: selezione lessicale, casuale abbinata e attribuzioni, a budget controllato. | Più effetti confermati su dati nuovi o costo inferiore per trovarli. | Nessun vantaggio pratico, oppure beneficio limitato alla classificazione degli span. |
| **H5 — Gli effetti si trasferiscono a un altro protocollo?** | Generalizzazione | E6: replica su sessioni autonome E-DAIC disgiunte. | Effetti compatibili nelle condizioni confrontabili. | Eterogeneità o mancato trasferimento; dati insufficienti rendono l’ipotesi non valutabile. |
| **H6 — Una mitigazione mirata riduce la dipendenza?** | Robustezza, opzionale | E7: intervento su dati o modello e ripetizione dell’audit. | Dipendenza ridotta con prestazioni preservate entro una tolleranza prefissata. | Dipendenza invariata, perdita predittiva eccessiva o effetto non replicabile. |

H2–H4 costituiscono il nucleo; H1 descrive il segnale disponibile. H5 e H6 sono estensioni. Un risultato positivo di H2 dimostra sensibilità agli indizi studiati: chiamarla *shortcut* richiede escludere che si tratti di contesto legittimamente informativo.

## 3. Stack tecnico

### 3.1 Dataset

- **DAIC-WOZ:** trascrizioni inglesi, ruoli di Ellie e partecipante, etichetta ufficiale associata al PHQ-8. [Fonte ufficiale](https://dcapswoz.ict.usc.edu/).
- **E-DAIC, opzionale:** utilizzare sessioni autonome solo dopo verifica di sovrapposizioni, trascrizioni, ruoli e label. Documentare differenze nella qualità delle trascrizioni e nella composizione dei campioni. [Manuale E-DAIC](https://dcapswoz.ict.usc.edu/wwwedaic/E-DAIC%20Manual.pdf).
- Split per partecipante e separazione fra eventuale training, scelta di prompt/soglie, localizzazione dei siti e conferma finale. Tutte le varianti di un’intervista rimangono nello stesso split.
- Le perturbazioni devono conservare referenti e coerenza domanda–risposta. Rimuovere una domanda cui segue «yes» non costituisce un controllo adeguato.
- Se si usano estratti per limiti di memoria, selezionarli con una regola indipendente dagli esiti e dichiarare che l’audit riguarda tali input.

### 3.2 Task del modello e ambiente

**Task principale:** classificazione binaria PHQ-8 con un LLM aperto e congelato durante l’audit. Un adattamento LoRA sul training è possibile se necessario; non costituisce da solo il contributo scientifico.

Confrontare quattro condizioni:

| Condizione | Che cosa consente di osservare |
|---|---|
| Participant-only | Segnale disponibile nelle risposte. |
| Ellie-only | Segnale disponibile nelle domande. |
| Dialogo completo | Comportamento del predittore oggetto dell’audit. |
| Dialogo con modifiche controllate | Sensibilità del medesimo predittore agli indizi manipolati. |

Per E2 usare anche classificatori semplici, dichiarando quali siano addestrati separatamente per condizione. La loro prestazione misura il segnale nel corpus. Per E3 usare invece lo stesso checkpoint e lo stesso prompt, modificando soltanto il testo previsto dal protocollo.

Definire `s(x)` come differenza fra i logits dei due simboli di classe; congelare la soglia sul development. Baseline: classe maggioritaria, TF-IDF e conteggi lessicali con regressione logistica regolarizzata.

**Ambiente previsto:** Python, PyTorch, Transformers, PEFT se necessario, strumenti di patching compatibili con il modello, scikit-learn e pandas. Partire da un modello piccolo e da pochi layer; valutare tempi e memoria prima di approfondire teste di attenzione o MLP.

### 3.3 Explainability, lessico e confronti

#### 3.3.1 Sviluppo del lessico specializzato

Costruire una risorsa inglese con tre famiglie distinte:

| Famiglia | Esempi di categorie | Funzione |
|---|---|---|
| Espressioni sintomatologiche | Umore, interesse, sonno, energia, concentrazione | Selezionare evidenze del partecipante e controlli pertinenti. |
| Storia clinica e trattamento | Diagnosi riferite, terapia, farmaci, episodi passati | Distinguere contenuti storici e informazioni attuali. |
| Indizi d’intervista | Approfondimenti, domande su trattamento, transizioni tematiche | Individuare formulazioni del protocollo da sottoporre ad audit. |

Un LLM può generare candidati e parafrasi da definizioni e dati di training. Persone revisionano inclusione e categoria; conservare fonte, prompt, versione, regole di matching e modifiche. Annotare un campione separato per categoria, speaker, negazione, temporalità e persona cui l’esperienza si riferisce.

Misurare accordo fra annotatori, precisione/recall e copertura del matcher. Il lessico non entra come feature nel predittore principale; alimenta solo una baseline distinta e la selezione degli interventi. Una categoria non rende automaticamente un’espressione informativa o spuria.

#### 3.3.2 Localizzazione e verifica interna

1. **Occlusione e attribuzioni mediante gradienti:** individuare span candidati, mantenendo la cancellazione grezza come stress test quando compromette il dialogo.
2. **Coppie controllate:** riformulare indizi di Ellie mantenendo le risposte, con revisione della coerenza e controlli di lunghezza, posizione e intensità della modifica. Studiare separatamente le perturbazioni delle evidenze del partecipante.
3. **Activation patching bidirezionale:** trasferire attivazioni fra originale e variante, inizialmente in pochi layer e alla posizione finale di predizione. Per token intermedi richiedere allineamento esplicito.
4. **Ablazioni mirate e controlli:** verificare i siti selezionati su persone nuove; distinguere l’effetto di siti specifici da un disturbo generico del modello.

La scelta di coppie, siti e riferimenti condiziona le conclusioni. [Zhang e Nanda, 2024](https://arxiv.org/abs/2309.16042). Probe lineari e spiegazioni verbali possono generare ipotesi, ma non sono necessari al nucleo e non provano l’uso causale dell’informazione.

#### 3.3.3 Confronti e posizionamento

Per valutare il lessico, confrontare selezione lessicale, casuale abbinata e attribuzioni generiche, usando budget comparabili per span, siti e interventi. La misura principale è la frequenza di effetti interni confermati su dati nuovi; la sola frequenza di cambiamenti dell’output può premiare perturbazioni troppo distruttive.

Il segnale dell’intervistatore ha un precedente diretto in [Burdisso et al., 2024](https://arxiv.org/abs/2404.14463); esistono inoltre lavori sulla [mitigazione del bias dell’intervistatore](https://aclanthology.org/2025.findings-emnlp.650/). Il contributo candidato è distinguere disponibilità, uso nel LLM e contributo interno, misurando il vantaggio specifico del lessico.

### 3.4 Analisi quantitative

- **Predizione:** macro-F1, precision/recall della classe positiva, balanced accuracy e confronto con baseline.
- **Lessico:** accordo fra annotatori, precisione/recall per categoria, copertura e analisi degli errori contestuali.
- **Dipendenza comportamentale:** `Δs = s(originale) − s(variante)`, frequenza dei cambi di classe e confronto con perturbazioni abbinate.
- **Contributo interno:** `P = s(variante con patch) − s(variante)`, direzione dell’effetto e distanza residua dallo score sorgente. Il rapporto `P/Δs` si usa solo sopra una soglia di contrasto fissata sul development; riportare anche gli effetti grezzi e i casi esclusi.
- **Valore della guida:** quota di effetti confermati, replica fra persone e costo per ottenere una conferma, includendo selezione e verifica.
- **Incertezza:** confronti appaiati, bootstrap per partecipante, soglie di rilevanza prefissate e repliche. Non equiparare molte varianti di una persona a molti soggetti indipendenti.

Un’elevata prestazione Ellie-only non prova che il predittore sul dialogo completo usi gli stessi indizi. Se E3 non produce contrasti informativi, E4 su quelle coppie non dimostra assenza di meccanismi. Se E3 è informativo ma E4 fallisce, resta il risultato comportamentale senza una localizzazione interna confermata.

### 3.5 Perimetro e piano alternativo

Il nucleo richiede E0–E5. E6 dipende dai dati; E7 si attiva solo dopo una dipendenza replicabile. Se il calcolo è oneroso, limitare il patching a pochi layer e alla posizione finale. Se gli effetti delle domande sono trascurabili con controlli sensibili, approfondire le evidenze del partecipante come domanda distinta, senza sostenere che siano state trovate shortcut. Se modello o interventi non superano il pilot, riformulare il task o la portata dell’audit con il relatore.

## 4. Esperimenti da realizzare

| Nome esperimento | A quale Ipotesi/Domanda risponde |
|---|---|
| **E0 — Dati e lessico:** split, sovrapposizioni, annotazioni contestuali e qualità del matcher. | H0, H4: risorsa utilizzabile e base per confrontare le selezioni. |
| **E1 — Modello predittivo e controlli sensibili:** prestazioni, stabilità e risposta a modifiche pertinenti del partecipante. | H0: condizioni per interpretare l’audit. |
| **E2 — Segnale nelle domande:** Ellie-only, participant-only e baseline su persone separate. | H1: disponibilità del segnale nel corpus. |
| **E3 — Uso degli indizi nel dialogo:** variazioni coerenti delle domande e controlli abbinati. | H2: dipendenza effettiva del predittore dagli indizi manipolati. |
| **E4 — Verifica interna:** patching bidirezionale e ablazioni dei siti candidati. | H3: contributo dei siti alla differenza osservata in E3. |
| **E5 — Valore aggiunto del lessico:** confronto con casuale abbinato e attribuzioni a budget controllato. | H4: utilità nella selezione di casi e siti confermati. |
| **E6 — Trasferimento a E-DAIC, opzionale:** replica su sessioni autonome disgiunte e confrontabili. | H5: trasferibilità e differenze tra protocolli. |
| **E7 — Mitigazione mirata, opzionale:** intervento sui segnali individuati e ripetizione dell’audit. | H6: riduzione della dipendenza con prestazioni preservate. |

## 5. Titoli in base all’output degli esperimenti

| Esiti osservati | Titolo finale possibile |
|---|---|
| E3 ed E4 confermano la dipendenza e i siti; E5 mostra un vantaggio del lessico. | **Audit meccanicistico guidato da lessici della dipendenza di un LLM dagli indizi d’intervista su DAIC-WOZ.** |
| E3 conferma la sensibilità, ma E4 non identifica siti robusti. | **Sensibilità delle predizioni PHQ-8 di un LLM agli indizi d’intervista: un audit controllato su DAIC-WOZ.** |
| E3 ed E4 sono positivi, ma E5 non mostra un vantaggio del lessico. | **Analisi meccanicistica della dipendenza di un LLM dagli indizi d’intervista.** |
| E3 rileva effetti trascurabili con intervalli stretti e controlli sensibili; E4 sulle domande è non applicabile. | **Limiti della dipendenza dagli indizi d’intervista nelle predizioni di un LLM su DAIC-WOZ.** |
| E3 resta inconcludente per variabilità o qualità delle perturbazioni. | **Sensibilità alle formulazioni d’intervista nelle predizioni PHQ-8: uno studio sperimentale.** |
| E6 evidenzia effetti diversi nei due protocolli. | **Dipendenza dal protocollo e trasferibilità degli effetti di intervento negli LLM su DAIC-WOZ ed E-DAIC.** |
| E7 conferma una riduzione della dipendenza preservando le prestazioni. | **Analisi e mitigazione della dipendenza dagli indizi d’intervista in un LLM per la predizione PHQ-8.** |

Citare E-DAIC nel titolo solo se è stato effettivamente valutato. Usare «shortcut» solo quando i controlli sostengono tale interpretazione. Se il pilot fallisce, occorre ridefinire il progetto: cambiare il titolo non sostituisce gli esperimenti mancanti.
