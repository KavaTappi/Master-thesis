# DAIC-WOZ: feature, scopo e modelli per una tesi di interpretabilita' meccanicistica

## Sintesi esecutiva

DAIC-WOZ e' la porzione Wizard-of-Oz del *Distress Analysis Interview Corpus* (DAIC) resa nota nel contesto AVEC. Contiene interviste semi-strutturate in inglese: il partecipante parla con **Ellie**, un'intervistatrice virtuale animata e controllata da un operatore umano. Il corpus mette a disposizione testo, audio, feature vocali e facciali gia' estratte, oltre a label di questionario.

Per questa tesi il dataset non va presentato come strumento per diagnosticare la depressione. Il suo uso corretto e' studiare un modello che **stima un questionario di screening** e verificare, con tecniche meccanicistiche, quali segnali interni usi e se tali segnali dipendano da shortcut del protocollo.

La pipeline consigliata e' progressiva:

```text
DAIC transcript del partecipante
        |
        +--> baseline testuale e controlli su domanda/lunghezza
        |
        +--> LLM open-weight tracciabile
                   |
                   +--> Jacobian Lens / J-space: concetti intermedi
                   +--> Circuit Tracer: feature e percorsi verso il logit
                   +--> ablation e patching: verifica causale

DAIC audio e feature COVAREP/OpenFace
        |
        +--> baseline multimodale o token acustici interpretabili
        +--> solo dopo: estensione alla fusione multimodale
```

## 1. DAIC, DAIC-WOZ ed E-DAIC: terminologia

| Nome | Che cosa indica | Rilevanza per la tesi |
|---|---|---|
| **DAIC** | Corpus piu' ampio di interviste umane, teleconferenza, Wizard-of-Oz e agente autonomo, creato per studiare distress psicologico. | Contesto scientifico e raccolta multimodale originaria. |
| **DAIC-WOZ** | Sottoinsieme delle interviste con Ellie controllata da un operatore; e' la versione AVEC usata frequentemente per depression severity estimation. | Dataset principale suggerito. |
| **E-DAIC** | Estensione del corpus con ulteriori asset/label in specifiche release. | Da usare solo dopo aver verificato contratto, versioni e disponibilita' effettiva. |

Questo documento descrive la release **DAIC-WOZ Depression Database / AVEC 2017**. Non assumere che ogni asset elencato sia incluso nella propria copia: la licenza e la distribuzione possono cambiare.

## 2. Scopo originario e confine della tesi

Il corpus e' stato creato per supportare studi su depressione, ansia e PTSD tramite interviste cliniche e segnali verbali/non verbali. L'obiettivo originario includeva lo sviluppo di un agente intervistatore e sistemi di identificazione automatica di indicatori di distress.

Il target PHQ-8 e' una misura self-report di sintomi depressivi correnti. E' utile come ground truth supervisionato per un compito di machine learning, ma non e' una diagnosi psichiatrica indipendente. Ne seguono quattro regole:

1. parlare di **stima PHQ-8**, classificazione rispetto a una soglia operativa, oppure supporto allo screening;
2. non descrivere il modello come diagnostico o come sostituto di un clinico;
3. non attribuire un sintomo a una singola frase senza annotazione a livello di turno;
4. distinguere sempre tra comportamento del modello e stato mentale del partecipante.

## 3. Unita' statistica, split e label

### 3.1 Sessioni

La documentazione AVEC descrive **189 sessioni** organizzate in cartelle `300_P` fino a `492_P`; quattro sessioni sono escluse per problemi tecnici. L'unita' primaria per il target PHQ-8 e' il **partecipante/sessione**, non il singolo frame, token o turno.

Gli split ufficiali sono:

| Split | File | Cosa contiene |
|---|---|---|
| Train | `train_split_Depression_AVEC2017.csv` | ID, genere, PHQ8 binary, punteggio totale e risposte ai singoli otto item. |
| Development | `dev_split_Depression_AVEC2017.csv` | Stessi campi di train. |
| Test ufficiale | `test_split_Depression_AVEC2017.csv` | ID e genere; le label di depressione non sono distribuite. |

Non usare il test ufficiale per scegliere modello, soglie, prompt o feature. Se non si puo' sottomettere a una challenge, adottare train/dev in modo trasparente oppure nested cross-validation per partecipante.

### 3.2 Target disponibili

| Campo concettuale | Tipo | Uso appropriato |
|---|---|---|
| `PHQ8_Score` | Intero 0-24, somma degli item | Regressione; MAE/RMSE; analisi di severita'. |
| `PHQ8_Binary` | Classe da soglia `PHQ8_Score >= 10` | Classificazione; logit gap tracciabile con Circuit Tracer. |
| Otto item PHQ-8 | Ordinale 0-3 per partecipante | Multi-task o analisi per dominio sintomatologico. |
| Genere | Metadata | Audit di sottogruppo e controllo, non input predefinito. |

Gli otto item corrispondono a interesse/piacere, umore depresso, sonno, stanchezza, appetito, autovalutazione negativa, concentrazione e rallentamento/agitazione. Queste label sono **a livello di questionario**: non annotano quali parole, risposte o secondi dell'intervista esprimano l'item.

## 4. Asset per sessione

Ogni cartella di sessione puo' includere i file seguenti.

| File | Modalita' | Granularita' e contenuto | Uso per la tesi | Attenzione |
|---|---|---|---|---|
| `XXX_TRANSCRIPT.csv` | Testo | Turni timestampati di Ellie e partecipante; formato tab-separated; alcune sovrapposizioni temporali. | **Core della tesi.** Separare parlanti, mantenere i timestamp e confrontare risposta sola / domanda+risposta. | Parti di Ellie possono avere identificativi generati automaticamente; alcune sessioni non hanno le domande di Ellie nel transcript. |
| `XXX_AUDIO.wav` | Audio | Microfono head-mounted, 16 kHz, registrazione scrubbed. | Baseline acustica; segmentazione per turno; estensione multimodale. | Puo' contenere bleed-over di Ellie; porzioni identificabili sono azzerate. |
| `XXX_COVAREP.csv` | Audio pre-estratto | Feature vocali ogni 10 ms, quindi 100 Hz. | Baseline audio interpretabile e token/proxy acustici per LLM. | Feature vocali non valide nei frame non voiced; entry scrubbed a zero. |
| `XXX_FORMANT.csv` | Audio pre-estratto | Primi cinque formanti lungo l'intervista. | Spazio vocalico, dinamica articolatoria e baseline vocale. | Entry scrubbed a zero; richiede maschera VUV e aggregazione robusta. |
| `XXX_CLNF_features.txt` | Visione | 68 landmark facciali 2D per frame. | Movimento/variabilita' facciale e baseline video. | Coordinate in pixel; usare `confidence` e `detection_success`. |
| `XXX_CLNF_features3D.txt` | Visione | 68 landmark 3D per frame, coordinate mondo. | Distanze e dinamiche meno dipendenti dalla scala immagine. | Coordinate in millimetri rispetto alla camera. |
| `XXX_CLNF_AUs.csv` | Visione | Intensita' (`_r`) e presenza binaria (`_c`) di Action Units facciali. | Baseline facciale relativamente interpretabile. | `_r` e `_c` hanno semantica diversa; non aggregarli indistintamente. |
| `XXX_CLNF_gaze.txt` | Visione | Direzioni di sguardo in coordinate mondo e testa. | Gaze stability, aversione e variabilita'. | Sguardo e pose sono stimati, non annotazioni manuali. |
| `XXX_CLNF_pose.txt` | Visione | Posizione X/Y/Z e rotazioni Rx/Ry/Rz della testa. | Head-motion dynamics e baseline non verbale. | Posizione in mm; rotazioni in radianti. |
| `XXX_CLNF_hog.bin` | Visione | HOG su volto allineato 112x112; vettore 4464 per frame. | Baseline ad alta dimensionalita' o encoder visuale. | File binario; e' meno interpretabile e piu' costoso di AU/pose. |

### 4.1 Transcript: uso corretto

Il transcript e' il punto di ingresso piu' robusto per una tesi su LLM. Preparare almeno quattro rappresentazioni:

| Input | Cosa contiene | Perche' serve |
|---|---|---|
| `P-only` | Solo turni del partecipante. | Input principale; riduce leakage dall'intervistatore. |
| `E-only` | Solo domande/turni di Ellie. | Controllo: misura quanto il protocollo predice il target. |
| `E+P` | Domanda e risposta in ordine. | Baseline contestuale; puo' migliorare predizione ma introdurre shortcut. |
| `P-matched` | Risposte P con lunghezza e domanda bilanciate. | Valuta robustezza a quantita' di testo e tipo di domanda. |

I timestamp consentono di allineare transcript e feature a 100 Hz/frame. L'allineamento e' utile, ma non deve creare un falso target a livello di turno: il PHQ-8 resta un label dell'intera sessione.

### 4.2 Audio e COVAREP

`XXX_COVAREP.csv` include, tra le altre, le feature seguenti:

| Gruppo | Feature | Lettura intuitiva |
|---|---|---|
| Voicing e pitch | `F0`, `VUV` | Frequenza fondamentale e indicatore voiced/unvoiced. |
| Glottal source | `NAQ`, `QOQ`, `H1H2`, `PSP`, `MDQ`, `peakSlope`, `Rd`, `Rd_conf` | Proprietà del ciclo glottale e della qualità vocale. |
| Spettro/cepstrum | `MCEP_0-24`, `HMPDM_0-24`, `HMPDD_0-12` | Descrittori spettrali e dinamici della voce. |
| Risonanza | Formanti F1-F5 in `XXX_FORMANT.csv` | Frequenze di risonanza del tratto vocale. |

`VUV=0` segnala un segmento non sonoro: le feature glottali e `F0` non devono essere considerate come valori misurati. Inoltre, i valori azzerati per scrubbing non sono pause naturali. La pipeline deve distinguerli con il transcript e una maschera di validita'.

### 4.3 Visione: OpenFace/CLNF

Le feature CLNF sono output del toolkit OpenFace, non etichette cliniche. Sono comunque utili per una baseline quantitativa:

- **Action Units:** media, varianza, durata e dinamica di AUs come misure di espressione; non interpretarle isolatamente come emozioni certe.
- **Pose:** velocita', varianza e frequenza di movimenti della testa.
- **Gaze:** stabilita' e cambiamenti di direzione; la documentazione separa coordinate mondo e coordinate relative alla testa.
- **Landmark/HOG:** piu' informazione grezza, ma minore interpretabilita' diretta e maggior rischio di overfitting su un campione piccolo.

Ogni file CLNF include `confidence` in `[0,1]` e un indicatore di successo del tracking. La pulizia deve quindi essere effettuata prima dell'aggregazione temporale, dichiarando soglia di confidenza e gestione dei frame falliti.

## 5. Feature engineering consigliato

La dimensione modesta del dataset rende preferibile l'aggregazione per turno/sessione e modelli regolarizzati come baseline. Non iniziare con un grande modello end-to-end sulle serie a 100 Hz.

| Modalita' | Feature aggregate iniziali | Modello baseline | Controlli |
|---|---|---|---|
| Testo | TF-IDF word/char n-gram, lunghezza, type-token ratio, numero di turni e pause da transcript | Logistic Regression / Ridge | `E-only`, permutazione domande, lunghezza matched. |
| Audio | Media, deviazione, quantili, slope e percentuale VUV per COVAREP/Formanti | Elastic Net / XGBoost leggero | maschera scrubbed, solo turni P, durata parlata. |
| Visione | Statistiche AU, pose, gaze e tracking confidence | Logistic Regression / SVM lineare | frame validi, analisi per sottogruppo. |
| Fusione tardiva | Score calibrati dei tre modelli unimodali | Regressione logistica / Ridge | ablazione di modalita', no leakage tra split. |

La fusione tardiva e' una baseline migliore di una fusione neurale precoce: rende visibile se audio/video aggiungono valore oltre al testo prima di aumentare drasticamente i gradi di liberta'.

## 6. Modelli e strumenti gia' coinvolti nel corpus

Questa tabella separa strumenti di estrazione delle feature dai modelli predittivi della tesi. Non confondere i primi con classificatori di depressione.

| Componente | Ruolo nel dataset | Link | Implicazione |
|---|---|---|---|
| **OpenFace / CLNF** | Estrae landmark, AUs, gaze, pose e HOG dalle registrazioni. | [OpenFace](https://github.com/TadasBaltrusaitis/OpenFace) | Le feature video sono output stimati da questo sistema; occorre trattare confidence/failure. |
| **COVAREP v1.3.2** | Estrae descrittori vocali e glottali dall'audio. | [COVAREP](https://github.com/covarep/covarep) | Consente baseline vocali riproducibili e feature interpretabili per turno. |
| **Formant tracker della release** | Fornisce F1-F5 lungo l'intervista. | [Documentazione AVEC](https://dcapswoz.ict.usc.edu/wp-content/uploads/2022/02/DAICWOZDepression_Documentation.pdf) | Usare con VUV, scrubbed mask e normalizzazione per speaker. |
| **PHQ-8** | Genera i target di questionario. | [Kroenke et al., 2009](https://doi.org/10.1016/j.jad.2008.06.026) | E' una misura di screening, non un modello e non una diagnosi. |

## 7. Modelli predittivi da usare nella tesi

### 7.1 Gerarchia raccomandata

| Livello | Modello | Scopo | Quando fermarsi |
|---|---|---|---|
| 0 | Majority class / media PHQ-8 | Riferimento minimo. | Mai ometterlo. |
| 1 | TF-IDF + Logistic Regression/Ridge | Baseline testuale forte, poco costosa e ispezionabile. | Se un LLM non supera questa baseline, non giustificarne la complessita'. |
| 2 | Audio/video regolarizzati | Misurano contributo delle altre modalita'. | Se non aggiungono valore, non procedere alla fusione neurale. |
| 3 | Fusione tardiva calibrata | Quantifica guadagno multimodale. | Se il guadagno non e' robusto, mantenere la tesi text-first. |
| 4 | Decoder LLM open-weight tracciabile | Audit J-space e Circuit Tracer del comportamento interno. | E' il modello principale per la domanda meccanicistica, non necessariamente il migliore per score puro. |

### 7.2 Modello principale raccomandato: Gemma-2-2B

**[google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b)** e' il candidato iniziale piu' equilibrato:

- decoder-only, quindi adatto a una classificazione generativa a output vincolato;
- dimensione gestibile per esperimenti locali e interventi;
- famiglia supportata da Jacobian Lens su Neuronpedia;
- transcoders disponibili per Circuit Tracer/GemmaScope;
- Circuit Tracer e Neuronpedia hanno supporto documentato per Gemma-2 2B.

Usare una formulazione del tipo:

```text
Transcript del partecipante: <P-only>
Task: output exactly one label: below_10 or at_or_above_10.
Label:
```

Il target meccanicistico e' allora:

```text
logit(at_or_above_10) - logit(below_10)
```

Questo e' piu' semplice da tracciare rispetto a una regressione continua. La regressione `PHQ8_Score` resta una baseline distinta.

### 7.3 Alternativa: Qwen3-4B

**[Qwen/Qwen3-4B](https://huggingface.co/Qwen/Qwen3-4B)** e' un'alternativa utile se la disponibilita' hardware lo consente:

- decoder-only da 4B parametri;
- transcoders Circuit Tracer pubblicati per la famiglia Qwen3, incluso 4B;
- Jacobian Lens disponibile per modelli della famiglia Qwen su Neuronpedia;
- licenza Apache-2.0 sulla model card, che semplifica la riproducibilita' rispetto a licenze con accesso condizionato.

Il prezzo e' una maggiore richiesta di memoria e tempo, soprattutto se si devono salvare attivazioni, grafi e risultati di intervento. Non scegliere Qwen3-4B solo per prestazione: la compatibilita' esatta tra checkpoint, tokenizer, lens e transcoder deve essere verificata e fissata con commit/versioni.

### 7.4 Altre alternative compatibili

| Modello | Perche' considerarlo | Limite principale |
|---|---|---|
| **[Llama-3.2-1B](https://huggingface.co/meta-llama/Llama-3.2-1B)** | Molto leggero; transcoders Circuit Tracer disponibili. | Capacita' linguistica inferiore; richiede accettazione della licenza Meta. |
| **Qwen3 0.6B/1.7B/8B/14B** | Famiglia con transcoders pubblicati; scala flessibile. | Cambiando dimensione cambiano anche il lens, il transcoder e il budget computazionale. |
| **Gemma 3** | Nuovi transcoders disponibili in alcune dimensioni. | Non e' una sostituzione automatica di Gemma-2: verificare intersezione con J-lens e toolchain. |

### 7.5 Regola essenziale su LoRA e fine-tuning

Un modello LLM adattato con LoRA non e' identico al checkpoint per cui sono stati fittati Jacobian Lens e transcoders. Perciò:

1. iniziare da backbone congelato e prompt/target controllati;
2. se serve un adattamento, eseguire LoRA solo dopo i baseline;
3. rivalidare il lens e i transcoders sul checkpoint adattato oppure dichiarare l'analisi post-LoRA come esplorativa;
4. non usare la visualizzazione Neuronpedia come prova unica: esportare script, JSON, pesi, prompt e risultati di ablation.

## 8. Jacobian Lens, J-space e Circuit Tracer nella pipeline

| Strumento | Domanda a cui risponde | Input e output | Condizione per una conclusione |
|---|---|---|---|
| **Jacobian Lens / J-space** | Quali concetti il modello e' predisposto a verbalizzare nei layer intermedi? | Attivazioni residuali -> readout di concetti/tokens per layer e posizione. | Stabilita' su split/esempi/controlli; non basta un readout plausibile. |
| **Circuit Tracer** | Quali token e feature sparse contribuiscono al logit gap del target? | Prompt + target logit -> grafo di feature, edge, token e error nodes. | Ablation/patching deve cambiare selettivamente il logit previsto. |
| **Neuronpedia** | Come esplorare, annotare e condividere grafi/lens? | UI e artefatti di interpretabilita'. | E' una superficie di analisi, non una validazione sperimentale. |

Per definizioni, limiti ed esperimenti dedicati consultare:

- [J-space e Jacobian Lens](jspace-jacobian-lens.md)
- [Circuit Tracer](circuit-tracer.md)

## 9. Multimodalita': percorso corretto

### 9.1 Fase A - Audit testuale (obbligatoria)

Usare `P-only` con Gemma-2-2B o Qwen3-4B, il target binario e controlli `E-only` / `E+P`. Questa fase stabilisce se esiste un meccanismo testuale analizzabile prima di introdurre altri segnali.

### 9.2 Fase B - Feature audio interpretabili

Allineare COVAREP/Formanti ai turni del partecipante e costruire:

- un baseline audio aggregato;
- token discreti e trasparenti, ad esempio `PAUSE_HIGH`, `F0_VARIABILITY_LOW` o `VOICED_RATIO_LOW`, con soglie calcolate soltanto sul training set;
- un input `P-only + audio tokens` per il LLM.

In questa configurazione J-space e Circuit Tracer spiegano come il **LLM usa i proxy acustici tokenizzati**, non come un encoder comprende l'onda audio. E' gia' una forma utile di multimodalita' interpretabile.

### 9.3 Fase C - Video o modello nativamente multimodale (facoltativa)

Un encoder audio/video e un layer di fusione richiedono strumenti che coprano encoder, proiettore e residual stream. Un Circuit Tracer applicato al solo decoder non spiega l'intero modello multimodale. Con il numero ridotto di sessioni, questa fase e' una estensione ad alto rischio: procedere solo se le fasi A e B mostrano valore incrementale e l'hardware/toolchain sono disponibili.

## 10. Esperimenti, metriche e controlli minimi

| Blocco | Esperimento | Metriche / evidenza |
|---|---|---|
| Predizione | `P-only`, `E-only`, `E+P`, TF-IDF e LLM. | Binary: balanced accuracy, macro-F1, AUROC; score: MAE/RMSE. |
| Confondenti | Permutare domanda, normalizzare lunghezza, escludere turni con artefatti. | Delta di prestazione e intervalli bootstrap. |
| J-space | Confrontare top-k concetti per layer tra classi e controlli. | Persistenza, overlap, associazione con logit gap, replicazione su seed. |
| Circuiti | Grafi del logit gap per casi bilanciati; aggregazione di feature/edge. | Overlap pesato e descrizioni da max-activating examples. |
| Causalita' | Ablation e patching di feature candidate e controlli abbinati. | Delta-logit, flip rate, selettivita' rispetto a target linguistico di controllo. |
| Multimodalita' | Testo, audio, fusione tardiva, testo+audio token. | Guadagno incrementale e ablation per modalita'. |
| Robustezza | Seed, split, genere se i campioni lo permettono, e SWMH come dominio esterno. | Intervalli di confidenza e failure table. |

## 11. Qualita' dei dati e rischi specifici

| Rischio | Perche' conta | Mitigazione |
|---|---|---|
| Label a livello partecipante | Un turno puo' ricevere un segnale debole/rumoroso dal PHQ-8 complessivo. | Aggregare a sessione, usare multiple-instance reasoning solo se dichiarato; non fare claim per-turno. |
| Leakage da Ellie | La domanda, la sua presenza o la sequenza possono predire il target. | `E-only`, permutazione domanda, `P-only` come analisi primaria. |
| Piccolo campione | Grande rischio di varianza e overfitting. | Modelli piccoli, regolarizzazione, split fissi, bootstrap, pochi gradi di liberta'. |
| Scrubbing | Zeri in audio/feature non sono dati naturali. | Maschere esplicite e controlli sui timestamp scrubbed. |
| Tracking visuale | Landmark/AU/gaze possono fallire in alcuni frame. | Filtrare per confidence/success, report della percentuale valida. |
| Comorbidita' e confondenti | Depression, PTSD e ansia possono essere correlati. | Non interpretare feature come specifiche di una patologia; analisi condizionate solo se statisticamente sostenibili. |
| Bias demografico | Il comportamento non verbale e la raccolta non sono necessariamente invarianti. | Audit per sottogruppo con incertezza; evitare conclusioni generalizzanti. |

## 12. Etica, privacy e riproducibilita'

Il corpus contiene dati sensibili. Conservare i dati secondo licenza e accordo di accesso; non pubblicare audio, testo riconoscibile, ID dei partecipanti o esempi che possano facilitare re-identificazione. Le figure possono usare statistiche aggregate o esempi sintetici chiaramente marcati.

Ogni esperimento deve salvare:

- versione e hash del dataset e della lista di sessioni;
- regole di esclusione, maschere scrubbed e soglie di tracking;
- split, seed, prompt e tokenizer;
- checkpoint esatto, tipo di precisione e parametri LoRA, se presenti;
- versione/commit del Jacobian Lens, transcoders e Circuit Tracer;
- grafi completi JSON, criterio di pruning e risultati numerici degli interventi;
- risultati negativi e failure table, non solo esempi efficaci.

## 13. Decisione raccomandata per la tesi

**Nucleo consigliato:** DAIC-WOZ transcript `P-only`, target binario PHQ-8, TF-IDF + Logistic Regression come baseline, **Gemma-2-2B** come modello meccanicistico, Jacobian Lens e Circuit Tracer, con ablation/patching e controlli sulla domanda.

**Primo ampliamento:** COVAREP/Formanti aggregati e token acustici interpretabili.

**Secondo ampliamento:** SWMH per verificare il domain shift intervista -> social media, senza trattare le sue label di community come equivalenti al PHQ-8.

**Da non rendere requisito iniziale:** video end-to-end, fusione multimodale profonda, grande fine-tuning del LLM o inferenze diagnostiche.

## Riferimenti e link

- Gratch et al. (2014), *The Distress Analysis Interview Corpus of human and computer interviews*: file locale `misc/DAIC-docs/gratch_etal_2014.pdf`.
- [DAIC-WOZ Depression Database documentation, AVEC 2017](https://dcapswoz.ict.usc.edu/wp-content/uploads/2022/02/DAICWOZDepression_Documentation.pdf).
- [OpenFace](https://github.com/TadasBaltrusaitis/OpenFace) e file locale `misc/DAIC-docs/scherer_stratou_etal_2014.pdf`.
- [COVAREP](https://github.com/covarep/covarep) e file locale `misc/DAIC-docs/COVAREP_2014.pdf`.
- [Gemma-2-2B](https://huggingface.co/google/gemma-2-2b), [Qwen3-4B](https://huggingface.co/Qwen/Qwen3-4B), [Llama-3.2-1B](https://huggingface.co/meta-llama/Llama-3.2-1B).
- [Circuit Tracer repository](https://github.com/decoderesearch/circuit-tracer) e [available transcoders](https://github.com/decoderesearch/circuit-tracer/blob/main/README.md).
- [Neuronpedia Jacobian Lens](https://www.neuronpedia.org/blog/jacobian-lens), [published fitted lenses](https://huggingface.co/neuronpedia/jacobian-lens) e [Neuronpedia Circuit Tracer](https://www.neuronpedia.org/blog/circuit-tracer).
