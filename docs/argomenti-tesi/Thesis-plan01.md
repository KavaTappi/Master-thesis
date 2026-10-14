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
