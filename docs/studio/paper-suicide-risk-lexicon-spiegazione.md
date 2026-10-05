# Spiegazione del paper: *Using Large Language Models to Create Lexicons for Interpretable Text Models With High Content Validity: The Suicide Risk Lexicon*

## Riferimento bibliografico

Daniel M. Low, Osiris Rankin, Daniel D. L. Coppersmith, Kate H. Bentley, Matthew K. Nock e Satrajit S. Ghosh (2026), *Journal of Psychopathology and Clinical Science*, pubblicazione online anticipata del 24 settembre 2026, [DOI 10.1037/abn0001152](https://doi.org/10.1037/abn0001152).

Il paper allegato presenta un metodo semi-automatico per costruire lessici specialistici con un LLM e lo applica al rischio suicidario. Il risultato e' il **Suicide Risk Lexicon (SRL)**, validato da esperti e usato come insieme di feature interpretabili per predire tre livelli di rischio in conversazioni di Crisis Text Line.

## 1. Tesi centrale in una frase

Un LLM puo' essere usato non solo come classificatore black-box, ma anche come **strumento di costruzione di una teoria lessicale esplicita**: genera parole e brevi frasi candidate per costrutti definiti a priori; esseri umani, dati di training ed esperti di dominio le filtrano; i conteggi finali alimentano un modello leggero, deterministico e ispezionabile.

Il paper non sostiene che un lessico superi in generale gli LLM. Sostiene che un lessico costruito con attenzione puo' offrire un compromesso utile fra:

- validita' di contenuto;
- interpretabilita' delle feature;
- costo computazionale;
- privacy e uso locale;
- prestazione predittiva sufficiente come baseline o componente di un sistema ibrido.

## 2. Il problema che gli autori vogliono risolvere

Per misurare costrutti psicologici nel testo si usano spesso due famiglie di metodi.

### 2.1 Lessici

Un lessico associa un costrutto a parole o brevi espressioni. Si conta quante corrispondenze compaiono in un documento e si ottiene una feature leggibile, ad esempio un conteggio normalizzato per il costrutto "umore depresso".

Vantaggi:

- e' chiaro quali token hanno attivato la feature;
- l'esecuzione e' economica e deterministica;
- il testo puo' restare in locale;
- e' possibile garantire la ricerca di espressioni note e importanti.

Limiti:

- il contesto e' modellato male;
- negazione, ironia, modalita' e intensita' possono essere perse;
- sinonimi, errori e nuove espressioni possono produrre falsi negativi;
- parole polisemiche possono produrre falsi positivi;
- costruire e validare un lessico a mano richiede molto lavoro.

### 2.2 Deep learning e LLM

I modelli neurali catturano meglio contesto, parafrasi e variazioni linguistiche e in genere sono preferibili quando l'obiettivo primario e' massimizzare la performance. Possono pero' essere costosi, difficili da ispezionare, incompatibili con alcuni vincoli di privacy e non necessariamente fedeli quando generano una spiegazione del proprio output.

La proposta del paper e' usare il punto di forza generativo dell'LLM **a monte**, per costruire velocemente una rappresentazione interpretabile che poi puo' essere ispezionata e validata.

## 3. Concetti psicometrici essenziali

### 3.1 Costrutto

Nel paper "costrutto" e' un termine ombrello per:

- fattori di rischio, come l'assenza di speranza;
- sintomi, come l'umore depresso;
- comportamenti, come l'autolesionismo diretto.

Il costrutto deve avere una definizione. Senza una definizione condivisa, non e' possibile decidere se una parola sia un esempio prototipico o solo vagamente correlato.

### 3.2 Prototipicita'

Gli autori si ispirano alla *Prototype Theory*: nel lessico devono entrare espressioni tipiche e riconoscibili del costrutto, non qualsiasi parola semanticamente vicina. Una parola puo' essere associata al tema della depressione ma non essere una manifestazione prototipica di burdensomeness, panico o isolamento.

Questo criterio serve soprattutto alla specificita': includere parole troppo generiche rende il modello apparentemente interpretabile, ma non chiarisce davvero che cosa stia misurando.

### 3.3 Validita' di contenuto

Nel paper la validita' di contenuto ha due livelli:

1. **copertura del dominio:** il modello deve rappresentare una porzione ampia e teoricamente giustificata dei fattori rilevanti per il rischio;
2. **qualita' interna del costrutto:** i token di ogni categoria devono essere espressioni prototipiche di quel costrutto.

Un modello puo' essere trasparente ma avere bassa validita' di contenuto. Sapere che una categoria generica come "fisico" ha contribuito alla predizione non significa sapere quale meccanismo psicologico e' stato misurato.

### 3.4 Validita' di criterio concorrente

Il SRL viene anche valutato verificando se le feature predicono, fuori campione, il livello di rischio assegnato dai counselor. Questa e' evidenza di validita' concorrente rispetto a quel target. Non e' evidenza che il modello predica futuri tentativi o decessi.

## 4. Domande di ricerca implicite

Il paper risponde a quattro domande principali:

1. un LLM puo' generare rapidamente un lessico ampio per un dominio ad alto rischio?
2. la curatela manuale e il giudizio clinico migliorano qualita' e performance?
3. un modello basato su lessico puo' competere con LIWC e con baseline neurali?
4. le feature interpretabili consentono di discutere quali fattori siano piu' associati al rischio imminente nel contesto di Crisis Text Line?

## 5. Dataset Crisis Text Line

### 5.1 Corpus iniziale

Gli autori partono da **96.723 conversazioni de-identificate** di crisis counseling raccolte fra il 2017 e il 2022. Gli anni 2020 e 2021 vengono esclusi per evitare che il dominio sia dominato da sintomi e preoccupazioni specifiche della pandemia COVID-19.

I dati demografici sono disponibili soltanto per circa il 22% degli utenti che compilano un questionario opzionale dopo la sessione. In quel sottoinsieme la popolazione e' soprattutto giovane, femminile, bianca ed eterosessuale. Questo limita la generalizzabilita'.

### 5.2 Target a tre livelli

Il target non e' una diagnosi e non e' un esito osservato. E' la valutazione del rischio effettuata durante la conversazione dai counselor e, nei casi ad alto rischio, dai supervisori.

| Valore | Significato operativo |
|---:|---|
| 1 - non suicidario | conversazione di crisi senza indicatori di pensieri o comportamenti suicidari secondo la valutazione disponibile |
| 2 - suicidario | presenza di pensieri o comportamenti suicidari, senza i criteri del livello imminente |
| 3 - rischio imminente | piano con orizzonte entro 48 ore, valutazione esplicita di imminenza e/o intervento dei servizi di emergenza |

Nei set di validation e test viene mantenuta la prevalenza naturale riportata: circa 80% non suicidario, 18% suicidario e 2% imminente.

### 5.3 Split e bilanciamento

Il corpus viene diviso 90% train, 5% validation e 5% test. Nel solo training le classi piu' frequenti vengono sottocampionate fino alla numerosita' della classe imminente, per evitare che il modello impari a predire quasi sempre il valore maggioritario. Validation e test conservano la distribuzione naturale.

Dopo esclusione degli anni COVID, filtri e sottocampionamento del training, il dataset modellato contiene **16.360 conversazioni**; sette esempi senza testo vengono rimossi. La tabella principale riporta **5.353 conversazioni nel test held-out**.

Questa scelta rende l'addestramento piu' equilibrato, ma richiede cautela nella calibrazione: un modello addestrato su prevalenze artificiali non produce automaticamente probabilita' calibrate per l'uso reale.

I numeri del testo principale non si riconciliano in modo immediato: il 5% di 96.723 non e' 5.353 e il totale 16.360 incorpora un training sottocampionato insieme a validation e test non bilanciati. Per una replica esatta bisogna usare la distribuzione completa della Tabella C.1 del supplemento e gli script degli autori, evitando di ricostruire le numerosita' dai soli tre totali riportati nel corpo dell'articolo.

### 5.4 Solo messaggi dell'utente

Gli autori usano solo i messaggi della persona in crisi, escludendo quelli del counselor. La motivazione e' metodologicamente importante: una domanda del counselor su un tema ad alto rischio potrebbe contenere termini fortemente predittivi anche quando la risposta e' negativa.

Questa decisione e' analoga alla distinzione `Participant-only` / `Ellie-only` necessaria in DAIC-WOZ.

### 5.5 Disponibilita'

Il dataset Crisis Text Line non e' condiviso per la sua sensibilita'. Sono invece pubblicati lessico, dati Reddit utilizzati nella calibrazione, tutorial e codice.

## 6. Il Suicide Risk Lexicon

Il paper descrive **49 fattori di rischio noti**. Il repository ufficiale precisa che l'implementazione comprende 49 fattori piu' un costrutto ausiliario relativo a relazioni e parentela; questo chiarisce perche' le tabelle modellistiche riportano vettori SRL con **50 feature**.

Le categorie teoriche coprono:

- costrutti direttamente suicidari;
- percezione negativa di se';
- sintomi depressivi;
- sintomi ansiosi;
- fattori interpersonali;
- esternalizzazione e uso di sostanze;
- altri disturbi e condizioni di salute;
- determinanti sociali e barriere al trattamento.

Fra i costrutti direttamente collegati alla suicidarieta', il paper riporta:

| Costrutto | Token finali |
|---|---:|
| ideazione passiva | 42 |
| ideazione attiva e pianificazione | 80 |
| mezzi letali | 72 |
| autolesionismo diretto | 58 |
| esposizione al suicidio | 22 |
| altro linguaggio suicidario | 72 |
| ospedalizzazione | 24 |

Fra tutti i costrutti la mediana e' 59 token, con intervallo 22-173. Il lessico include parole singole e brevi espressioni, non soltanto unigrammi.

## 7. Protocollo di costruzione in sette passi

### Passo 1 - Definire target, costrutti ed esempi seed

Gli autori definiscono come target il rischio suicidario imminente e raccolgono fattori associati a ideazione, tentativi o morte per suicidio da review e meta-analisi. Un autore compila i costrutti e aggiunge pochi esempi prototipici per guidare la generazione.

Punto forte: il lessico parte da una struttura teorica, non solo dalle correlazioni del dataset.

Rischio: selezione dei costrutti e seed dipendono comunque da decisioni umane e dalla letteratura scelta.

### Passo 2 - Generare il lessico preliminare con GPT-4 Turbo

Viene usato `GPT-4-1106-preview`, scelto per la sua posizione nelle leaderboard disponibili al momento della costruzione. Il prompt chiede molte parole e alcune frasi brevi riferite a un costrutto nel dominio della salute mentale, separate in un formato semplice, includendo gli esempi seed e senza spiegazioni aggiuntive.

Gli autori provano diverse formulazioni e tre temperature, 0, 0,5 e 1, ispezionando manualmente le liste per ottenere token prototipici, non ridondanti e nel formato desiderato.

Il ruolo dell'LLM e' quindi di **generatore di candidati**, non di autorita' clinica finale.

### Passo 3 - Integrare altre fonti e curare manualmente

Le liste generate vengono ampliate con:

- word-score analysis su post Reddit relativi alla salute mentale;
- word-score analysis sui dati di training Crisis Text Line;
- thesauri;
- item di questionari.

Un autore accorcia le espressioni quando una forma piu' breve aumenta i veri match senza introdurre troppi falsi positivi. Rimuove inoltre parole solo correlate, non prototipiche, o troppo polisemiche.

### Passo 4 - Estrarre le feature

Il pacchetto `construct-tracker` conta le corrispondenze per costrutto. La pipeline:

- lemmatizza con spaCy 3.6.1;
- impone corrispondenza esatta per token di quattro caratteri o meno, per ridurre match interni spurii;
- normalizza i conteggi per il numero di parole del documento.

Il risultato e' una matrice `documento x costrutto` interpretabile.

### Passo 5 - Calibrare falsi positivi e falsi negativi

La calibrazione usa soltanto dati di training e fonti esterne, non validation o test. Comprende:

1. confronto fra 10.000 post Reddit di salute mentale e 10.000 non relativi alla salute mentale;
2. ispezione nel contesto dei match in Crisis Text Line per scoprire sensi sbagliati;
3. aggiunta di espressioni mancate;
4. ispezione di 20 post casuali da `r/SuicideWatch` per falsi negativi;
5. rimozione di token mai osservati nel training Crisis Text Line o nei post SuicideWatch usati.

Il passaggio e' cruciale: una parola e' utile non solo se sembra pertinente isolatamente, ma se si comporta bene nei contesti reali del dominio.

### Passo 6 - Validazione di dominio

Tre coautori con esperienza nella ricerca e valutazione del rischio suicidario - una psicologa clinica abilitata e due ricercatori in formazione - giudicano quanto ogni token sia un'espressione prototipica del costrutto.

La scala e' 0-3:

- 0: non prototipico;
- 1: piu' no che si';
- 2: piu' si' che no;
- 3: chiaramente prototipico e adatto a far segnalare il documento.

Ai giudici viene chiesto di considerare anche la frequenza di sensi alternativi non pertinenti. Le liste sono ordinate per similarita' con l'embedding medio degli esempi seed usando `all-MiniLM-L6-v2`, per facilitare la revisione.

### Passo 7 - Affidabilita' fra valutatori

Tutti e tre i valutatori giudicano i token dei costrutti suicidari. Gli altri costrutti sono giudicati da due persone.

- costrutti suicidari: ICC medio **0,48**, deviazione standard **0,20**;
- altri costrutti: accordo assoluto medio **0,54**, deviazione standard **0,18**.

Gli autori classificano questi valori come **fair**, non buoni o eccellenti. La validazione esperta aumenta la credibilita' del lessico, ma non elimina l'incertezza concettuale.

## 8. Modelli confrontati

| Famiglia | Rappresentazione | Modello predittivo |
|---|---|---|
| LIWC-22 semantic | 85 categorie semantiche | LGBMRegressor |
| LIWC-22 completo | 117 feature semantiche e linguistiche | LGBMRegressor |
| SRL solo GPT-4 | 50 feature | LGBMRegressor |
| SRL GPT-4 + curatela | 50 feature | LGBMRegressor |
| SRL GPT-4 + curatela + clinici | 50 feature | LGBMRegressor |
| all-MiniLM-L6-v2 | embedding documento di 384 dimensioni, non fine-tuned | Ridge |
| DistilBERT | testo, fine-tuning supervisionato | rete neurale |
| MPNet-base | testo, fine-tuning supervisionato | rete neurale |

Per le feature lessicali gli autori confrontano anche Ridge e gradient boosting; nella tabella principale riportano il migliore per ciascuna rappresentazione. DistilBERT e MPNet funzionano come upper bound black-box, non come modelli interpretabili equivalenti.

## 9. Metriche

Il target e' ordinale `1 < 2 < 3` e il test e' fortemente sbilanciato. Il paper usa:

- **macro-average RMSE**: calcola l'RMSE separatamente per ciascun livello e poi ne fa la media, riducendo il dominio della classe maggioritaria;
- **Spearman rho**: misura quanto le predizioni aumentino monotonicamente con la severita' reale.

RMSE piu' basso e rho piu' alto sono migliori. Nessuna delle due metriche misura calibrazione clinica, utilita' decisionale o sicurezza di deployment.

## 10. Risultati principali

### 10.1 Prestazione sul test held-out

| Modello | Macro RMSE | RMSE per classe `[non suicidario, suicidario, imminente]` | Spearman rho | Validita' di contenuto assegnata dagli autori |
|---|---:|---:|---:|---|
| LIWC-22 semantic | 0,53 | `[0,61, 0,42, 0,56]` | 0,53 | bassa |
| LIWC-22 completo | 0,52 | `[0,60, 0,41, 0,55]` | 0,53 | bassa |
| SRL solo GPT-4 | 0,57 | `[0,62, 0,43, 0,67]` | 0,53 | alta |
| SRL + curatela | 0,51 | `[0,55, 0,44, 0,55]` | 0,57 | alta |
| SRL + curatela + clinici | **0,50** | `[0,55, 0,43, 0,53]` | **0,57** | molto alta |
| MiniLM embedding + Ridge | 0,51 | `[0,58, 0,40, 0,54]` | 0,56 | non determinata |
| DistilBERT fine-tuned | **0,43** | `[0,46, 0,44, 0,40]` | 0,59 | non determinata |
| MPNet fine-tuned | 0,46 | `[0,52, 0,54, 0,32]` | **0,66** | non determinata |

Lettura corretta:

- il lessico generato automaticamente da GPT-4 e' gia' competitivo con LIWC in rho, ma e' peggiore in macro RMSE;
- la curatela manuale produce il salto piu' evidente;
- la revisione clinica migliora ancora, soprattutto l'errore della classe imminente;
- il SRL validato e' molto vicino alla baseline MiniLM + Ridge;
- DistilBERT ha il miglior macro RMSE;
- MPNet ha la migliore correlazione di rango;
- i modelli neurali fine-tuned restano globalmente piu' predittivi.

Gli autori usano bootstrap appaiato con 10.000 iterazioni. Differenze di Spearman superiori a 0,02 risultano significative secondo il loro criterio. Il SRL curato supera LIWC in modo statisticamente significativo ma con effetto modesto; MPNet supera i lessici.

Con soli 150 esempi casuali di training, i modelli SRL risultano i migliori nelle analisi supplementari. Questo suggerisce un vantaggio del bias teorico in condizioni di scarsita' di dati, ma il risultato va replicato.

### 10.2 Perche' la curatela aiuta

La sequenza dei risultati supporta l'idea che la generazione LLM sia un buon acceleratore, non un sostituto della validazione:

```text
GPT-4 candidato
    -> rimozione di polisemia e falsi match
    -> integrazione di fonti empiriche
    -> giudizio clinico
    -> feature piu' specifiche
    -> migliore performance e interpretabilita'
```

Il miglioramento predittivo e' coerente con un aumento della qualita' delle feature, ma non prova da solo la validita' di contenuto: quest'ultima deriva anche dalla copertura teorica e dal giudizio esperto.

## 11. Feature importance e risultato teorico

Nel modello SRL basato su gradient boosting, le feature piu' importanti sono:

1. mezzi letali;
2. ideazione attiva e pianificazione;
3. altro linguaggio suicidario;
4. autolesionismo diretto;
5. altro uso di sostanze;
6. lutto;
7. relazioni e parentela;
8. disturbo borderline di personalita';
9. fatica;
10. trauma e PTSD.

Ideazione passiva, umore depresso e hopelessness risultano meno importanti dei segnali attivi e dei mezzi letali. Questo e' coerente con l'ipotesi che indicatori prossimali e ad alta intensita' siano piu' informativi per il **rischio imminente**.

Due ipotesi non sono confermate:

- l'umore depresso risulta piu' importante di ospedalizzazione e aggressivita/irritabilita';
- l'impulsivita' non ha importanza rilevante nel modello.

Queste importanze non sono effetti causali. Dipendono da frequenza delle parole, correlazioni fra costrutti, regole di annotazione, intervento del counselor e criterio di split del modello. Il target e' la valutazione del counselor, non un tentativo o decesso osservato in seguito.

## 12. Perche' LIWC e' un confronto importante

LIWC contiene categorie ampie e non costruite specificamente per il rischio suicidario. Alcune possono essere predittive perche' includono termini direttamente collegati al suicidio insieme a molte parole estranee. La feature resta leggibile nel nome, ma il nome non garantisce che il contenuto della categoria sia specifico.

Il SRL cerca invece una corrispondenza piu' stretta:

```text
token osservato -> costrutto definito -> fattore teorico -> predizione
```

Questa catena rende piu' utile discutere la teoria, confrontare studi e controllare errori. Tuttavia e' valida soltanto nella misura in cui token e definizioni sono corretti e stabili nel nuovo dominio.

## 13. Limiti riconosciuti dal paper

### 13.1 Contesto linguistico insufficiente

Il lessico non distingue bene negazione, ipotesi, citazione, ironia, modalita', intensita' o chi sia il soggetto dell'enunciato. Due frasi con lo stesso token possono ricevere lo stesso conteggio pur esprimendo livelli molto diversi.

### 13.2 Evidenza di validita' incompleta

Il lavoro mostra soprattutto:

- validita' di contenuto valutata qualitativamente;
- validita' concorrente rispetto al punteggio assegnato dal counselor.

Non mostra adeguatamente:

- validita' discriminante;
- stabilita' in contesti e tempi diversi;
- concordanza con valutazioni cliniche indipendenti;
- predizione prospettica di tentativi o decessi;
- sicurezza o beneficio di un uso operativo.

### 13.3 Un solo dominio

Il corpus proviene da una piattaforma statunitense di crisis texting e da una popolazione demograficamente specifica. Non e' noto quanto il SRL generalizzi a cartelle cliniche, psicoterapia, dialoghi orali, altre culture, lingue o fasce d'eta'.

### 13.4 Differenze prestazionali modeste

Il vantaggio sul lessico LIWC e' reale nel test presentato ma piccolo. Il ranking dei modelli non deve essere generalizzato oltre questo dataset e questo target.

### 13.5 Seed e curatela soggettivi

Gli esempi seed influenzano forma e dominio dei token generati. Le fasi manuali 3 e 5 sono eseguite da un solo autore e non hanno una misura di affidabilita' fra curatori.

### 13.6 Affidabilita' clinica soltanto discreta

ICC 0,48 e accordo 0,54 indicano che anche esperti informati non concordano sempre sulla prototipicita' dei token.

### 13.7 Studio non preregistrato

Le ipotesi sulla feature importance derivano da letteratura precedente, ma l'analisi non e' preregistrata. Le conclusioni teoriche vanno quindi considerate esplorative/confermative in misura limitata.

## 14. Limiti metodologici da aggiungere nella lettura critica

Oltre ai limiti discussi dagli autori, e' utile osservare:

- il target e' prodotto all'interno della stessa conversazione da cui vengono estratte le feature; parte della correlazione puo' riflettere il protocollo di assessment;
- le classi del training sono artificialmente bilanciate mentre il test conserva la prevalenza reale;
- l'importanza di una feature in gradient boosting non equivale a un contributo causale o indipendente;
- la validita' di contenuto nella Tabella 2 e' una valutazione qualitativa degli autori, non un indice psicometrico con intervallo d'incertezza;
- l'esclusione del linguaggio del counselor riduce un leakage evidente, ma la persona puo' riprendere le parole del counselor nelle proprie risposte;
- un match lessicale puo' riguardare terze persone, eventi passati o una negazione e non il rischio attuale dell'utente.

## 15. Conclusione reale del paper

Il paper non conclude che i lessici siano superiori ai modelli neurali. Conclude che:

1. gli LLM rendono molto piu' rapida la generazione iniziale di lessici estesi;
2. curatela, calibrazione sui dati ed esperti restano indispensabili;
3. il SRL validato supera modestamente LIWC e raggiunge una semplice baseline di embedding;
4. i modelli fine-tuned ottengono performance superiori;
5. i lessici rimangono utili per interpretabilita', determinismo, privacy, basso costo, piccoli campioni e benchmark di validita' di contenuto;
6. un approccio ibrido lessico + modello neurale + revisione umana e' piu' ragionevole di un lessico usato da solo in un contesto ad alto rischio.

## 16. Collegamento con DAIC-WOZ ed E-DAIC

Il paper puo' essere utile alla tesi, ma non trasferito direttamente.

### 16.1 Differenze di target

| Paper SRL | DAIC-WOZ/E-DAIC |
|---|---|
| rischio suicidario percepito dal counselor, 3 livelli | sintomi depressivi self-report PHQ-8, totale 0-24 o soglia 10 |
| include costrutti direttamente suicidari | il PHQ-8 omette l'item suicidario del PHQ-9 |
| crisis text messaging | intervista orale semi-strutturata trascritta |
| target prodotto durante la crisi | questionario riferito alle due settimane precedenti |

Il SRL non deve essere usato come se fosse una ground truth di depressione o suicidarieta' in DAIC. In particolare, un PHQ-8 alto non annota ideazione suicidaria.

### 16.2 Usi corretti nella tesi

Il SRL puo' essere usato come:

- baseline lessicale separata, chiarendo il domain shift;
- strumento descrittivo sui soli turni del partecipante;
- set di concetti candidati per interrogare feature interne di un LLM;
- controllo di validita' di contenuto per spiegazioni e circuiti;
- sorgente di controfattuali, negazioni e casi di polisemia per testare il modello;
- esempio metodologico per costruire un **Depression/PHQ-8 Lexicon** specifico, con nuova validazione su DAIC.

Non e' corretto usarlo per:

- assegnare etichette cliniche individuali;
- dedurre rischio suicidario dalle label PHQ-8;
- validare un circuito solo perche' attiva parole presenti nel lessico;
- sostituire la verifica causale del modello;
- pubblicare esempi sensibili tratti dai transcript.

### 16.3 Esperimento coerente con la tesi

Un possibile uso controllato e':

```text
Transcript DAIC P-only
        |
        +--> feature SRL, solo come descrittori esplorativi
        |
        +--> LLM che stima PHQ-8 binario
                    |
                    +--> feature/circuiti candidati
                    +--> confronto con costrutti SRL e PHQ-8
                    +--> ablation e counterfactual test
```

La domanda non sarebbe "il SRL diagnostica la depressione?", ma "il modello usa concetti teoricamente nominabili e il loro ruolo e' causale, robusto al contesto e specifico del target?".

## 17. Riproducibilita' e risorse

- [Suicide Risk Lexicon e pacchetto `construct-tracker`](https://github.com/danielmlow/construct-tracker).
- [Codice di riproduzione del paper](https://github.com/danielmlow/lexicon).
- [Articolo, DOI 10.1037/abn0001152](https://doi.org/10.1037/abn0001152).
- Supplemento indicato dall'articolo: [10.1037/abn0001152.supp](https://doi.org/10.1037/abn0001152.supp).

La Crisis Text Line non e' condivisa. E' quindi possibile riprodurre la costruzione e l'uso del lessico su dati propri, ma non ricostruire integralmente gli esperimenti del paper senza accesso autorizzato al corpus sensibile.

## 18. Glossario rapido

| Termine | Significato nel paper |
|---|---|
| Lexicon | insieme di token/frasi organizzati per costrutto |
| SRL | Suicide Risk Lexicon |
| Costrutto | fattore di rischio, sintomo o comportamento teoricamente definito |
| Token match | corrispondenza fra testo e voce del lessico |
| Prototipicita' | quanto un token e' un esempio tipico del costrutto |
| Content validity | copertura dei costrutti rilevanti e qualita' dei loro indicatori |
| Criterion validity | capacita' di predire una label esterna |
| ICC | affidabilita' fra valutatori per giudizi quantitativi |
| Macro RMSE | media degli RMSE calcolati separatamente per classe |
| Spearman rho | correlazione di rango fra rischio vero e predetto |
| Word score | analisi che identifica parole caratteristiche di un corpus rispetto a un altro |
| LGBMRegressor | modello ad alberi gradient boosting usato sui conteggi |

## 19. Punti da ricordare

- GPT-4 genera candidati; non valida il lessico.
- La teoria decide quali costrutti devono essere coperti.
- Il contesto reale decide quali token producono falsi positivi o negativi.
- Gli esperti aumentano la validita', ma l'accordo resta soltanto discreto.
- Il SRL validato e' competitivo con una baseline MiniLM, non con il miglior modello fine-tuned.
- L'importanza delle feature descrive associazioni con il giudizio del counselor, non causalita' o rischio futuro.
- Interpretabilita' senza validita' di contenuto puo' essere fuorviante.
- In DAIC il PHQ-8 non e' una label di suicidarieta': il trasferimento del SRL deve restare esplorativo e validato da zero.
