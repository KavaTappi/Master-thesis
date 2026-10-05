# DAIC-WOZ ed E-DAIC: guida completa al corpus, alle release e all'uso sperimentale

## Sintesi

**DAIC** (*Distress Analysis Interview Corpus*) e' il corpus generale descritto da Gratch et al. (2014). Comprende interviste faccia a faccia, in teleconferenza, con agente virtuale controllato da esseri umani e con agente autonomo. **DAIC-WOZ** e' la release composta dalle interviste *Wizard-of-Oz* con Ellie e resa celebre dalle challenge AVEC 2016/2017. **E-DAIC** (*Extended DAIC*) estende il benchmark con interviste in parte WoZ e in parte condotte da Ellie in modalita' autonoma; e' la base del task *Detecting Depression with AI* di AVEC 2019.

I tre nomi non sono sinonimi:

| Nome | Ambito | Protocollo dell'intervistatore | Dimensione da citare |
|---|---|---|---:|
| DAIC | Corpus originario complessivo | umano in presenza, umano via teleconferenza, Ellie WoZ, Ellie autonoma | 621 interviste censite nel paper LREC 2014, in quattro condizioni |
| DAIC-WOZ | Release WoZ per depressione | Ellie controllata in tempo reale da operatori umani | 189 sessioni distribuite; 107 train, 35 development, 47 test |
| E-DAIC / Extended DAIC | Estensione usata da AVEC 2019 | mix WoZ e autonomo in train/dev; solo autonomo nel test della challenge | 275 sessioni nel benchmark AVEC 2019; 163 train, 56 development, 56 test |

L'unita' statistica e' sempre il **partecipante/sessione**. Il PHQ-8 e le altre scale sono label a livello di intervista, non annotazioni di singoli turni, frasi, token o frame.

## 1. Come nasce DAIC

Il corpus fu raccolto dall'USC Institute for Creative Technologies per studiare indicatori verbali, vocali, visivi e fisiologici di distress psicologico e per sviluppare un agente capace di condurre un'intervista semi-strutturata. Lo scopo era il supporto alla valutazione e allo screening, non la sostituzione della diagnosi clinica.

Il paper originario distingue quattro condizioni:

| Condizione | Numero di interviste nel censimento del 2014 | Caratteristica |
|---|---:|---|
| Faccia a faccia | 120 | partecipante e intervistatrice nella stessa stanza |
| Teleconferenza | 45 | intervistatrice umana mostrata su uno schermo |
| Wizard-of-Oz | 193 | Ellie animata, controllata da operatori umani |
| Agente autonomo | 263 | Ellie governata dalle policy e dai moduli automatici del sistema |
| **Totale** | **621** | corpus DAIC complessivo allora censito |

Questi 621 colloqui non coincidono con le 189 sessioni della release DAIC-WOZ. Quest'ultima seleziona il ramo WoZ e ne esclude quattro per problemi tecnici.

### 1.1 Partecipanti e raccolta

I partecipanti provenivano dall'area metropolitana di Los Angeles e da due canali principali: annunci rivolti alla popolazione generale e reclutamento presso una struttura per veterani. Tutti erano fluenti in inglese e le interviste furono svolte in inglese.

Il protocollo generale comprendeva:

1. consenso informato, con consenso opzionale alla condivisione dei dati per ricerca;
2. questionari pre-intervista compilati al computer;
3. intervista semi-strutturata registrata;
4. questionari post-intervista.

L'intervista seguiva una progressione intenzionale:

```text
domande neutrali e di rapport
        -> eventi personali e benessere
        -> sintomi collegati a depressione e PTSD
        -> fase finale di decompressione o "cool-down"
```

Le interviste umane duravano tipicamente 30-60 minuti; il paper originario riporta circa 5-20 minuti per le sessioni WoZ e 15-25 minuti per quelle autonome. Il sito di distribuzione della release DAIC-WOZ descrive le 189 sessioni condivise come comprese tra 7 e 33 minuti, con media di circa 16 minuti. Le due indicazioni si riferiscono a inventari e momenti diversi della raccolta.

### 1.2 Che cos'e' il Wizard-of-Oz

In DAIC-WOZ il partecipante vede Ellie e interagisce con lei come con un agente virtuale. Dietro le quinte, due operatrici controllano il comportamento:

- una seleziona gli enunciati verbali;
- una gestisce segnali non verbali come cenni del capo, espressioni e gesti.

Gli enunciati di Ellie appartengono a un insieme prefissato di registrazioni e animazioni. La scelta e la sequenza, invece, dipendono dal contesto e dalle decisioni dell'operatore. La policy d'intervista fu resa progressivamente piu' strutturata durante la raccolta. Ne segue che la domanda di Ellie non e' un contesto neutro: puo' rivelare il percorso dell'intervista, cio' che il partecipante ha detto prima e le scelte dell'operatore.

### 1.3 Agente autonomo

Nelle sessioni autonome non vi e' selezione umana in tempo reale. Percezione, comprensione e generazione del comportamento dipendono dai moduli e dalle policy implementate nel sistema. E-DAIC usa queste sessioni per costruire un test che misura anche il passaggio di dominio da interviste almeno in parte guidate da esseri umani a interviste interamente governate dall'AI.

## 2. DAIC-WOZ Depression Database

### 2.1 Inventario e identificativi

La documentazione ufficiale descrive **189 cartelle** con ID nell'intervallo `300`-`492`:

```text
Pack/
  300_P/
  301_P/
  ...
  492_P/
  util/
  documents/
  train_split_Depression_AVEC2017.csv
  dev_split_Depression_AVEC2017.csv
  test_split_Depression_AVEC2017.csv
```

Le sessioni `342`, `394`, `398` e `460` sono escluse per ragioni tecniche. Il conteggio e' quindi `193 - 4 = 189`.

La documentazione segnala inoltre casi speciali da non ignorare:

| Sessione | Problema noto |
|---|---|
| `373` | interruzione intorno a 5:52-7:00 per un problema tecnico |
| `444` | interruzione intorno a 4:46-6:27 per il telefono del partecipante |
| `451`, `458`, `480` | manca la parte di Ellie nel transcript; la parte del partecipante e' presente |
| `402` | il video termina circa due minuti prima della conversazione |

Questi casi possono alterare durata, numero di turni, disponibilita' di domande e statistiche visive. Devono comparire in una tabella di qualita' dei dati e non essere trattati come sessioni ordinarie.

### 2.2 Split ufficiali AVEC 2017

| Split | Sessioni | Label distribuite nel pacchetto documentato | Uso corretto |
|---|---:|---|---|
| Train | 107 | genere, `PHQ8_Binary`, `PHQ8_Score`, otto item PHQ-8 | addestramento e stima dei parametri |
| Development | 35 | stessi campi del train | selezione del modello e degli iperparametri |
| Test | 47 | ID e genere; i target non sono inclusi nel file ufficiale documentato | valutazione challenge, non tuning |
| **Totale** | **189** |  |  |

Molti lavori valutano sul development set perche' le label ufficiali del test non erano disponibili ai partecipanti. Prima di riprodurre un risultato bisogna quindi verificare se "test" indica il vero test AVEC, il development usato come test, oppure un nuovo split creato dagli autori.

### 2.3 PHQ-8

Il *Patient Health Questionnaire-8* misura otto domini sintomatologici nelle due settimane precedenti. Ogni item assume valore 0-3 e il totale varia da 0 a 24.

| Item | Dominio sintetico |
|---:|---|
| 1 | riduzione di interesse o piacere |
| 2 | umore depresso |
| 3 | problemi di sonno |
| 4 | stanchezza o bassa energia |
| 5 | alterazioni dell'appetito |
| 6 | autovalutazione negativa o senso di fallimento |
| 7 | difficolta' di concentrazione |
| 8 | rallentamento o agitazione psicomotoria |

I due target usati piu' spesso sono:

- `PHQ8_Score`: regressione sul totale 0-24;
- `PHQ8_Binary`: classificazione con soglia `PHQ8_Score >= 10`.

Il PHQ-8 non contiene l'item del PHQ-9 relativo a pensieri di morte o autolesionismo. Una classificazione rispetto alla soglia 10 e' una stima di screening, non una diagnosi indipendente. Gli item sono riferiti alla sessione/partecipante: non indicano quale turno esprima un sintomo.

### 2.4 File di sessione

La release documentata elenca:

```text
XXX_P/
  XXX_AUDIO.wav
  XXX_TRANSCRIPT.csv
  XXX_COVAREP.csv
  XXX_FORMANT.csv
  XXX_CLNF_features.txt
  XXX_CLNF_features3D.txt
  XXX_CLNF_AUs.csv
  XXX_CLNF_gaze.txt
  XXX_CLNF_pose.txt
  XXX_CLNF_hog.bin
```

Le registrazioni originali comprendevano audio e video, ma il pacchetto condiviso descritto dalla documentazione non elenca il video grezzo: distribuisce audio, transcript e feature visive gia' estratte. Non bisogna promettere un file video se non e' presente nella copia ottenuta con la licenza.

#### Transcript

`XXX_TRANSCRIPT.csv` contiene turni timestampati e un'etichetta di parlante. In genere i parlanti di interesse sono `Ellie` e `Participant`. Le trascrizioni furono segmentate in corrispondenza di silenzi di almeno circa 300 ms e sottoposte a revisione; le informazioni identificative furono annotate e rimosse o oscurate.

Punti operativi:

- il file e' comunemente letto come tab-separated anche se l'estensione e' `.csv`;
- i timestamp permettono l'allineamento con audio e feature visive;
- i turni di Ellie vanno conservati separatamente da quelli del partecipante;
- le sessioni con domande di Ellie mancanti non possono entrare senza cautela in analisi `E-only` o domanda-risposta;
- sovrapposizioni e silenzi non devono essere eliminati prima di averne salvato una rappresentazione.

#### Audio e COVAREP

`XXX_AUDIO.wav` contiene l'audio del partecipante, distribuito a 16 kHz nella release AVEC. Le porzioni sensibili possono essere state *scrubbed*, cioe' azzerate. Uno zero non e' quindi necessariamente silenzio naturale.

`XXX_COVAREP.csv` contiene low-level descriptors a passo temporale fine, tipicamente 10 ms. I gruppi principali includono:

- frequenza fondamentale e voicing (`F0`, `VUV`);
- descrittori glottali e di qualita' vocale (`NAQ`, `QOQ`, `H1H2`, `PSP`, `MDQ`, `peakSlope`, `Rd` e confidenza);
- coefficienti cepstrali e harmonic model phase (`MCEP`, `HMPDM`, `HMPDD`).

Quando `VUV=0`, molte misure vocali non hanno il significato di un valore osservato. Bisogna mascherarle, non mediare gli zeri come se fossero misure reali.

`XXX_FORMANT.csv` fornisce i primi cinque formanti. Anche qui gli zeri dovuti allo scrubbing vanno distinti dai dati validi.

#### Feature visive CLNF/OpenFace

| File | Contenuto | Dettagli essenziali |
|---|---|---|
| `CLNF_features.txt` | 68 landmark 2D | coordinate pixel, frame, timestamp, confidence, successo del tracking |
| `CLNF_features3D.txt` | 68 landmark 3D | coordinate in millimetri nello spazio della camera |
| `CLNF_AUs.csv` | Action Units facciali | suffisso `_r` per intensita' regressa, `_c` per presenza binaria |
| `CLNF_gaze.txt` | direzioni dello sguardo | vettori nello spazio globale e relativo alla testa |
| `CLNF_pose.txt` | posizione e rotazione della testa | posizione in mm, rotazioni Euler in radianti |
| `CLNF_hog.bin` | HOG del volto allineato | area 112x112, vettore di 4464 valori per frame |

Ogni pipeline visiva deve filtrare o pesare i frame usando `confidence` e `detection_success`. Le Action Units sono stime automatiche, non etichette certe di emozioni o stati clinici.

## 3. E-DAIC / Extended DAIC

### 3.1 Che cosa estende

E-DAIC amplia DAIC-WOZ con sessioni condotte da Ellie in modalita' autonoma e con un set di feature predisposto per AVEC 2019. Train e development mescolano sessioni WoZ e autonome; il test AVEC 2019 contiene solo sessioni autonome. Il benchmark misura quindi sia la stima del PHQ-8 sia la robustezza al cambiamento del meccanismo d'intervista.

Gli ID distinguono il protocollo:

- `300`-`492`: agente controllato in modalita' WoZ;
- `600`-`718`: agente controllato dall'AI.

Questi sono intervalli, non conteggi: alcuni ID possono non essere inclusi nella specifica distribuzione.

### 3.2 Conteggi AVEC 2019

Il paper ufficiale AVEC 2019 riporta:

| Split | Partecipanti | Durata totale | Protocollo |
|---|---:|---:|---|
| Train | 163 | 43:30:20 | mix WoZ e AI |
| Development | 56 | 14:47:31 | mix WoZ e AI |
| Test | 56 | 14:52:42 | solo AI autonomo |
| **Totale** | **275** | **73:10:33** |  |

La separazione non e' un semplice split casuale i.i.d.: nel test il tipo di intervistatore cambia. Una caduta di prestazione puo' dipendere dal cambio di protocollo, dalla qualita' della trascrizione, dalla distribuzione dei partecipanti o da tutti questi fattori.

### 3.3 Perche' il manuale E-DAIC parla di 219 cartelle

Il manuale di distribuzione piu' recente dichiara **219 participant directories**, numero uguale a `163 train + 56 development`. Il paper della challenge descrive invece l'intero benchmark di **275** sessioni, includendo le 56 del test.

Questi due numeri vanno riportati con il loro ambito:

- **275** = inventario completo del benchmark AVEC 2019;
- **219** = cartelle partecipante descritte dal manuale della distribuzione corrente consultata.

Non si deve dedurre che un archivio locale contenga il test solo perche' e' chiamato E-DAIC. Al momento dell'accesso bisogna produrre un manifest reale di cartelle, ID, file e label e confrontarlo con gli split forniti.

### 3.4 Struttura della distribuzione documentata

```text
XXX_P/
  XXX_AUDIO.wav
  XXX_Transcript.csv
  features/
    XXX_BoAW_openSMILE_2.3.0_eGeMAPS.csv
    XXX_BoAW_openSMILE_2.3.0_MFCC.csv
    XXX_BoVW_openFace_2.1.0_Pose_Gaze_AUs.csv
    XXX_CNN_ResNet.mat
    XXX_CNN_VGG.mat
    XXX_densenet201.csv
    XXX_OpenFace2.1.0_Pose_gaze_AUs.csv
    XXX_OpenSMILE2.3.0_egemaps.csv
    XXX_OpenSMILE2.3.0_mfcc.csv
    XXX_vgg16.csv
```

La release E-DAIC e' quindi diversa da DAIC-WOZ anche per le rappresentazioni distribuite. Non basta cambiare il percorso del dataset mantenendo lo stesso loader.

### 3.5 Feature E-DAIC

| Famiglia | Modalita' | Descrizione |
|---|---|---|
| BoAW eGeMAPS | audio | eGeMAPS quantizzato e riassunto su finestre di 4 s con passo di 1 s |
| BoAW MFCC | audio | MFCC quantizzati su finestre di 4 s con passo di 1 s |
| BoVW Pose/Gaze/AU | video | distribuzione di pose, sguardo e Action Units su blocchi temporali |
| openSMILE eGeMAPS | audio | 88 misure spettrali, cepstrali, prosodiche e di qualita' vocale |
| openSMILE MFCC | audio | MFCC 1-13 con delta e delta-delta |
| DenseNet/VGG audio | audio profondo | mel-spettrogrammi con 128 bande, finestra 4 s e hop 1 s inviati a CNN preaddestrate |
| OpenFace 2.1 | video esperto | intensita' di 17 FAU, posa, sguardo e confidenza per frame |
| ResNet/VGG visuali | video profondo | volto rilevato e allineato, poi codificato con reti preaddestrate a pesi congelati |

Il paper AVEC 2019 e il manuale corrente non elencano sempre esattamente gli stessi artefatti. La challenge descrive anche rappresentazioni baseline aggiuntive. La regola corretta e' documentare il contenuto effettivo dell'archivio ricevuto e citare la release, non assumere che ogni copia E-DAIC sia identica.

### 3.6 Label E-DAIC

Il manuale indica per i file di train/development/test:

- `Participant_ID`;
- genere;
- `PHQ_Binary`;
- `PHQ_Score`;
- punteggio `PCL-C` per PTSD;
- severita' PTSD.

`Detailed_PHQ8_Labels.csv` contiene inoltre le risposte ai singoli item PHQ-8. La disponibilita' effettiva delle label del test dipende dalla distribuzione e dal contratto di accesso: durante la challenge i target del test erano nascosti ai partecipanti.

### 3.7 Trascrizioni

Il paper AVEC 2019 descrive testo trascritto automaticamente con Google Cloud Speech Recognition. DAIC-WOZ, invece, distribuisce trascrizioni costruite e revisionate secondo il protocollo del corpus. Lavori successivi hanno anche prodotto nuove trascrizioni con altri sistemi ASR.

Pertanto "testo E-DAIC" puo' indicare versioni diverse. Ogni esperimento deve registrare:

- origine del transcript;
- versione del sistema ASR, se automatica;
- presenza o assenza dei turni di Ellie;
- regole di punteggiatura e segmentazione;
- correzioni manuali;
- esclusione di turni vuoti o non riconosciuti.

## 4. Confronto diretto

| Aspetto | DAIC-WOZ | E-DAIC |
|---|---|---|
| Intervistatore | Ellie controllata da esseri umani | WoZ e Ellie autonoma |
| Benchmark principale | AVEC 2016/2017 | AVEC 2019 DDS |
| Totale benchmark | 189 | 275 |
| Split | 107/35/47 | 163/56/56 |
| Test | WoZ, label originariamente nascoste | solo AI autonomo, label challenge nascoste |
| Testo | transcript revisionati del corpus | transcript automatici nella challenge; possono esistere revisioni successive |
| Audio | WAV e COVAREP/Formant | WAV, openSMILE, BoAW e deep audio representations |
| Visione distribuita | CLNF/OpenFace, landmark, AU, gaze, pose, HOG | OpenFace 2.1, BoVW e deep visual representations |
| Target depressione | PHQ-8 totale/binario e item per train/dev | PHQ-8 totale/binario e item secondo la distribuzione |
| Target PTSD | parte del contesto DAIC, non il target AVEC 2017 principale | PCL-C e severita' PTSD elencati dal manuale |
| Rischio metodologico dominante | leakage dal percorso scelto dal wizard | leakage piu' domain shift WoZ -> AI |

## 5. Sovrapposizione tra DAIC-WOZ ed E-DAIC

E-DAIC e' un'estensione, non un corpus esterno indipendente. Include materiale WoZ nello stesso intervallo di ID di DAIC-WOZ e aggiunge sessioni autonome. Non e' lecito addestrare su tutto DAIC-WOZ e presentare un sottoinsieme E-DAIC come test esterno senza controllare gli ID.

Procedura minima:

```text
1. estrarre gli ID da tutte le cartelle e da tutti i file split;
2. calcolare intersezione DAIC-WOZ <-> E-DAIC;
3. verificare se lo stesso ID indica la stessa sessione o una nuova registrazione;
4. impedire che lo stesso partecipante compaia in train e test;
5. documentare separatamente protocollo WoZ e protocollo AI.
```

La separazione per file o per turno non basta: lo split deve essere a livello di partecipante.

## 6. Leakage e shortcut del protocollo

### 6.1 Domande di Ellie

Le domande possono diventare predittive perche' il wizard o la policy autonoma scelgono follow-up sulla base delle risposte precedenti. Un modello `E+P` puo' quindi imparare il percorso dell'intervistatore anziche' segnali nella risposta del partecipante.

Sono necessari almeno quattro input:

| Variante | Contenuto | Domanda sperimentale |
|---|---|---|
| `P-only` | soli turni del partecipante | quanto e' predittivo il linguaggio del partecipante? |
| `E-only` | soli turni di Ellie | quanto e' predittivo il protocollo? |
| `E+P` | dialogo completo | quanto aiuta il contesto e quanto introduce shortcut? |
| domanda permutata | risposte abbinate a domande di altre sessioni compatibili | il modello usa davvero la relazione domanda-risposta? |

Una prestazione elevata di `E-only` non prova che Ellie "diagnostichi" il partecipante: segnala dipendenza statistica dal percorso di intervista.

### 6.2 Durata e quantita' di testo

Numero di turni, lunghezza delle risposte, durata parlata e presenza di follow-up possono correlare con il target. Vanno incluse baseline che usano solo queste quantita' e confronti *length-matched*.

### 6.3 Scrubbing e missingness

Zeri nell'audio, frame falliti, domande mancanti e file incompleti sono pattern potenzialmente predittivi. Le maschere di validita' devono essere input espliciti per l'analisi di qualita', non feature cliniche implicite.

### 6.4 Label leakage

Le risposte al PHQ-8 non devono essere concatenate al transcript usato come input. Se il protocollo include domande molto vicine agli item, bisogna dichiararlo e distinguere stima del questionario da rilevazione spontanea dei sintomi.

## 7. Protocollo sperimentale consigliato

### 7.1 Audit iniziale

1. creare un manifest per sessione con ID, split, protocollo, file presenti, durata e anomalie;
2. verificare disgiunzione a livello di partecipante;
3. fissare una rappresentazione primaria `P-only`;
4. usare `E-only` ed `E+P` come controlli;
5. stimare le trasformazioni solo sul train;
6. scegliere iperparametri solo sul development;
7. toccare il test una volta, con pipeline congelata.

### 7.2 Baseline minime

| Modalita' | Baseline trasparente | Controllo obbligatorio |
|---|---|---|
| Testo | TF-IDF word/character n-gram + regressione logistica o Ridge | lunghezza, `E-only`, domande permutate |
| Audio | statistiche robuste di COVAREP/eGeMAPS + modello regolarizzato | VUV, durata parlata, maschera scrubbing |
| Video | statistiche AU/pose/gaze + modello lineare/SVM | confidence e percentuale di frame validi |
| Multimodale | fusione tardiva di score calibrati | ablazione di ogni modalita' |

Per classificazione binaria riportare balanced accuracy, macro-F1, recall/specificita' e AUROC con intervalli di confidenza. Per regressione riportare MAE e RMSE; se si confronta con AVEC 2019, usare anche CCC quando richiesto dal protocollo della challenge.

### 7.3 Interpretabilita'

Per una tesi di interpretabilita' meccanicistica, il target piu' semplice da tracciare e' il gap tra le classi `PHQ8_Score >= 10` e `< 10`. L'analisi interna del modello va comunque preceduta da:

- confronto con una baseline lineare;
- audit degli shortcut di Ellie;
- test di robustezza alla lunghezza;
- verifica causale tramite ablation o patching, non sola visualizzazione.

## 8. Cosa si puo' e non si puo' concludere

Conclusioni ammesse:

- il modello stima il PHQ-8 o la sua soglia nella specifica release;
- una modalita' aggiunge informazione predittiva rispetto a una baseline;
- il modello dipende da particolari parole, turni o feature sotto interventi controllati;
- esiste un domain shift tra protocollo WoZ e protocollo autonomo.

Conclusioni non giustificate:

- il sistema diagnostica la depressione;
- una singola frase prova un sintomo;
- un'Action Unit o una caratteristica vocale indica da sola uno stato mentale;
- E-DAIC e DAIC-WOZ sono train e test indipendenti per definizione;
- una migliore performance implica validita' clinica o possibilita' di deployment.

## 9. Etica, accesso e riproducibilita'

L'accesso ufficiale e' limitato a ricerca accademica/non-profit e richiede accettazione della licenza. I dati contengono materiale sensibile anche se de-identificato.

Non pubblicare:

- transcript riconoscibili o esempi testuali reali non autorizzati;
- audio o ricostruzioni vocali;
- ID associati a predizioni individuali;
- frame o feature che facilitino re-identificazione;
- copie del corpus o derivate vietate dalla licenza.

Salvare invece, per ogni esperimento:

- hash o versione del pacchetto e manifest degli ID;
- split e intersezioni fra release;
- regole di esclusione;
- versione del transcript;
- preprocessing, seed e iperparametri;
- percentuale di dati validi per modalita';
- metriche con incertezza e risultati negativi.

## 10. Checklist prima di usare il corpus

- [ ] Ho distinto DAIC, DAIC-WOZ ed E-DAIC nel testo della tesi.
- [ ] Ho contato le cartelle effettivamente ricevute, senza assumere 189, 219 o 275.
- [ ] Ho verificato gli ID sovrapposti tra le release.
- [ ] Ho documentato se il transcript e' manuale, Google ASR, Whisper o altro.
- [ ] Ho separato `P-only`, `E-only` ed `E+P`.
- [ ] Ho controllato le sessioni speciali e i file mancanti.
- [ ] Ho trattato zeri scrubbed e frame falliti come missingness.
- [ ] Ho impedito leakage a livello di partecipante.
- [ ] Ho usato il PHQ-8 come misura di screening, non come diagnosi.
- [ ] Ho rispettato licenza e vincoli di privacy.

## Fonti primarie

- [Gratch et al. (2014), *The Distress Analysis Interview Corpus of Human and Computer Interviews*](../misc/DAIC-docs/gratch_etal_2014.pdf).
- [Sito ufficiale DAIC-WOZ ed Extended DAIC](https://dcapswoz.ict.usc.edu/).
- [Documentazione ufficiale DAIC-WOZ Depression Database / AVEC 2017](https://dcapswoz.ict.usc.edu/wp-content/uploads/2022/02/DAICWOZDepression_Documentation.pdf).
- [Manuale E-DAIC](https://dcapswoz.ict.usc.edu/wwwedaic/E-DAIC%20Manual.pdf).
- [Ringeval et al. (2019), *AVEC 2019 Workshop and Challenge*](https://arxiv.org/pdf/1907.11510).
- [Kroenke et al., documentazione PHQ-8 disponibile nel progetto](../misc/DAIC-docs/PHQ8.pdf).

## Nota sui numeri

Questa guida usa i conteggi delle fonti ufficiali e li associa alla loro specifica release. Se un articolo secondario riporta numeri diversi, prevalgono il manifest della copia realmente usata e la documentazione allegata a quella copia. Questa precauzione e' particolarmente importante per E-DAIC, dove il benchmark AVEC 2019 da 275 sessioni e la distribuzione corrente documentata con 219 cartelle non descrivono lo stesso perimetro.
