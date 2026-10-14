# Brainstorming 02 — proposte misurabili di explainability degli LLM su interviste DAIC

**Data della ricognizione:** 14 ottobre 2026. **Stato:** quattro possibili argomenti, da selezionare con il relatore dopo lettura integrale dei precedenti e un pilot. DAIC-WOZ è la fonte principale delle trascrizioni e delle etichette PHQ-8; E-DAIC è un'eventuale verifica su un altro protocollo, solo se i dati accessibili permettono confronti corretti. Nessuna delle proposte attribuisce un'etichetta clinica a una singola frase.

## Regole sperimentali comuni

- Usare uno o due LLM **a pesi accessibili**, con prompt e versione congelati. Definire uno score continuo della decisione, per esempio `s(x) = log P(classe positiva | x) − log P(classe negativa | x)`, verificando la tokenizzazione delle risposte. La F1 sul corpus originale verifica che il predittore sia studiabile; non misura la fedeltà delle spiegazioni.
- Separare partecipanti di sviluppo e conferma. Selezionare esempi, prompt, soglie e siti interni solo nello sviluppo. Riportare intervalli di confidenza ottenuti ricampionando **partecipanti**, perché più modifiche della stessa intervista non sono osservazioni indipendenti.
- Le modifiche controfattuali della trascrizione servono a studiare la **sensibilità del modello**. La label PHQ-8 originale non diventa automaticamente la label corretta del testo modificato. Verificare manualmente plausibilità, significato e coerenza delle coppie; misurare il tasso di scarto.
- Per *span* si intende un tratto delimitato del testo, per esempio un enunciato o una domanda. Citazioni, attribuzioni e interventi vanno confrontati alla **stessa granularità** e a parità di budget. Non interpretare attenzione o una citazione come prova causale senza un intervento.
- Per ogni esperimento conservare una baseline semplice: selezione casuale di span abbinati per lunghezza e posizione, e dove pertinente un metodo generico di attribuzione. Un risultato nullo con intervallo ampio resta inconcludente.

## 1. Fedeltà causale delle citazioni del partecipante

**Domanda.** Quando l'LLM cita una frase del partecipante per giustificare la stima associata al PHQ-8, quella frase influisce realmente sullo score? Le citazioni cambiano in modo coerente quando cambiano negazione, tempo o soggetto dell'esperienza? Non si presume che il modello confonda questi casi: è proprio ciò che si misura.

**Tecniche di explainability da usare.**

1. Richiedere al **medesimo modello predittivo** classe, score e citazioni testuali con riferimenti verificabili alla trascrizione; annotare anche frasi potenzialmente pertinenti ma non citate.
2. Costruire coppie minime plausibili che cambiano un attributo contestuale alla volta (per esempio «ultimamente dormo male» / «dormivo male anni fa») e parafrasi che mantengono il significato come controlli. Usare differenze di score come stima locale dell'effetto dell'intervento, senza assegnare una nuova label clinica alla coppia.
3. Confrontare i risultati con attribuzioni su span o con selezione casuale abbinata. Se esiste un effetto comportamentale stabile e ci sono risorse, aggiungere activation patching sul predittore, in entrambe le direzioni e con siti di controllo.

**Valutazioni quantitative.** Variazione appaiata `Δs` per tipo di intervento e per parafrasi; quota di citazioni esattamente rintracciabili; correlazione di rango tra importanza implicita nelle citazioni e `|Δs|`; precisione@k delle citazioni rispetto ai k span con maggior effetto nel set di interventi **validi**; tassi di «citato ma effetto piccolo» e «effetto grande ma non citato» con soglie fissate sullo sviluppo; stabilità delle citazioni su ripetizioni del medesimo input. Riportare separatamente le direzioni degli effetti per negazione, tempo e soggetto, perché un riferimento passato può avere comunque valore contestuale.

**Stato dell'arte vicino.** [RED](https://aclanthology.org/2025.findings-acl.517/) recupera evidenze dalle interviste; [Explain-DAIC](https://arxiv.org/abs/2501.16106) offre riassunti allineati al PHQ-8; [Yeo et al.](https://aclanthology.org/2025.emnlp-main.529/) applicano patching alla fedeltà delle spiegazioni; [Siegel et al.](https://aclanthology.org/2024.acl-short.49/) misurano spostamenti nelle probabilità con controfattuali. [R-GCRED](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2026.1931579/full) include polarità e coerenza temporale dell'evidenza. Questi lavori escludono come novità la sola generazione di citazioni, l'annotazione contestuale o il patching in generale.

**Che cosa aggiunge.** Un confronto **nello stesso LLM e sulle stesse interviste** fra testo citato, effetto causale della frase e sensibilità al suo contesto semantico. Il contributo è un audit della fedeltà, non un nuovo classificatore. La novità esatta rispetto ai precedenti va ancora confermata leggendo i metodi integrali.

## 2. Quali indizi di Ellie usa un LLM che legge il dialogo completo?

**Domanda.** Il fatto che le sole domande dell'intervistatore contengano segnale predittivo significa che un LLM, quando legge domande e risposte insieme, usi quel segnale? Se lo usa, l'effetto dipende dalla formulazione di una domanda, dal suo tipo, dalla presenza di un approfondimento o dalla posizione nel colloquio?

**Tecniche di explainability da usare.**

1. Confrontare condizioni *participant-only*, *interviewer-only* e dialogo completo per lo **stesso LLM**, con prompt e score fissati. Le prime due sono controlli di disponibilità del segnale, non una prova dell'uso nel dialogo completo.
2. Nel dialogo completo, parafrasare o neutralizzare domande candidate lasciando invariata l'informazione fornita dal partecipante **solo quando lo scambio resta coerente**. Confrontare parafrasi equivalenti e domande di controllo nella stessa zona dell'intervista. Per approfondimenti condizionati dalla risposta, evitare sostituzioni che spezzano la logica del dialogo.
3. Se emerge una sensibilità replicabile, localizzare componenti con activation patching e ablazioni controllate. Distinguere i siti scelti sulla base delle domande dai siti di controllo scelti fuori campione.

**Valutazioni quantitative.** F1 e AUROC delle tre condizioni su interviste originali; `Δs` appaiato per modifiche delle domande nel dialogo completo, sottraendo l'effetto medio delle parafrasi di controllo; quota di predizioni che cambiano classe; eterogeneità dell'effetto per tipo di domanda e posizione; stabilità su più riformulazioni; per il patching, recupero dello score rispetto alla differenza originale, effetto inverso e confronto con siti di controllo della stessa dimensione. Riportare intervalli di confidenza per partecipante e limiti superiori quando si conclude che l'effetto è piccolo.

**Stato dell'arte vicino.** [Burdisso et al.](https://aclanthology.org/2024.clinicalnlp-1.8/) individuano parole e domande discriminanti con GCN e Longformer; [Watawana et al.](https://aclanthology.org/2026.lrec-1.185/) estendono l'analisi a E-DAIC e altri corpus, mostrando anche differenze tra modelli e split. [Zhang e Nanda](https://proceedings.iclr.cc/paper_files/paper/2024/hash/06a52a54c8ee03cd86771136bc91eb1f-Abstract-Conference.html) mostrano che metrica e tipo di perturbazione influenzano la localizzazione tramite patching.

**Che cosa aggiunge.** Passa da «le domande **possono** predire l'etichetta» a «questo LLM **usa** un indizio specifico quando ha anche le risposte»; se l'effetto esiste, ne studia la mediazione interna. Replicare semplicemente l'accuratezza della condizione *interviewer-only* non costituirebbe il contributo. È la proposta più vicina al piano di tesi attuale.

## 3. Stabilità causale delle spiegazioni nelle interviste lunghe

**Domanda.** Uno span che influenza la predizione continua a farlo, e continua a essere citato, quando cambia il modo tecnicamente plausibile in cui una lunga intervista viene presentata al modello? La domanda riguarda possibili effetti della posizione o della segmentazione, non una generica ricerca della migliore F1.

**Tecniche di explainability da usare.**

1. Identificare sullo sviluppo span con effetto tramite modifiche minime valide; fissare questi span prima delle prove di stabilità.
2. Confrontare intervista intera, segmentazione in blocchi con aggregazione prefissata e varianti che spostano **solo blocchi autonomi** senza alterare dipendenze tra domanda e risposta. Registrare il testo realmente visto dal modello e il budget di token: una frase tagliata fuori non è un fallimento di interpretazione.
3. Ripetere gli interventi sugli stessi span sotto le diverse presentazioni. Confrontare citazioni auto-prodotte e attribuzioni basate su perturbazioni; usare il patching soltanto su un piccolo sottoinsieme se serve distinguere localizzazione interna da effetto del prompt.

**Valutazioni quantitative.** Differenza di score e di `Δs` dello **stesso span** fra condizioni; correlazione di rango degli span per effetto; Jaccard o F1 delle citazioni dopo allineamento degli span; probabilità che uno span con effetto nel formato originale resti tra i primi k in ogni formato; curva degli effetti per posizione normalizzata e lunghezza del contesto; tasso di trasformazioni scartate perché cambiano il significato. Confrontare con modifiche puramente formali che mantengono posizione e contenuto.

**Stato dell'arte vicino.** [Liu et al.](https://aclanthology.org/2024.tacl-1.9/) studiano la dipendenza dei LLM dalla posizione delle informazioni nel contesto lungo; [Dep-LLM](https://arxiv.org/abs/2606.10796) scompone interviste lunghe per l'analisi della depressione; [Loweimi et al.](https://arxiv.org/abs/2605.09634) studiano robustezza di score ed evidenze su trascrizioni ASR per un altro strumento di valutazione. Quindi né il problema del contesto lungo né la semplice concordanza delle citazioni sono nuovi.

**Che cosa aggiunge.** Misura se la **relazione causale fra uno span e la predizione**, oltre alla citazione, sopravvive a formati equivalenti della stessa intervista DAIC. La distinzione fra instabilità dello score, della citazione e dell'effetto dello span è il risultato specifico. La fattibilità dipende dalla disponibilità di trasformazioni veramente equivalenti.

## 4. Meccanismi interni che legano un sintomo al suo contesto

**Domanda.** Quando il modello cambia score fra due frasi con lo stesso termine sintomatologico ma diversa negazione, temporalità o persona, quali componenti interne contribuiscono alla differenza? Il sito trovato su una coppia funziona anche con altre frasi e altri partecipanti?

**Tecniche di explainability da usare.**

1. Costruire coppie controllate in cui cambia un solo attributo, accanto a parafrasi equivalenti e coppie con token sintomatico cambiato ma contesto invariato.
2. Eseguire activation patching fra le due versioni in entrambe le direzioni, su posizioni del termine sintomatologico e del marcatore contestuale; confrontare layer, heads o gruppi di neuroni con siti casuali di pari dimensione. Un probe può mostrare se un attributo è decodificabile, ma l'intervento deve verificare se contribuisce allo score.
3. Se il patching identifica candidati robusti, usare ablazioni e replica fuori campione. Evitare di chiamare «circuito della depressione» un sito che media solo queste coppie.

**Valutazioni quantitative.** `Δs` originale delle coppie, quota di differenza recuperata dal patching, effetto inverso, confronto fra siti candidati e controlli, stabilità della posizione del sito tra categorie di intervento, trasferimento a partecipanti e parafrasi non usati per selezionarlo, intervalli di confidenza per partecipante. Riportare anche quanti confronti comportamentali non hanno prodotto un effetto studiabile: su questi il patching non può dimostrare la mediazione di una differenza assente.

**Stato dell'arte vicino.** [Yeo et al.](https://aclanthology.org/2025.emnlp-main.529/) usano il patching per la fedeltà delle spiegazioni; [Ahsan et al.](https://aclanthology.org/2025.findings-emnlp.789/) intervengono su rappresentazioni demografiche in LLM sanitari; [Zhang e Nanda](https://proceedings.iclr.cc/paper_files/paper/2024/hash/06a52a54c8ee03cd86771136bc91eb1f-Abstract-Conference.html) documentano la sensibilità del patching a metriche e perturbazioni. In NLP clinico, [ConText](https://pmc.ncbi.nlm.nih.gov/articles/PMC2757457/) rende chiaro che negazione, tempo e soggetto sono attributi già studiati nell'estrazione testuale.

**Che cosa aggiunge.** Una verifica **meccanicistica e composizionale** nello specifico predittore LLM: se e dove l'informazione «sintomo + contesto» contribuisce alla stima in interviste DAIC. È la proposta più impegnativa; richiede effetti comportamentali stabili e accesso alle attivazioni. Non è originale per il solo uso del patching.

## Scelta tra gli argomenti

| Argomento | Contributo più netto | Rischio principale | Decisione di pilot |
|---|---|---|---|
| 1. Fedeltà delle citazioni | Discrepanza misurabile tra spiegazione e dipendenza dal testo | Controfattuali poco plausibili o precedenti troppo vicini | Verificare coppie valide e score stabile |
| 2. Indizi di Ellie nel dialogo completo | Uso effettivo del protocollo nel singolo LLM | Domande e risposte difficili da separare senza incoerenze | Trovare perturbazioni di domande coerenti |
| 3. Interviste lunghe | Stabilità **degli effetti**, oltre che delle citazioni | Pochi blocchi indipendenti da spostare | Controllare equivalenza delle presentazioni |
| 4. Meccanismi del contesto | Mediazione interna replicata di tempo, negazione o soggetto | Costo e fragilità del patching | Procedere solo dopo effetti stabili del punto 1 |

**Raccomandazione:** scegliere **1 o 2** come domanda principale. Il punto 3 può diventare un controllo mirato per entrambi; il punto 4 è un'estensione condizionata dal pilot. Prima della scelta definitiva verificare disponibilità dei dati, risorse computazionali, revisione umana delle varianti e contenuto integrale dei paper più vicini. La ricerca bibliografica qui riportata individua un possibile spazio di contributo, non garantisce che nessuno abbia già pubblicato un esperimento identico.
