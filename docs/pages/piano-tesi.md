---
title: Piano della tesi
permalink: /
description: Proposta di tesi su LLM, DAIC-WOZ, lessici ed explainability meccanicistica, con piano di riserva.
---

[Diario settimanale]({{ '/settimane/' | relative_url }}) · [Bibliografia ragionata]({{ '/bibliografia/' | relative_url }})

<!-- Contenuto informativo tratto da docs/argomenti-tesi/brainstorming00.md. -->

# Interpretabilità meccanicistica degli LLM nella predizione della depressione: un audit guidato da lessici su DAIC-WOZ ed E-DAIC

Proposta di tesi magistrale in Computer Science applicata alla psichiatria. 

## 1. Domanda centrale e idea della tesi

**In quale misura le predizioni di un LLM sulle etichette di depressione di DAIC-WOZ ed E-DAIC dipendono da espressioni sintomatologiche del partecipante oppure da indizi del protocollo d'intervista, e quali componenti interne mediano queste dipendenze?**

La proposta combina tre elementi:

- costruire con un LLM un lessico specializzato e revisionato da esseri umani;
- usarlo per individuare categorie di evidenze e progettare confronti controllati;
- intervenire sulle attivazioni del modello predittivo per verificare se le componenti individuate contribuiscono alle sue decisioni.

Il lessico diventa quindi uno **strumento per formulare e verificare ipotesi sul modello**, oltre che una baseline interpretabile.

---
### 1.1 - Dove collocare l'originalità cont'd

L'**oggetto scientifico principale** è dunque distinguere tre livelli che spesso vengono confusi: 
- **segnale disponibile nel corpus** (E2),
- **segnale effettivamente usato dal predittore** (E3)
- **componenti interne che mediano l'effetto osservato** (E4).
  
Il contributo metodologico aggiuntivo è verificare se il lessico aiuta a trovare tali effetti più efficacemente dei controlli (E5). Una categoria lessicale non è, di per sé, una feature interna: nel protocollo principale **il predittore riceve il dialogo, non il lessico come feature aggiuntiva**. I conteggi lessicali possono alimentare una baseline distinta.

Un esempio delimita la conclusione cercata: se una domanda di approfondimento sulla storia clinica viene parafrasata mantenendo una risposta autosufficiente del partecipante, un cambiamento replicabile dello score indica sensibilità del modello a quella modifica. Se il patching di attivazioni selezionate trasferisce parte della differenza fra le due versioni, si ottiene evidenza che quei siti contribuiscono a **quella** dipendenza. Solo con controlli semantici e contestuali adeguati si può discutere se la dipendenza sia una scorciatoia indesiderata, anziché un uso legittimo del contesto. Se il lessico permette di individuare più casi e siti confermati dello stesso tipo, con lo stesso budget dei metodi alternativi, se ne dimostra l'utilità per progettare l'audit; il semplice fatto di avere generato il lessico non basta.

La verifica su sessioni autonome E-DAIC disgiunte (E6) serve a stimare **quanto il risultato si trasferisce** cambiando protocollo, se trascrizioni e label lo permettono. 

La mitigazione (E7) è un'estensione, dato che [Zhang e Poellabauer (2025)](https://aclanthology.org/2025.findings-emnlp.650/) hanno già proposto un metodo per ridurre il bias dell'intervistatore. 

La rivendicazione finale va calibrata sugli esiti:
esempio:
- E2 positivo da solo replica un fenomeno noto;
- E3 positivo aggiunge evidenza sull'uso nel LLM;
-  E4 positivo aggiunge una verifica interna delimitata; E5 positivo mostra il valore specifico del lessico.
-  E3 è nullo con misure informative, l'esito circoscrive la dipendenza del modello testato;
-   E4 sulle domande non è allora applicabile e non si può rivendicare che E5 abbia trovato meglio
quegli effetti interni.

### 1.3 - Possibili casi d'uso dei risultati

Questi sono scenari concreti per usare i **risultati della ricerca**; non presuppongono che il modello studiato sia già idoneo a prendere decisioni cliniche.

1. **Revisione di un assistente per interviste semistrutturate.** Un team sta sviluppando un sistema di supporto per analizzare trascrizioni. Se una categoria di domande di approfondimento produce effetti sistematici sulle predizioni, può decidere di riformulare quelle domande nei dati di addestramento, costruire esempi di controllo e ripetere l'audit dopo la modifica. Se le domande non producono effetti rilevanti, può concentrare la revisione su altre fonti di errore. Esistono già approcci che cercano di ridurre il bias dell'intervistatore tramite addestramento adversarial: [Zhang e Poellabauer (2025)](https://aclanthology.org/2025.findings-emnlp.650/).
2. **Passaggio da interviste guidate da persone a interviste con agente autonomo.** Un laboratorio vuole verificare un modello allenato su colloqui WoZ anche su sessioni autonome. La mappa degli indizi e delle componenti interne può indicare quali effetti sopravvivono al cambiamento del protocollo e quali richiedono nuova verifica. Per attribuire le differenze al protocollo bisogna controllare sovrapposizione delle sessioni, qualità ASR e distribuzione delle label; il manuale [E-DAIC](https://dcapswoz.ict.usc.edu/wwwedaic/E-DAIC%20Manual.pdf) descrive entrambi i tipi di sessione.
3. **Esame di errori e spiegazioni da parte di ricercatori e clinici.** In una revisione sperimentale, il sistema assegna un punteggio alto a una trascrizione. L'audit può mostrare se il punteggio è sensibile a una domanda di Ellie sulla terapia, a ciò che il partecipante ha detto sui sintomi, o a entrambi. La categoria del lessico rende il caso facile da cercare e discutere; il patching verifica una dipendenza del modello. L'interpretazione del caso resta compito delle persone e non trasforma un'etichetta PHQ-8 in una diagnosi.
4. **Strumento riutilizzabile per altri studi su interviste.** Se il lessico seleziona perturbazioni efficaci e replicabili meglio dei controlli a parità di budget, pubblicarne definizioni, criteri di revisione e protocollo sperimentale può aiutare altri gruppi a verificare modelli su colloqui simili. Anche un risultato negativo è utile: chiarisce quando una categoria lessicale intuitiva non aiuta a spiegare il calcolo dell'LLM.


## 2. Ipotesi verificabili

| Ipotesi | Esperimento discriminante | Conclusione ammissibile |
|---|---|---|
| H1 — Il modello dipende da indizi del protocollo | Modificare indizi di Ellie mantenendo il contenuto informativo del partecipante, con controlli di coerenza e lunghezza | Dipendenza dagli indizi manipolati; evidenza di shortcut solo se il disegno esclude spiegazioni alternative plausibili |
| H2 — Alcune componenti interne mediano tale dipendenza | Activation patching e ablazioni mirate, confrontati con componenti di controllo | Contributo causale interno nel modello, per gli input e gli interventi studiati |
| H3 — Il lessico aiuta a progettare l'audit | Confrontare span/interventi scelti tramite lessico, selezione casuale abbinata e attribuzioni, a parità di budget | Utilità del lessico nel trovare effetti interni replicabili; l'LLM predittivo riceve sempre il dialogo |
| H4 — Gli effetti si replicano cambiando protocollo | Ripetere gli esperimenti su sessioni E-DAIC autonome e disgiunte | Trasferibilità degli effetti nel campione esaminato; eventuale eterogeneità tra protocolli |

H1–H3 costituiscono il nucleo. H4 dipende dalla disponibilità effettiva di trascrizioni, ruoli dei parlanti e label utilizzabili. Le ipotesi possono essere smentite senza invalidare l'intero lavoro.

## 3. Dataset e task

### 3.1 - Target operativo

Usare come task principale la predizione dell'**etichetta binaria ufficiale associata al PHQ-8**, verificandone la codifica nella release disponibile. 

#### 3.1.1 - Confronti principali

- **Participant-only:** solo risposte del partecipante.
- **Ellie-only:** solo domande dell'intervistatore, come controllo del segnale predittivo del protocollo.
- **Dialogo completo:** domande e risposte con ruoli espliciti.
- **Dialogo con modifiche controllate:** neutralizzazione o parafrasi di indizi selezionati tramite lessico, più modifiche di controllo abbinate per lunghezza e posizione.

Confrontare anche baseline semplici: classe maggioritaria, TF-IDF con regressione logistica e conteggi per categoria del lessico con regressione logistica regolarizzata, LLM deve essere utile effettivamente (passaggio facoltativo, posso presuppporlo come precondizione per la domanda di ricerca)

**Importante:** cancellare una domanda può rendere incomprensibile una risposta come «yes». Le perturbazioni principali devono conservare il referente e la coerenza del dialogo, con verifica umana su un sottoinsieme. **La cancellazione grezza può restare uno stress test, senza attribuirle automaticamente un significato causale specifico.**

**Importante 2** : Un'alta performance Ellie-only segnala informazione disponibile nelle domande; non prova che il modello sul dialogo completo la sfrutti. Analogamente, una diminuzione dopo aver eliminato sintomi informativi è attesa e non dimostra uno shortcut. Quest'ultimo richiede evidenza di dipendenza da indizi non necessari al target definito, sostenuta da confronti controllati.


### 3.2 - Costruzione del lessico specializzato

### Struttura proposta

Costruire un lessico in inglese, coerente con la lingua delle interviste, con tre famiglie distinte:

| Famiglia | Esempi di categorie da definire | Funzione nell'audit |
|---|---|---|
| Espressioni sintomatologiche | umore depresso, anedonia, sonno, energia, concentrazione, autosvalutazione | Individuare evidenze compatibili con il target |
| Storia clinica e trattamento | diagnosi riferite, psicoterapia, farmaci, precedenti episodi | Separare storia clinica e sintomi attuali |
| Indizi dell'intervista | formule di approfondimento, domande su diagnosi/trattamento, transizioni tematiche | Individuare possibili segnali del protocollo |

Queste famiglie sono **categorie di analisi**, non classi di parole automaticamente «buone» o «spurie». La stessa espressione cambia significato a seconda di chi parla, della negazione e del riferimento temporale. Una diagnosi passata riferita dal partecipante è informazione reale, pur non coincidendo necessariamente con lo stato attuale.


## 4. Tecniche di explainability e nucleo meccanicistico

### 4.1 - Localizzazione preliminare

**Esempio comune, inventato solo per spiegare gli esperimenti:** Ellie chiede «Do you still go to therapy?» e il partecipante risponde «I stopped therapy two years ago». Il lessico marca la domanda di Ellie come *approfondimento sul trattamento* e la risposta come *storia di trattamento riferita dal partecipante*. Non assegna la label PHQ-8 a queste frasi e non viene passato all'LLM predittivo. Una possibile variante è la domanda «Could you tell me about the help you received?», lasciando invariata la risposta. La coppia va comunque controllata da persone: cambiando domanda si può cambiare anche il contesto interpretativo.

| Tecnica | Esperimento esemplificativo | Che cosa può indicare, e limite |
|---|---|---|
| **Occlusione di span** | Nascondere «Do you still go to therapy?» e misurare quanto cambia lo score del dialogo. | Segnala un passaggio candidato; cancellarlo può però rendere meno comprensibile la risposta, perciò serve una variante coerente e un controllo abbinato. |
| **Attribuzione basata sui gradienti** | Calcolare quanto lo score è sensibile agli embedding dei token della domanda, confrontandoli con quelli della risposta. | Offre una graduatoria di token da esaminare; un gradiente elevato è una sensibilità locale, non prova che un token sia necessario alla decisione. |
| **Probe lineare** | Addestrare un piccolo classificatore su rappresentazioni interne, su interviste separate, per riconoscere se una domanda appartiene alla categoria «trattamento». | Mostra se l'informazione è decodificabile in un layer; non mostra che l'LLM la usi per predire il PHQ-8. |
| **Spiegazione verbale dell'LLM** | Chiedere perché ha predetto una classe e verificare se cita «therapy». | Fornisce un'ipotesi da controllare: il modello può nominare una prova che non ha avuto effetto causale sulla sua predizione. |

Queste tecniche servono a generare candidati e controlli. La verifica del contributo interno richiede gli interventi descritti sotto; i siti vanno selezionati su dati separati da quelli usati per confermarli.

### 4.2 - Activation patching: metodo principale

Il patching sostituisce attivazioni di una computazione con quelle ottenute da un altro input, misurando l'effetto sull'output. La scelta della coppia di input, del punto d'intervento e della metrica condiziona l'interpretazione. [Zhang e Nanda, ICLR 2024](https://arxiv.org/abs/2309.16042).

**Esempio:** eseguire l'LLM sul dialogo con «Do you still go to therapy?» e sulla variante con «Could you tell me about the help you received?», mantenendo la stessa risposta. Se i due score differiscono, prendere l'attivazione di un layer dalla prima esecuzione e inserirla nella seconda, alla posizione finale da cui si genera la classe. Se lo score della seconda esecuzione si avvicina a quello della prima più che nei patch di controllo, quel sito **media parte della differenza causata dalla coppia di input studiata**. Questo non isola automaticamente la categoria «trattamento»: le formulazioni differiscono anche per lunghezza e sfumature semantiche. Servono più coppie, controlli linguistici e, se si interviene su token intermedi, un allineamento esplicito delle posizioni.

**Protocollo proposto per questa tesi su come applicarlo:**

1. Costruire coppie originale/variante in cui cambia un indizio definito dal lessico. Separare interventi sul protocollo e interventi sui sintomi: rispondono a ipotesi diverse.
2. Misurare lo score di entrambe le versioni. Se una coppia non produce differenze apprezzabili, conservarla nell'analisi della frequenza degli effetti; non usarla per rapporti di recupero instabili.
3. Intervenire inizialmente sul residual stream di pochi layer e su posizioni comparabili, per esempio la posizione finale da cui si predice la classe. Approfondire i contributi di attention head o MLP soltanto dove emerge un effetto replicabile.
4. Trasferire le attivazioni dall'originale alla variante e viceversa. Per interventi su token intermedi, usare template allineati o una mappatura esplicita degli span; non associare meccanicamente posizioni che hanno cambiato significato.
5. Misurare quanto lo score si sposta verso quello della computazione sorgente e confrontare l'effetto con patch su siti di controllo.

Per rendere la misura trasparente, definire `s(x)` come score della classe positiva. L'effetto primario del patching è `s(variante con patch) - s(variante)`. Un recupero normalizzato può essere calcolato dividendo per `s(originale) - s(variante)`, solo quando il denominatore supera una soglia stabilita sul development. Riportare sempre anche gli effetti non normalizzati e i casi esclusi dal rapporto.


   
---

## 6. Valutazione, fattibilità e risultati attesi

### Tre valutazioni distinte

| Livello | Misure proposte |
|---|---|
| Predizione | macro-F1, precision/recall della classe positiva, balanced accuracy e intervalli d'incertezza; soglie fissate sul development |
| Lessico | accordo tra annotatori, precisione/recall del riconoscimento dei costrutti su passaggi annotati, copertura e tipi di errore |
| Spiegazione meccanicistica | variazione dello score, frequenza degli effetti, recupero dopo patch, confronto con controlli e replica su input nuovi |



Per valutare H3, confrontare metodi di selezione con lo stesso numero di span/componenti e lo stesso budget di interventi. Il lessico è utile se favorisce effetti più replicabili, migliore copertura di categorie o minore costo di ricerca: non deve necessariamente migliorare la classificazione.

## 7. Esperimenti e Come cambia il titolo in base ai risultati 

### 7.1 - Esperimenti da riportare separatamente

| Codice | Esperimento e misura principale | Esito positivo | Esito negativo o non risolto |
|---|---|---|---|
| **E0  -  Dati e lessico** | Verificare split e sovrapposizioni tra DAIC-WOZ/E-DAIC; annotare un campione per categoria, parlante, negazione e temporalità; misurare accordo e precisione del matcher. | Campioni utilizzabili e categorie sufficientemente affidabili per scegliere interventi. | Trascrizioni/label/ruoli insufficienti, sovrapposizioni non risolte o categorie poco affidabili: limitare corpus o ipotesi; non dichiarare validità clinica del lessico. |
| **E1 - Modello predittivo** | Misurare macro-F1 e stabilità del modello sul task PHQ-8 rispetto a baseline; includere un controllo con cambiamenti pertinenti nelle affermazioni del partecipante. | Il modello produce uno score informativo e reagisce a evidenze rilevanti: ha senso spiegare le sue predizioni. | Prestazioni vicine alla baseline, output instabili o assenza di sensibilità ai controlli: un effetto nullo sulle domande di Ellie sarebbe poco interpretabile. |
| **E2 - Segnale disponibile nelle domande** | Valutare Ellie-only rispetto a participant-only e a baseline banali su partecipanti separati. | Ellie-only predice sopra i controlli: le domande veicolano segnale nel corpus. | Ellie-only non supera i controlli: manca evidenza di segnale autonomo, ma il modello sul dialogo completo potrebbe comunque usare le domande come contesto. |
| **E3 - Uso degli indizi di Ellie nel dialogo completo** | Riformulare o neutralizzare domande candidate, mantenere le risposte e controllare la coerenza; confrontare cambiamenti dello score e della classe con modifiche abbinate. | Effetto replicabile superiore alla soglia prefissata e ai controlli: il modello usa almeno gli indizi manipolati. La parola *shortcut* richiede anche argomentare perché quegli indizi siano inappropriati per il target. | Limite superiore dell'intervallo sotto la soglia: evidenza di effetto trascurabile nelle condizioni testate. Intervallo ampio o coppie incoerenti: risultato inconcludente. |
| **E4 -Mediazione interna** | Sulle coppie E3 con effetto, fare patching in entrambe le direzioni su siti selezionati fuori dal campione finale; confrontare ablazioni e siti di controllo. | Componenti specifiche modificano lo score nella direzione attesa e l'effetto si replica su nuovi partecipanti: evidenza di mediazione interna per quelle coppie. | Nessun sito regge ai controlli: non attribuire l'effetto a un circuito identificato. Se E3 non produce differenze, E4 su quelle coppie è *non applicabile*, non un risultato negativo. |
| **E5 - Valore aggiunto del lessico** | Con lo stesso numero di span, componenti e interventi, confrontare selezione guidata dal lessico, scelta casuale abbinata e attribuzioni generiche; misurare precisione degli span e frequenza/replica degli effetti interni. | Il lessico migliora una misura prefissata rispetto ai controlli: rivendicare utilità del lessico per **progettare l'audit**. | Nessun vantaggio o lessico inaffidabile: riportare il risultato e attribuire i meccanismi al patching, senza dire che il lessico li ha trovati meglio. |
| **E6 -Trasferimento di protocollo** | Ripetere E1–E3, ed E4 se fattibile, su sessioni autonome E-DAIC disgiunte; documentare differenze di ASR e disponibilità di Ellie. | Direzione ed entità dell'effetto coerenti: supporto alla trasferibilità nei corpus testati. | Effetto che cambia o scompare: risultato di dipendenza dal contesto di raccolta, da analizzare anche alla luce di ASR e composizione del campione. Se i dati non bastano, E6 è *non valutabile*. |
| **E7 -Eventuale mitigazione** | Solo dopo E3/E4 positivi, attenuare gli indizi identificati nei dati o nel modello e ripetere predizione, E3 ed E6. | Minore dipendenza dagli indizi, senza un degrado eccessivo della predizione, su dati di conferma. | Prestazione o robustezza peggiorata, oppure dipendenza invariata: la mitigazione tentata non è supportata. E7 è facoltativo e non condiziona la tesi di audit. |

E2 ed E3 rispondono a domande differenti: **informazione presente nelle domande** e **informazione usata dal predittore sul dialogo completo**. Per E3 un confronto «non significativo» con intervallo ampio resta *inconcludente*. Le condizioni per sostenere un effetto piccolo sono descritte nella sezione 7. E4 non può trovare la mediazione di una differenza che E3 non ha mostrato; in quel caso si possono applicare interventi interni a coppie basate sulle evidenze del partecipante, dichiarando il cambio di domanda di ricerca.

### 7.2 - Tipologie di titoli

| Risultati osservati | Titolo coerente e portata della conclusione |
|---|---|
| **E0+, E1+, E2+, E3+, E4+, E5+, E6+**; E7 facoltativo | **Indizi del protocollo d'intervista nelle predizioni di un LLM sulla depressione: un audit meccanicistico guidato da lessici su DAIC-WOZ ed E-DAIC.** Si può discutere un effetto interno replicato e il contributo del lessico, senza chiamare automaticamente «circuito della depressione» i siti trovati. |
| **E0+, E1+, E2±, E3+, E4+, E5+, E6 n.a.**; E7 non necessario | **Un audit meccanicistico guidato da lessici degli indizi d'intervista in un LLM su DAIC-WOZ.** Il risultato è interno e il lessico aiuta, ma non si rivendica trasferimento a E-DAIC. E2 può essere positivo o negativo, perché non decide l'uso nel dialogo completo. |
| **E0+, E1+, E2±, E3+, E4? o −, E5± o ?, E6+ o ? o n.a.** | **Sensibilità di un LLM alle domande dell'intervistatore nella predizione della depressione: perturbazioni controllate su interviste semistrutturate.** La dipendenza comportamentale è osservata, ma non c'è prova stabile di quali componenti interne la medino. Citare E-DAIC nel titolo solo se E6 è positivo e sufficientemente studiato; E6− ricade nel caso sui limiti di trasferimento. |
| **E0+, E1+, E2+, E3− con intervalli stretti e controlli sensibili, E4 n.a., E5 solo precisione degli span, E6+ o ? o n.a.** | **Verifica della dipendenza dagli indizi d'intervista in un LLM per la predizione della depressione.** Il segnale è disponibile nelle domande, ma l'effetto degli indizi manipolati sul modello completo è inferiore alla soglia prefissata nei dati testati. Se E6 è positivo, specificare che riguarda la replica dell'effetto trascurabile, non una dipendenza trovata. |
| **E0+, E1+, E2±, E3− con intervalli stretti, E4+ su evidenze del partecipante, E5± sugli span del partecipante, E6+ o ? o n.a.** | **Evidenze del partecipante e indipendenza dagli indizi d'intervista: un audit meccanicistico di un LLM per la depressione.** Il patching qui risponde a una domanda diversa da E4 sugli indizi di Ellie: il modello usa specifiche informazioni del partecipante, mentre l'effetto delle domande manipolate è trascurabile nelle condizioni testate. |
| **E0+, E1+, E2±, E3? per intervalli ampi o perturbazioni incerte, E4 n.a. sugli indizi di Ellie, E5 solo precisione degli span, E6? o n.a.** | **Sensibilità di un LLM a domande e risposte nelle interviste per la depressione: un'analisi sperimentale.** Non affermare né presenza né assenza di shortcut; riportare con precisione quali confronti restano aperti. |
| **E0+, E1+, E2±, E3+, E4+, E5−, E6+ o ? o n.a.** | **Analisi meccanicistica della dipendenza di un LLM dagli indizi d'intervista.** L'effetto interno resta un risultato, mentre il lessico non ha superato i metodi di selezione di controllo. Indicare il corpus effettivamente studiato nel sottotitolo. |
| **E0+, E1+, E2±, E3+ in DAIC-WOZ, E4+ o ?, E5+ o − o ?, E6− in sessioni autonome disgiunte** | **Dipendenza dal protocollo d'intervista e limiti di trasferibilità delle spiegazioni di un LLM tra DAIC-WOZ ed E-DAIC.** Esporre anche quanto delle differenze potrebbe dipendere dalle trascrizioni e dalla composizione dei campioni. |
| **E0+ per la sola valutazione del lessico, E1? o −, E2–E4 non interpretabili, E5 limitato alla precisione degli span, E6 n.a.** | **Fattibilità di un audit lessicale delle interviste DAIC-WOZ per la predizione della depressione.** È un progetto ridimensionato: il titolo da solo non rende sufficiente una tesi meccanicistica senza un modello predittivo studiabile; occorre ridefinire la domanda scientifica con il relatore. |

La presenza di E6 non valutabile elimina «ed E-DAIC» da qualsiasi titolo che prometta risultati su entrambi i corpus. E7 entra nel titolo solo se eseguito e confermato, per esempio aggiungendo «e mitigazione degli indizi di protocollo» al primo caso. Il titolo iniziale resta appropriato durante lo svolgimento: l'esito finale non è noto in anticipo. Nessun risultato nullo su questo campione autorizza formule generali come «gli LLM non hanno bias».

La proposta da discutere con il relatore è dunque un **audit delle decisioni del modello guidato da un lessico**, con verifica causale interna come obiettivo principale e scoperta di circuiti completi o mitigazione come possibili sviluppi.
