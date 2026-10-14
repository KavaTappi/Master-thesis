# Brainstorming 01 — explainability degli LLM con interviste DAIC-WOZ

**Stato:** proposte da discutere con il relatore, dopo una prima ricerca bibliografica svolta il 14 ottobre 2026. La ricerca riduce il rischio di duplicare lavori noti, ma non certifica la novità assoluta: servono lettura integrale dei lavori più vicini, controllo delle loro citazioni successive e un pilot.

## 1. Vincoli e distinzione essenziale

DAIC-WOZ ed eventualmente E-DAIC sono **fonti di interviste e di etichette PHQ-8 a livello di partecipante**. Una frase della trascrizione non ha, di per sé, un'etichetta PHQ-8 né una validazione clinica. Il PHQ-8 chiede la frequenza dei sintomi nelle **ultime due settimane**; una frase su un episodio lontano nel tempo non è quindi la stessa evidenza di una frase su un sintomo recente, pur potendo essere informativa come storia personale. [Questionario CDC](https://www.cdc.gov/nchs/data/nhis/adult_questionnaire.pdf).

L'obiettivo qui è spiegare **il comportamento di un modello**, non inferire da una singola frase il vero stato clinico della persona. Distinguere sempre: plausibilità di una citazione, contributo causale di quel testo alla predizione e correttezza della predizione rispetto al target.

Per *span* intendo un intervallo delimitato nella trascrizione, per esempio una frase, un turno del partecipante o una domanda di Ellie. Negazione, riferimento temporale e soggetto dell'esperienza sono attributi dello span nel suo contesto, non proprietà affidabili di una parola isolata.

## 2. Prima scrematura: idee già coperte dai precedenti

| Idea iniziale | Precedente trovato | Decisione |
|---|---|---|
| Generare fattori o riassunti PHQ-8 interpretabili da DAIC-WOZ | [Lee et al. (2026)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12885269/) estraggono fattori interpretabili; [Zheng et al. (2025/2026)](https://arxiv.org/abs/2501.16106) propongono Explain-DAIC e riassunti allineati al PHQ-8. | Scartare come contributo autonomo. I riassunti possono essere baseline o dati di confronto, verificandone l'accessibilità. |
| Recuperare citazioni dall'intervista per spiegare una stima | [RED (2025)](https://aclanthology.org/2025.findings-acl.517/) recupera passaggi; [Dep-LLM (2026)](https://arxiv.org/abs/2606.10796) produce ragionamenti ancorati a evidenze su DAIC-WOZ ed E-DAIC. | Scartare la sola generazione di spiegazioni come novità. Studiare se le citazioni corrispondono alle dipendenze del predittore. |
| Misurare genericamente la fedeltà di spiegazioni di LLM con perturbazioni o patching | [Madsen et al. (2024)](https://aclanthology.org/2024.findings-acl.19/), [Kamahi e Yaghoobzadeh (2024)](https://aclanthology.org/2024.blackboxnlp-1.28/) e [Yeo et al. (2025)](https://aclanthology.org/2025.emnlp-main.529/) studiano già questi metodi. | Non rivendicare il metodo generale; definire un test specifico, controllato e falsificabile sulle interviste. |
| Verificare solo la concordanza delle keyword citate fra modelli o la robustezza ad ASR | [Loweimi et al. (2026)](https://arxiv.org/abs/2605.09634) misurano robustezza ASR, groundedness e dissociazione tra accordo sui punteggi e sulle keyword, usando HADS. | Come tesi autonoma è troppo vicina; usare la stabilità delle evidenze come controllo. |
| Estrarre evidenza strutturata con polarità, tempo e contraddizioni | Il recente [R-GCRED (2026)](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2026.1931579/full) include citazioni, polarità, persistenza temporale e contraddizioni. In NLP clinico, [ConText](https://pmc.ncbi.nlm.nih.gov/articles/PMC2757457/) tratta già negazione, temporalità e soggetto dell'esperienza. | Non rivendicare questi attributi come invenzione. Usarli per costruire **interventi sul predittore e controlli di fedeltà**. |
| Riprovare la shortcut dell'intervistatore in un altro modello | [Burdisso et al. (2024)](https://aclanthology.org/2024.clinicalnlp-1.8/) e [Watawana et al. (2026)](https://aclanthology.org/2026.lrec-1.185/) coprono più modelli e corpus. | Non sufficiente da sola. Diventa interessante solo se si misura una nuova dipendenza nel dialogo completo o un suo meccanismo. |

## 3. Proposta A — raccomandata: fedeltà **contestuale** delle evidenze citate

**Titolo possibile:** *Le citazioni spiegano davvero la predizione? Audit controfattuale delle evidenze di un LLM nelle interviste DAIC-WOZ*.

**Domanda:** quando un LLM cita uno span del partecipante, la sua predizione dipende dal **significato contestuale** di quello span oppure dalla presenza di parole salienti? In particolare, riconosce correttamente chi riferisce il sintomo, se è negato e quando è avvenuto? E cita anche gli span che hanno davvero effetto sulla predizione?

La domanda **non presuppone che l'LLM sbagli**. È plausibile che distingua bene «ultimamente dormo male» da «anni fa dormivo male». Occorre misurarlo. Inoltre un ricordo passato può influire legittimamente sulla stima complessiva: non si impone che il suo effetto sia zero, ma si verifica se il modello lo tratta come **storia** anziché come sintomo attuale.

### Disegno minimo

1. Fissare modello a pesi accessibili, trascrizioni, split per partecipante, prompt, risposte di classe e score continuo. Verificare che il task sia almeno studiabile; non scegliere il modello solo in base alla F1.
2. Su train/development individuare un campione di span del partecipante, **citati e non citati** dal modello, con annotazioni di sintomo, negazione, tempo, soggetto e ambiguità. Revisionare manualmente le annotazioni; registrare il disaccordo.
3. Costruire coppie minime **plausibili**: una modifica di significato per volta (per esempio recente ↔ remoto, affermato ↔ negato, sé ↔ altra persona) e parafrasi che mantengano il significato come controllo. Non usare soltanto cancellazioni che rendono il dialogo innaturale. Verificare la coerenza con le battute vicine.
4. Misurare la differenza nello score della classe a parità di resto della trascrizione. Confrontare effetto di span citati, non citati ma annotati come pertinenti, e span di controllo abbinati per lunghezza, posizione e categoria. Separare **sensibilità alla modifica** da **direzione desiderata**: una singola frase non determina il PHQ-8 complessivo.
5. Chiedere una nuova spiegazione per ogni variante e misurare se la citazione cambia quando cambia l'evidenza. Confrontare tre eventi: citazione senza effetto, effetto senza citazione, citazione ed effetto concordanti. Stimare incertezza con unità di campionamento a livello di partecipante.
6. Solo dove l'effetto comportamentale è replicabile, considerare patching delle attivazioni dello **stesso predittore**, con siti di controllo e conferma su partecipanti non usati per la selezione. Il patching è un'estensione condizionata alla fattibilità, non una promessa necessaria.

**Contributo candidato:** una valutazione della fedeltà delle citazioni **condizionata al contesto** nelle interviste lunghe, con contrasto esplicito fra parole identiche o simili e diversa interpretazione di tempo, negazione o soggetto. I precedenti coprono i singoli ingredienti; nella ricerca svolta non ho trovato un lavoro che esegua **questo intero confronto causale sullo stesso LLM predittivo in DAIC-WOZ**. È una lacuna da verificare ulteriormente, non una rivendicazione di primato.

**Esito informativo anche se il modello distingue bene i contesti:** si può trovare alta fedeltà oppure una divergenza specifica fra citazioni e dipendenza causale. Se entrambe risultano solide, il contributo è una validazione delimitata e un protocollo riutilizzabile; se lo stesso audit è già stato pubblicato prima della tesi, servirà spostare la domanda.

## 4. Proposta B — robustezza delle spiegazioni alla gestione del contesto lungo

**Titolo possibile:** *Quanto dipendono le spiegazioni di un LLM dal modo in cui un'intervista lunga viene presentata?*

**Domanda:** con la medesima evidenza del partecipante, score e citazioni cambiano quando si modifica soltanto la segmentazione in blocchi, l'ordine dei blocchi tematicamente indipendenti o la posizione di una frase nel contesto? L'ordine originale dell'intervista resta la condizione principale; le trasformazioni devono essere controllate per non cambiare il significato del dialogo.

**Test:** usare interviste e passaggi con evidenze annotate; confrontare trascrizione intera, blocchi con aggregazione prefissata e spostamenti ammessi solo per blocchi autonomi. Misurare accordo di score, copertura delle citazioni, stabilità degli span causalmente importanti e perdita di evidenza in funzione della posizione. Controllare la lunghezza dei prompt e le informazioni effettivamente visibili al modello.

**Precedenti e revisione:** Dep-LLM affronta già la scomposizione del dialogo lungo; Loweimi et al. studiano stabilità delle evidenze tra modelli e trascrizioni. La proposta sopravvive solo se il contributo è la **stabilità causale delle spiegazioni sotto trasformazioni che conservano l'evidenza**, e non una nuova strategia per migliorare la classificazione. Rischio principale: pochi blocchi sono davvero scambiabili senza alterare la conversazione. [Dep-LLM](https://arxiv.org/abs/2606.10796), [Loweimi et al.](https://arxiv.org/abs/2605.09634).

## 5. Proposta C — verifica interna del legame tra contesto e sintomo

**Titolo possibile:** *Dalle parole al contesto: interventi sulle rappresentazioni interne di un LLM nelle interviste DAIC-WOZ*.

**Domanda:** per coppie di frasi in cui il sintomo resta lessicalmente simile ma cambia negazione, tempo o soggetto, esistono componenti interne che mediano in modo replicabile la differenza di score? Una componente individuata per un esempio trasferisce l'effetto ad altri partecipanti e ad altre formulazioni?

**Test:** partire da coppie della proposta A con effetto comportamentale; confrontare patching in entrambe le direzioni e siti di controllo, distinguendo attivazioni associate al termine sintomatologico da quelle associate al marcatore contestuale. Confermare su coppie nuove e riportare l'effetto per tipo di variazione. Una corrispondenza con un sito non equivale a scoprire un «neurone della depressione».

**Precedenti e revisione:** Yeo et al. usano già patching per spiegazioni; [Ahsan et al. (2025)](https://aclanthology.org/2025.findings-emnlp.789/) intervengono su rappresentazioni demografiche in LLM sanitari. La specificità qui sarebbe il **legame composizionale fra sintomo e contesto in interviste DAIC**, non il patching in quanto tale. È la proposta più originale sul piano meccanicistico, ma anche la più costosa: se A non produce effetti stabili o il modello non consente interventi affidabili, C non è fattibile.

## 6. Scelta consigliata e condizioni di avanzamento

Partire da **A** come domanda principale, usare **B** come controllo di robustezza circoscritto e sviluppare **C** solo dopo un pilot positivo. Il lessico può aiutare a campionare esempi, ma non va messo nel titolo né rivendicato come contributo se non supera una selezione senza lessico a parità di budget.

Prima di fissare il titolo definitivo:

- leggere integralmente RED, Explain-DAIC, Dep-LLM, R-GCRED e Yeo et al., verificando dati, task, metrica e tipo di verifica della spiegazione;
- controllare disponibilità e condizioni d'uso di trascrizioni e annotazioni, risorse per un LLM aperto e accesso a revisori delle coppie controfattuali;
- eseguire un pilot su train/development, poi congelare interventi e metriche prima della conferma su partecipanti disgiunti;
- ripetere la ricerca bibliografica al momento della proposta definitiva: il campo evolve rapidamente.

**Criterio di rinuncia:** se il modello non svolge il task in modo sufficientemente stabile, le varianti non sono giudicate coerenti, oppure un lavoro precedente ha già eseguito lo stesso audit sulle stesse condizioni, cambiare domanda prima di presentare A come contributo originale. Un risultato nullo con intervalli stretti è comunque informativo; un risultato nullo con misure instabili è inconcludente.

## 7. Chiarimento della frase nella sezione 8 del piano

«Il modello distingue sintomi attuali da riferimenti al passato, negazioni o descrizioni di altre persone?» è **un'ipotesi da testare**, non l'affermazione che non sappia distinguerli. Sarebbe più preciso chiedere: **«La predizione e le citazioni dell'LLM rispondono in modo coerente a variazioni controllate di tempo, negazione e soggetto dell'esperienza, mantenendo il resto del colloquio plausibile?»**

Esempio: «Nelle ultime settimane dormo male» e «Dormivo male anni fa» contengono un sintomo simile ma hanno diverso rapporto con il periodo cui si riferisce il PHQ-8. Un LLM può cogliere perfettamente la differenza. La tesi misurerebbe **se lo fa nel calcolo della predizione e nella spiegazione che fornisce**, senza assumere in anticipo né errore né correttezza.
