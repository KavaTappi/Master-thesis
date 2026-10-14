# Proposta Carman 00 — Lessici per la scoperta di circuiti negli LLM

**Data:** 14 ottobre 2026. **Stato:** proposta di tesi da discutere, con ipotesi e protocollo da verificare mediante un pilot. Nessun risultato sperimentale è già acquisito.

## 1. Titolo e idea in breve

**Titolo proposto:** *Il valore della guida lessicale nella scoperta di circuiti degli LLM: fedeltà, costo e generalizzazione su interviste DAIC-WOZ*.

**Domanda centrale:** a parità di risorse, usare un lessico per dare priorità ai candidati durante la scoperta dei circuiti permette di ottenere spiegazioni più fedeli, che restino valide su partecipanti ed espressioni nuovi?

Il sistema da spiegare è un LLM che stima l'etichetta associata al PHQ-8 a partire da un input testuale derivato da un'intervista. Il lessico è una risorsa del metodo di spiegazione: assegna una priorità alla ricerca interna. Il classificatore riceve lo stesso testo in tutte le condizioni.

Il risultato centrale sarà una **curva costo–fedeltà**, accompagnata da una verifica su parafrasi e riferimenti contestuali. La scoperta di una shortcut non è una condizione necessaria per svolgere la tesi.

**Valutazione preliminare:** è una possibile tesi magistrale di ricerca, con difficoltà medio-alta. Diventa sostenibile concentrandosi su un modello, un task, una variante di ricerca e pochi confronti controllati. La pubblicabilità dipenderà dalla solidità dei risultati e dal confronto finale con i precedenti.

### Come interpretare un esperimento nella tesi

Supponiamo che il modello produca uno score di 1,8 e che un intervento su un percorso lo porti a 1,1: la differenza 0,7 è l'effetto di quell'intervento, con quell'input e quel riferimento. Sono numeri illustrativi. Non autorizzano a chiamare il percorso «circuito della tristezza»: servono confronti su esempi diversi e controlli contestuali.

Un insieme di percorsi può condividere componenti e avere effetti non additivi. Inoltre, l'unione dei loro archi può introdurre combinazioni di percorsi ulteriori. La tesi deve quindi valutare il sottografo risultante mediante un intervento congiunto, evitando di stimarne la fedeltà sommando gli effetti individuali.

## 2. Domande di ricerca e ipotesi falsificabili

| Domanda | Ipotesi da verificare | Esito che la indebolirebbe |
|---|---|---|
| RQ1 — Efficienza | La guida lessicale riduce l'errore di ricostruzione a parità di costo. | Il vantaggio sparisce includendo il costo della guida o confrontando una guida casuale equivalente. |
| RQ2 — Generalizzazione | Il circuito scoperto resta fedele su persone nuove e parafrasi che evitano le voci usate nella guida. | La fedeltà crolla quando le parole iniziali non compaiono. |
| RQ3 — Specificità della risorsa | La selezione semantica del lessico aggiunge informazione rispetto a frequenza, posizione e numero di token. | Una selezione casuale abbinata per queste proprietà produce lo stesso risultato. |
| RQ4 — Contesto, secondaria | Il circuito riproduce la sensibilità del modello a negazione, tempo e persona cui è attribuita l'esperienza. | Riproduce gli esempi lessicalmente semplici ma perde le differenze contestuali. |

RQ1 è primaria; RQ2 e RQ3 sono necessarie per interpretarla. RQ4 è un approfondimento circoscritto, da ridurre se il campione o il tempo non bastano.

## 3. Sistema da spiegare e dati

### 3.1 Task e modello

Usare inizialmente la classificazione binaria fornita con il dataset, documentando la relazione con il punteggio PHQ-8 nella release effettivamente disponibile. Evitare di aggiungere contemporaneamente regressione, multimodalità e generazione di spiegazioni.

Il modello restituisce uno fra due simboli di classe, scelti in modo che siano singoli token nel contesto del prompt. Definire:

`s(x) = logit(classe 1 | x) − logit(classe 0 | x)`

`p(x) = sigmoid(s(x))`

`p` è la probabilità rinormalizzata sulle due opzioni, non una probabilità clinica automaticamente calibrata. Congelare prompt, simboli, checkpoint e soglia decisionale prima dell'audit. Verificare sul development che il risultato non dipenda esclusivamente dalla scelta dei simboli.

Scegliere un modello piccolo con pesi e attivazioni accessibili. **Qwen2.5-0.5B è un candidato per la prova tecnica**, dato che compare nella configurazione MIB; questo non garantisce prestazioni sufficienti su DAIC. Il [repository MIB](https://github.com/aaronmueller/MIB) e il [codice collegato a IPE](https://github.com/nepp1d0/MIB-circuit-track-with-paths) sono punti di partenza da verificare sul checkpoint esatto.

Se necessario, un singolo adattamento LoRA può servire a ottenere un classificatore utilizzabile. Va eseguito solo sui dati di addestramento e poi congelato: confrontare sistematicamente prima/dopo fine-tuning sarebbe una seconda tesi. Le repliche dell'audit e quelle dell'addestramento vanno riportate separatamente.

### 3.2 Interviste lunghe: una decisione esplicita

La [fonte ufficiale DAIC](https://dcapswoz.ict.usc.edu/) descrive 189 sessioni, con trascrizioni e interviste di durata variabile. L'accesso deve essere già disponibile o richiesto tramite il canale accademico previsto.

Per il pilot, partire da input di circa 1.024 token e verificare se sia sostenibile arrivare a 2.048; questi sono budget proposti, non limiti del metodo. Se l'intervista non entra nel budget, estrarre blocchi distribuiti lungo il dialogo con una regola deterministica, indipendente da lessico ed etichetta. Conservare interi turni ove possibile, ordine, speaker e marcatori dei tagli.

**In questo caso si spiega la predizione sull'input estratto**, non il comportamento che il modello avrebbe sull'intervista completa. Misurare la quota di testo coperta e dichiarare questa limitazione nel titolo sperimentale e nei risultati. Non scegliere gli estratti cercando parole del lessico, perché confonderebbe selezione dell'input e guida del circuito.

Nel protocollo principale mantenere lo stesso contesto domanda–risposta per tutte le strategie; applicare il lessico solo alle risposte del partecipante. Il circuito rimane libero di includere componenti legate alle domande. Prevedere un controllo descrittivo con soli turni dell'intervistatore, senza interpretarlo come un audit completo delle shortcut. [Burdisso et al., 2024](https://arxiv.org/abs/2404.14463) documentano il rischio di segnali predittivi nel protocollo di intervista.

### 3.3 Separazione dei dati

Distinguere quattro usi: eventuale addestramento del modello, scoperta dei circuiti, scelta degli iperparametri e valutazione finale. I dati di scoperta possono essere una parte del training; una replica su persone non viste nell'addestramento è preferibile quando il campione lo permette.

- Rispettare gli split ufficiali effettivamente disponibili e mantenere il test finale intatto.
- Se le label di test non sono disponibili, dichiarare una valutazione interna con split annidati per partecipante, senza chiamarla test ufficiale.
- Tutte le versioni di un'intervista rimangono nello stesso split.
- Congelare il lessico prima della valutazione; eventuali aggiunte vengono solo da fonti esterne o training.
- Nella prova lessicale, separare famiglie di lemmi ed espressioni fra scoperta e verifica, per quanto consentito dal campione. Una prova costruita con parafrasi va riportata separatamente dalla distribuzione naturale.

### 3.4. Che cosa contiene il lessico

Per il primo esperimento usare una risorsa inglese già documentata, per esempio una selezione dichiarata di categorie di **NRC EmoLex**. Questo lessico associa parole a emozioni e sentiment; non attribuisce diagnosi. Fonte e versioni: [risorse lessicali NRC, pagina dell'autore](https://www.saifmohammad.com/WebPages/lexicons.html).

Una seconda versione, opzionale, può aggiungere poche espressioni relative a sonno, energia o interesse, revisionate indipendentemente. L'espansione va confrontata con il lessico iniziale e deve includere le ore di annotazione fra i costi. Non costruire il lessico selezionando parole correlate al PHQ-8 sull'intero dataset.

Per ogni corrispondenza conservare lemma/espressione, categoria, offset nel testo, token del modello e speaker. Negazioni e indicatori temporali possono contribuire a definire una finestra contestuale; non devono diventare regole che assegnano automaticamente presenza o assenza del sintomo.

**«Una risorsa lessicale documentata» significa concretamente:** un file versionato di parole/espressioni e categorie, accompagnato da fonte bibliografica, lingua, versione, condizioni d'uso, criteri di inclusione, regole di matching e modifiche eventualmente apportate. Per una risorsa costruita nella tesi servono anche istruzioni per gli annotatori e verifica dell'accordo. È possibile usare una risorsa esistente: non è necessario costruire un nuovo lessico. Documentare non equivale a validare clinicamente; un termine associato a tristezza non identifica automaticamente un sintomo depressivo nel contesto.

**Controllo indispensabile:** maschere casuali con analogo numero di token, lunghezza degli span, posizione nei turni e frequenza lessicale. Se il metodo usa solo la presenza di una voce e ignora la categoria, rimescolare le etichette delle categorie non cambia nulla: non sarebbe un controllo informativo.

## 4. Metodo proposto: usare il lessico per ordinare i candidati

Questa sezione definisce una **variante sperimentale da implementare**, non una funzionalità già disponibile in IPE. L'obiettivo è mantenere identici interventi e criteri finali, cambiando la priorità con cui vengono esaminati i candidati.

### 4.1 Screening economico

Usare **EAP-IG**, una stima mediante gradienti integrati dell'effetto del patching sugli archi, per costruire una graduatoria iniziale. È una stima da verificare con interventi, non una misura esatta di ogni percorso. Riferimenti: [EAP](https://arxiv.org/abs/2310.10348) ed [EAP-IG](https://arxiv.org/abs/2403.17806).

Per ogni arco `e`, conservare ove possibile i contributi prima dell'aggregazione sulle posizioni del testo. Costruire:

- `G(e)`: importanza stimata aggregata su tutte le posizioni;
- `L(e)`: importanza stimata aggregata nelle posizioni coperte dal lessico e in una finestra contestuale fissata;
- `R(e)`: stessa quantità su posizioni casuali abbinate, per il controllo.

Usare magnitudini per lo screening e conservare i segni per l'interpretazione. Aggregare prima per persona, poi fra persone, per non privilegiare interviste lunghe o molte varianti della stessa persona. Normalizzare per il numero di posizioni incluse e convertire i punteggi in ranghi comparabili.

`L(e)` indica associazione fra contributi stimati e posizioni selezionate. Non dimostra che l'arco trasporti soltanto informazione lessicale. Se la libreria aggrega troppo presto i contributi, sarà necessario adattare questa parte e verificarla su un esempio semplice.

### 4.2 Priorità e ricerca

Definire una priorità degli archi:

`Q_λ(e) = (1 − λ) rank(G(e)) + λ rank(L(e))`

con rango alto corrispondente a priorità alta e `λ` scelto soltanto sul development, per esempio fra 0, 0,25 e 0,5. Per il controllo casuale sostituire `L` con `R`.

Nella ricerca all'indietro, assegnare a un percorso parziale la media delle priorità dei suoi archi. È una regola euristica semplice, identica nei confronti. Valutare i candidati con interventi sul percorso e con uno stesso criterio di prosecuzione. Riservare, per esempio, una scelta ogni cinque alla graduatoria globale senza guida lessicale, in modo da mantenere esplorazione fuori dal lessico.

La variante `λ = 0` costituisce il confronto diretto: stessa implementazione, stesso screening, stessa regola di ricerca. Riportare separatamente il confronto con IPE originale. Budget, profondità massima, soglia e regola di esplorazione devono essere uguali nelle varianti comparabili.

È un'ipotesi verificabile: il punteggio lessicale potrebbe non predire i percorsi utili oppure favorire percorsi troppo specifici. La valutazione deve poter rilevare entrambi i casi.

### 4.3 Costruzione e congelamento del circuito

Dai percorsi valutati ricavare sottografi di diverse dimensioni. Definire una regola comune di ordinamento e connessione degli archi, conservando i punteggi con segno. Usare nodi a livello di testa/MLP e archi condivisi fra posizioni, così che il circuito sia applicabile a testi con parole diverse.

La selezione finale deve essere identica fra metodi. A parità di budget si confrontano circuiti di dimensioni comparabili; a parità di dimensione si confrontano i costi necessari a trovarli. La ricerca lessicale non deve vincere semplicemente perché produce un sottografo più grande.

Congelare i circuiti prima di valutarli sulle nuove persone. Riscoprire ogni volta un circuito sul test misurerebbe un'altra domanda.

### 4.4 Confronti minimi

| Sigla | Strategia | Perché serve |
|---|---|---|
| B0 | EAP-IG con selezione standard di archi | Verifica se la ricerca aggiuntiva dei percorsi ripaga il costo. |
| B1 | Ricerca di percorsi con priorità globale, `λ = 0` | Controllo diretto della variante proposta. |
| S | Stessa ricerca, con maschere ottenute dalla salienza dei token e abbinate per numerosità | Verifica se la conoscenza lessicale aggiunge valore a una guida ricavata direttamente dal modello. |
| L | Stessa ricerca con priorità lessicale | Metodo candidato. |
| R | Stessa ricerca con maschere casuali abbinate | Isola il valore della scelta linguistica delle posizioni. |
| IPE | Procedura originale, sul sottoinsieme sostenibile | Colloca la variante rispetto al punto di partenza. |

Una variante che vieta del tutto i candidati esterni al lessico può essere un'ablazione secondaria per misurare il costo di un vincolo troppo rigido. Non è necessaria al primo ciclo.

**Aggiornamento bibliografico:** [Position-aware Automatic Circuit Discovery, ACL 2025](https://aclanthology.org/2025.acl-long.141/) richiede inoltre un confronto specifico, discusso nella sezione 14. Una baseline S non riproduce quel metodo: serve un confronto con la sua implementazione su un sottoinsieme compatibile, oppure una delimitazione esplicita della rivendicazione se il confronto non è sostenibile. Tenere separati gli effetti di guida della ricerca e granularità del grafo.

## 8. Interventi, riferimenti e prove di generalizzazione

### 8.1 Riferimenti per il patching

Ogni intervento richiede di specificare che cosa sostituisce il segnale originale. Preparare sul discovery un insieme di coppie `x, x'` con modifiche locali revisionate, includendo sia espressioni lessicali sia passaggi esterni al lessico. Usare le stesse coppie per tutti i metodi.

Per la prima implementazione ammettere nel patching soltanto coppie con lunghezza tokenizzata compatibile e allineamento esplicito delle posizioni. Scartare le coppie non allineabili da questa analisi e riportare la quota esclusa. Un testo grammaticalmente corretto può comunque introdurre un artefatto sperimentale: validare fluidità, coerenza con il resto dell'intervista e proprietà effettivamente modificata.

La prova di fedeltà usa un insieme separato e più ampio di interventi, senza concentrarsi solo sulle parole del lessico. Una verifica secondaria con un'altra baseline di ablazione, per esempio medie stimate solo sul training, misura quanto le conclusioni dipendano dal riferimento.

### 8.2 Tre prove distinte

| Prova | Esempio illustrativo in inglese | Che cosa misura |
|---|---|---|
| Nuove persone | Interviste mai usate per scoprire o scegliere circuiti | Trasferimento fra individui. |
| Parafrasi revisionate | `I feel sad` → `I feel down`, quando equivalenti nel contesto | Generalizzazione oltre le voci iniziali. |
| Modifica del contesto | `I feel sad` → `My sister feels sad` oppure negazione | Capacità di riprodurre le differenze del modello tra significati diversi. |

Queste frasi sono esempi didattici, non annotazioni DAIC. Per il tempo distinguere esplicitamente passato risolto, esperienza attuale e temporalità incerta: un verbo al passato da solo non prova irrilevanza per il periodo considerato.

Per le parafrasi misurare separatamente stabilità del modello completo e fedeltà del circuito. Se cambia il modello, il circuito può restargli fedele pur riproducendo una fragilità.

Per le modifiche semantiche **non imporre un cambio di etichetta PHQ-8**. Confrontare il cambiamento dello score del modello con quello del circuito. Una singola modifica può avere effetto piccolo se nell'intervista rimangono altre evidenze. L'eventuale adeguatezza clinico-linguistica richiede annotazioni proprie e rimane distinta dalla fedeltà.

## 9. Tecniche di explainability da usare

| Tecnica | Ruolo nella tesi | Limite interpretativo |
|---|---|---|
| EAP-IG | Screening e priorità degli archi | Approssima gli interventi; non certifica il circuito. |
| Activation/edge patching | Verifica diretta di un sottoinsieme di componenti o archi | L'effetto dipende dal riferimento e dalla propagazione consentita. |
| Interventi sui percorsi | Conferma dei candidati dopo lo screening | Il contributo di un percorso non va sommato automaticamente agli altri. |
| Ablazione del complemento del circuito | Misura di sufficienza operativa | Un buon risultato vale per il task e il regime di ablazione specificati. |
| Ablazione del circuito | Sensibilità alla sua rimozione | Ridondanza e circuiti alternativi possono compensare la rimozione. |
| Perturbazioni linguistiche controllate | Test di generalizzazione e contesto | Non generano automaticamente nuove label cliniche. |
| Visualizzazione del sottografo | Ispezione degli esempi e comunicazione dei risultati | Il grafico deve essere accompagnato dalle misure di intervento. |

SHAP sul testo, sparse autoencoder, probing e spiegazioni generate dall'LLM non sono necessari al nucleo di questa tesi. Aggiungerli richiederebbe una domanda specifica e aumenterebbe il perimetro.

## 10. Analisi quantitative

### 10.1 Notazione e misura primaria

Sia `M` il modello completo e `C` il circuito congelato. `p_C(x; x')` indica il risultato dell'esecuzione in cui i collegamenti esterni a `C` ricevono i messaggi di riferimento, mentre quelli interni mantengono il calcolo attivo secondo il protocollo di patching. Specificare esattamente questo intervento nell'implementazione.

**Errore di fedeltà primario:**

`E_fid(C) = media_per_persona [abs(p_M(x) − p_C(x; x'))]`

Prima mediare le varianti della stessa persona, poi fra persone. Riportare l'errore per ciascuna famiglia di riferimenti, la differenza nei logits e l'accordo delle classi. La distanza sulle probabilità può apparire piccola quando il modello è saturo: per questo non basta da sola.

Verificare sempre due estremi dell'implementazione: il grafo completo deve riprodurre il modello senza patching; il circuito vuoto deve corrispondere al riferimento definito dal protocollo. Confrontare inoltre con sottografi casuali di pari dimensione. Se anche il circuito vuoto ha errore basso, quel riferimento fornisce poca informazione sulla fedeltà: riportare l'errore del circuito vuoto, il guadagno `E_fid(vuoto) − E_fid(C)` e la forza del contrasto originale–riferimento. Conservare nell'analisi complessiva anche i casi poco informativi, affiancando una stratificazione con soglie fissate sul development.

Questo è un criterio operativo proposto per la tesi, non una formula presentata come nuova metrica universale. Il [benchmark MIB](https://arxiv.org/abs/2504.13151) offre riferimenti per valutazioni comparabili; eventuali metriche MIB vanno implementate con le loro definizioni originali, senza rinominare questa distanza come CMD.

### 10.2 Misure da riportare

| Aspetto | Misura concreta | Interpretazione |
|---|---|---|
| Qualità del classificatore | Macro-F1, AUROC, balanced accuracy e log-loss sul testo originale | Contestualizza il comportamento che si cerca di spiegare. |
| Fedeltà | `E_fid`, errore assoluto dello score, accordo delle classi | Quanto il circuito riproduce il modello completo. |
| Dimensione | Numero di archi e nodi; quota sul grafo disponibile | Quanto è compatta la spiegazione. |
| Costo | Secondi GPU, memoria di picco, forward/backward e componenti rivalutate | Evita di equiparare interventi con costi diversi. |
| Efficienza | Curva `E_fid` rispetto al tempo e sua area su un intervallo comune | Confronto principale a costo controllato. |
| Costo a qualità fissata | Minimo costo per raggiungere una tolleranza `δ` scelta sul development | Un metodo che non raggiunge `δ` viene indicato come tale. |
| Trasferimento lessicale | Aumento di `E_fid` fra originali e parafrasi accoppiate | Quanto la guida dipende dal vocabolario iniziale. |
| Sensibilità contestuale | Errore `abs(Δs_M − Δs_C)` per coppia | Se il circuito riproduce il cambiamento provocato dal contesto. |
| Specificità | Differenza di fedeltà e costo fra L e R | Valore della selezione lessicale rispetto alla selezione casuale abbinata. |
| Rimozione del circuito | Cambiamento di score rispetto a sottografi casuali di pari dimensione | Evidenza complementare di rilevanza; non prova da sola la sufficienza. |
| Stabilità | Variabilità fra repliche; Jaccard degli archi come descrizione secondaria | Se la conclusione dipende da un singolo campione di scoperta. |

Per i cambiamenti contestuali calcolare l'accordo del segno solo sopra una soglia prefissata di effetto del modello, riportando anche tutti i casi piccoli. Non usare rapporti di recupero quando il denominatore è prossimo a zero.



## 11. Precedenti e contributi candidati alla letteratura

### 11.1 Lavori da cui partire

| Precedente | Che cosa copre già | Spazio che questa proposta intende verificare |
|---|---|---|
| [IPE, Brunello et al., 2025](https://aclanthology.org/2025.blackboxnlp-1.30/) | Interventi e ricerca sui percorsi | Utilità della priorità lessicale a costo controllato. |
| [EAP, Syed et al., 2023](https://arxiv.org/abs/2310.10348) e [EAP-IG, Hanna et al., 2024](https://arxiv.org/abs/2403.17806) | Screening mediante gradienti e valutazione della fedeltà | Contributo incrementale di un lessico sopra uno screening già competitivo. |
| [MIB, Mueller et al., 2025](https://arxiv.org/abs/2504.13151) | Confronti standardizzati di interpretabilità meccanicistica | Valutazione su interviste con poche persone e variazioni linguistiche controllate. |
| [Improving Circuit Faithfulness via Better Edge Selection, Nikankin et al., 2025](https://arxiv.org/abs/2510.25786) | Selezione degli archi mediante stabilità e ottimizzazione | Il solo miglioramento di una graduatoria non è nuovo: serve isolare il valore del prior lessicale. |
| [Circuit Stability Characterizes Language Model Generalization, Sun, ACL 2025](https://aclanthology.org/2025.acl-long.442/) | Relazione fra stabilità dei circuiti e generalizzazione | Non rivendicare per primi la generalizzazione dei circuiti; focalizzare il costo della guida lessicale. |
| [Mechanistic Interpretability of GPT-2: Lexical and Contextual Layers in Sentiment Analysis, Hatua, 2025, preprint](https://arxiv.org/abs/2512.06681) | Studio meccanicistico di lessico e contesto nel sentiment | La sola domanda «parola o contesto?» ha già precedenti. |
| [Whether, Not Which, Keeman, 2026, preprint](https://arxiv.org/abs/2603.22295) e [AIPsy-Affect, 2026, preprint](https://arxiv.org/abs/2604.23719) | Stimoli emotivi senza parole emotive esplicite | La rimozione delle keyword è un controllo da adottare, non un contributo da rivendicare da solo. |
| [Uncovering Sentiment Analysis Circuit in Large Language Model, Li et al., ACL 2026](https://aclanthology.org/2026.acl-long.928/) | Circuiti del sentiment e sensibilità ai prompt | Trovare genericamente un circuito emotivo non basta come novità. |
| [Burdisso et al., 2024](https://arxiv.org/abs/2404.14463) | Segnali del protocollo dell'intervistatore in DAIC | La tesi misura l'utilità della guida nella scoperta, senza assumere che il segnale usato sia clinicamente valido. |

### 11.2 Tre contributi realistici

**C1 — Metodo e confronto controllato.** Una variante riproducibile di priorità lessicale nella ricerca dei circuiti, con controlli che separano l'effetto semantico da numero, posizione e frequenza delle parole. Diventa un contributo convincente se migliora il compromesso costo–fedeltà rispetto a B0, B1 e R, oppure ne chiarisce rigorosamente i limiti.

**C2 — Evidenza sulla generalizzazione delle spiegazioni guidate.** Misurare quanto il vantaggio ottenuto durante la scoperta si conservi quando scompaiono le parole usate dalla guida e cambia il contesto. Un risultato possibile è che il lessico acceleri la scoperta ma peggiori il trasferimento: documenterebbe un compromesso utile alla progettazione dei metodi.

**C3 — Protocollo riutilizzabile per interviste.** Codice, configurazioni, schemi di annotazione, definizioni degli interventi e analisi a livello di partecipante. Condividere trascrizioni o derivati soltanto nella misura consentita dai termini del dataset; il protocollo può essere pubblicato anche senza redistribuire i testi.

Una replica su un piccolo task testuale esterno rafforzerebbe C1 come contributo metodologico generale. Con il solo DAIC le conclusioni rimarrebbero un caso di studio: ciò può essere sufficiente per una buona magistrale, ma sostiene rivendicazioni più circoscritte.

## 12. Fattibilità come tesi magistrale

### Versione minima completa

1. Un LLM piccolo e congelato, con comportamento predittivo non degenere.
2. Un task binario su DAIC e una regola fissa per l'input.
3. Un lessico esistente o costruito con fonte, versione, categorie, regole di matching e criteri di revisione espliciti, come precisato nella sezione 6.
4. I confronti B0, B1, S, L e R; IPE originale come riferimento sul campione sostenibile e confronto con Position-aware Automatic Circuit Discovery secondo la sezione 14.
5. Valutazione su persone nuove e parafrasi revisionate, con budget e dimensioni controllati.
6. Intervalli di confidenza, analisi dei costi e casi di fallimento.

### Pilot e criteri per proseguire

| Verifica iniziale | Evidenza richiesta | Decisione se fallisce |
|---|---|---|
| Implementazione | Riproduzione di un caso piccolo di riferimento e controlli su output, hook e patching | Correggere prima di applicare il metodo a DAIC. |
| Task | Il modello usa il testo e non restituisce sempre la stessa classe; prestazioni e incertezza confrontate con baseline maggioritaria e lessicale | Provare un adattamento contenuto; se fallisce, riformulare il task con il relatore. |
| Calcolo | Stima misurata di memoria e costo per almeno tre lunghezze e due budget di ricerca | Ridurre modello, input o profondità; dichiarare il nuovo perimetro. |
| Screening | Disponibilità dei contributi per posizione e verifica su un esempio controllato | Implementare lo screening corretto oppure rivedere esplicitamente il metodo proposto. |
| Dati linguistici | Numero sufficiente di persone con coppie valide e accordo accettabile fra revisori | Restringere a meno categorie o riconoscere che la verifica di trasferimento è soltanto esplorativa. |

Il pilot può partire da 10–15 persone di discovery, ma la valutazione finale deve usare tutte le persone riservate compatibili con il protocollo. Un obiettivo iniziale di 60–100 coppie distribuite fra almeno 25–30 persone è un'indicazione di pianificazione, non una garanzia di potenza statistica: la numerosità effettiva va motivata con la variabilità osservata e l'ampiezza degli intervalli desiderata.

Se IPE resta troppo costoso, un piano alternativo è studiare la guida lessicale nella selezione di archi EAP-IG, confermando con patching diretto un sottoinsieme. È una variazione del contributo da concordare e dichiarare, non un risultato da chiamare IPE.




