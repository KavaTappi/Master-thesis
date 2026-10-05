---
title: Bibliografia della tesi
permalink: /bibliografia/
layout: default
description: Bibliografia ragionata per il piano della tesi sugli LLM, DAIC-WOZ ed explainability meccanicistica.
---

# Bibliografia ragionata per: lessici, shortcut e interpretabilità meccanicistica nella predizione della depressione

**Ricerca aggiornata al 5 ottobre 2026.** Riferimento progettuale: [piano della tesi]({{ '/' | relative_url }}).

## 1. Esito della ricerca

**La proposta ha precedenti molto vicini per ciascuna delle sue componenti. Il contributo potenzialmente originale è la loro integrazione in un audit causale degli indizi dell'intervistatore, non l'uso isolato di LLM, lessici o activation patching in ambito clinico.**

I tre risultati che cambiano maggiormente il posizionamento sono:

1. **Watawana et al., LREC 2026 [R02]** estendono già il lavoro di Burdisso a E-DAIC e ANDROIDS. Il confronto tra corpus, da solo, non basta come novità.
2. **Ahsan et al., Findings EMNLP 2025 [R03]** usano già interventi sulle attivazioni di LLM in ambito sanitario, anche per studiare decisioni sul rischio depressivo. Non si può rivendicare la prima applicazione del patching alla depressione.
3. **Milintsevich et al., ClinicalNLP 2024 [R06]** integrano già lessici in modelli transformer per stimare sintomi depressivi su DAIC-WOZ. Il solo collegamento «lessico + transformer + DAIC-WOZ» non è nuovo.

**Nelle fonti selezionate non ho individuato un lavoro che combini esplicitamente tutti questi elementi:** LLM generativo a pesi accessibili; DAIC-WOZ/E-DAIC; distinzione tra indizi di Ellie ed evidenze del partecipante; lessico costruito con LLM e revisionato; verifica tramite interventi interni; confronto dell'utilità del lessico con controlli abbinati. È un possibile spazio di ricerca, non una dimostrazione di assenza di precedenti.

## 2. Metodo e limiti della ricerca

Questa è una **ricerca bibliografica mirata**, non una revisione sistematica esaustiva o una meta-analisi. Ho cercato lavori vicini al problema, ai dati e al metodo, seguendo anche riferimenti e repository degli autori.

Fonti usate per verificare i riferimenti: ACL Anthology, arXiv, PLOS/PMC/PubMed, atti ICLR e PMLR, repository istituzionale Idiap, pagine editoriali e repository degli autori. Le sintesi di siti aggregatori sono servite soltanto a individuare candidati; le schede sotto rimandano a fonti primarie.

Principali interrogazioni utilizzate, in diverse combinazioni:

- `"DAIC-WOZ" "mechanistic" interpretability`
- `"depression" "activation patching" language model`
- `"DAIC-WOZ" "shortcut" LLM`
- `"mental health" "mechanistic interpretability"`
- `"DAIC-WOZ" "bias" "2026" interviewer`
- `depression LLM interpretable lexicon causal concepts DAIC WOZ`
- `"lexicon" "activation patching"`
- `"lexicon" "mechanistic interpretability" clinical`
- Ricerche per titolo, autore e DOI dei lavori pertinenti.

**Livello di verifica:** per i lavori più vicini ho consultato anche sezioni del testo integrale, in particolare metodi, dati e risultati. Per gli altri la selezione è basata su abstract e metadati, come indicato nelle schede. Non sono stati replicati esperimenti né verificati i repository tramite esecuzione. «Preprint» indica la versione verificata qui: non esclude una successiva pubblicazione non individuata. Si preferisce la versione pubblicata quando rintracciata; preprint e articolo dello stesso lavoro non sono contati due volte.

Sono esclusi dalla bibliografia principale i lavori che condividono soltanto il termine «depressione», gli autoencoder usati come semplici estrattori acustici e la causalità dei fenomeni clinici senza analisi dei meccanismi del modello.

## 3. Mappa dei precedenti più vicini

| Riferimento | Sovrapposizione principale | Differenza rilevante rispetto alla proposta |
|---|---|---|
| R01–R02, Burdisso/Watawana | Bias dell'intervistatore, DAIC-WOZ ed estensione ad altri corpus | GCN/Longformer; non l'audit proposto delle attivazioni di un LLM generativo |
| R03, Ahsan | Interventi interni e decisioni cliniche, anche sul rischio depressivo | Bias demografico in vignette e note cliniche, non indizi di Ellie |
| R04, Lee | LLM, fattori interpretabili, DAIC-WOZ ed E-DAIC | Trasparenza della pipeline predittiva; non verifica dei meccanismi interni dell'estrattore |
| R05, Low | Lessico specialistico costruito con LLM e valutato da clinici | Rischio suicidario; non circuiti dell'LLM predittivo |
| R06–R07, Milintsevich/Villatoro-Tello | Lessici e depressione nelle interviste | Incorporazione/estrazione lessicale; non patching guidato dal lessico |
| R08, Zhang–Poellabauer | Mitigazione del bias dell'intervistatore | Addestramento adversarial di un modello multimodale |
| R13, Wu | Rappresentazioni interne interpretabili in testi clinici | Codifica ICD e dizionario di feature latenti, non lessico linguistico generato |
| R14–R18 | Fedeltà, patching, allineamento a concetti, circuiti | Metodi trasferibili; non una validazione su questo specifico problema clinico |

## 4. Precedenti diretti: lettura prioritaria

### R01 — Burdisso et al. (2024): punto di partenza del problema

**Sergio Burdisso, Ernesto Reyes-Ramírez, Esaú Villatoro-Tello, Fernando Sánchez-Vega, Adrian Lopez Monroy e Petr Motlicek.** *DAIC-WOZ: On the Validity of Using the Therapist’s prompts in Automatic Depression Detection from Clinical Interviews*. ClinicalNLP 2024, pp. 82–90. DOI: `10.18653/v1/2024.clinicalnlp-1.8`.

[Articolo e PDF](https://aclanthology.org/2024.clinicalnlp-1.8/) · [Codice degli autori](https://github.com/idiap/bias_in_daic-woz)

**Contributo:** confronta testo dell'intervistatore e del partecipante con GCN e LongBERT/Longformer; individua domande e regioni dell'intervista capaci di sostenere scorciatoie predittive.

**Per la tesi:** definisce il controllo comportamentale di partenza: addestrare o valutare condizioni *interviewer-only*, *participant-only* e dialogo completo su split comparabili, senza interpretare la prima come prova dell'uso nel terzo caso (E2 contro E3). La regione dell'intervista sulle esperienze pregresse di salute mentale suggerisce quali domande candidare alla costruzione delle coppie originali/varianti; le categorie e gli span finali vanno scelti sul training/development. Il lavoro pone la domanda sullo shortcut, mentre la tesi aggiungerebbe il test di sensibilità dello specifico LLM al cambiamento delle domande e, se emerge un effetto, gli interventi sulle sue attivazioni. Le parole discriminanti del paper non sono già una mappa dei meccanismi interni del LLM.

**Da riprendere:** distinguere informazione disponibile nelle domande da informazione effettivamente usata dal modello quando vede il dialogo completo.

**Verifica:** pagina editoriale e sezioni del PDF consultate; articolo già alla base del brainstorming.

### R02 — Watawana et al. (2026): il seguito diretto da aggiungere alla proposta

**Hasindri Watawana, Sergio Burdisso, Diego A. Moreno-Galván, Fernando Sánchez-Vega, A. Pastor López-Monroy, Petr Motlicek ed Esaú Villatoro-Tello.** *When Consistency Becomes Bias: Interviewer Effects in Semi-Structured Clinical Interviews*. LREC 2026, pp. 2355–2361. DOI: `10.63317/34hw23mzd8c7`.

[Versione pubblicata](https://aclanthology.org/2026.lrec-1.185/) · [Testo consultato](https://arxiv.org/html/2603.24651v1) · [Codice](https://github.com/idiap/bias_in_daic-woz/tree/main/LREC_2026)

**Contributo:** estende l'audit a DAIC-WOZ, E-DAIC e ANDROIDS con GCN e Longformer, localizzando evidenze per parlante e posizione temporale. Per E-DAIC ricostruisce trascrizioni dei due parlanti mediante ASR.

**Risultato da leggere con attenzione:** il vantaggio interviewer-only dipende da modello e split; sul test DAIC-WOZ il Longformer participant-only supera quello interviewer-only. Non bisogna trasformare il messaggio generale del paper in una regola universale.

**Per la tesi:** serve a scegliere controlli per parlante, posizione nell'intervista, architettura e split, evitando di attribuire ogni effetto a un'unica domanda o di presumere che *interviewer-only* vinca sempre. Impone di non vendere il semplice passaggio da DAIC-WOZ a E-DAIC come novità: E6 dovrà usare sessioni disgiunte e verificare differenze di trascrizione/ASR. Per distinguersi, la tesi deve chiedere se il LLM sul dialogo completo usa quegli indizi (E3), quali siti interni mediano l'effetto (E4) e se il lessico migliora la scelta degli interventi (E5). I risultati di GCN e Longformer costituiscono precedenti, non esiti garantiti per un LLM generativo.

**Verifica:** pubblicazione confermata e sezioni di metodi/risultati del preprint consultate.

### R03 — Ahsan et al. (2025): il precedente meccanicistico più vicino

**Hiba Ahsan, Arnab Sen Sharma, Silvio Amir, David Bau e Byron C. Wallace.** *Elucidating Mechanisms of Demographic Bias in LLMs for Healthcare*. Findings of EMNLP 2025, pp. 14614–14631. DOI: `10.18653/v1/2025.findings-emnlp.789`.

[Articolo](https://aclanthology.org/2025.findings-emnlp.789/) · [PDF](https://aclanthology.org/2025.findings-emnlp.789.pdf) · [Codice](https://github.com/hibaahsan/interp-healthcare-bias/)

**Contributo:** localizza e manipola rappresentazioni demografiche tramite patching, studiandone l'effetto su generazione clinica e predizioni. Include una valutazione del rischio depressivo su testi clinici derivati da MIMIC.

**Per la tesi:** è il precedente da usare per definire un intervento interno collegato a un cambiamento della decisione, anziché fermarsi a un probe che decodifica una categoria. In E4 si possono riprendere il confronto tra attivazioni di coppie controllate e la misura dell'effetto sull'output, adattandoli a domande e risposte di colloqui lunghi. La verifica deve includere siti di controllo, patching inverso e replica su partecipanti non usati per selezionare i siti. Il loro oggetto è il bias demografico in testi sanitari diversi: non offre già una localizzazione degli indizi del protocollo di Ellie.

**Limite:** modificare una predizione sul rischio non prova accuratezza diagnostica né causalità clinica.

**Verifica:** metadati e sezioni del PDF finale consultati. Per una replica usare la versione pubblicata, più ampia del primo preprint.

### R04 — Lee, Han e Woo (2026): LLM e fattori interpretabili sugli stessi corpus

**Jae-Joong Lee, Jihoon Han e Choong-Wan Woo.** *Interpretable depression assessment using a large language model*. PLOS Digital Health, 5(2), e0001205. DOI: `10.1371/journal.pdig.0001205`.

[Articolo integrale](https://pmc.ncbi.nlm.nih.gov/articles/PMC12885269/) · [Risorse AIDA](https://github.com/cocoanlab/AIDA)

**Contributo:** usa un LLM per estrarre fattori legati alla depressione e una regressione lineare per stimare il PHQ-8; valuta il sistema su DAIC-WOZ e su un campione E-DAIC.

**Per la tesi:** offre una baseline a fattori leggibili e un elenco di evidenze clinicamente organizzate con cui confrontare il lessico proposto. Aiuta a distinguere due domande: «si può ottenere una stima del PHQ-8 da fattori estratti?» e «quali informazioni e componenti usa internamente il LLM studiato?». La seconda richiede perturbazioni e interventi sul medesimo predittore, anche quando i fattori prodotti appaiono ragionevoli. Per un confronto quantitativo servono target, metriche, subset e versione delle trascrizioni omogenei; gli errori di regressione del paper non vanno accostati direttamente alla F1 binaria della tesi.

**Dettaglio sperimentale:** le trascrizioni E-DAIC sono state rifatte e revisionate. I risultati non sono direttamente confrontabili usando una diversa versione dei testi.

**Verifica:** abstract, metodi e discussione del testo integrale consultati. Evitare di confrontare direttamente i loro errori di regressione con F1 di classificazione.

### R05 — Low et al. (2026): costruzione del lessico con LLM

**Daniel M. Low, Osiris Rankin, Daniel D. L. Coppersmith, Kate H. Bentley, Matthew K. Nock e Satrajit S. Ghosh.** *Using large language models to create lexicons for interpretable text models with high content validity: The suicide risk lexicon*. Journal of Psychopathology and Clinical Science, pubblicazione online anticipata, 24 settembre 2026. DOI: `10.1037/abn0001152`.

[Record e abstract](https://pubmed.ncbi.nlm.nih.gov/42782714/) · [DOI](https://doi.org/10.1037/abn0001152) · [Construct-tracker](https://github.com/danielmlow/construct-tracker)

**Contributo:** propone lessici specialistici assistiti da LLM e validazione delle categorie da parte di clinici, applicati al rischio suicidario.

**Per la tesi:** suggerisce di partire da costrutti espliciti, generare espressioni candidate con un LLM, registrare prompt e versioni, poi sottoporre categorie e passaggi a revisione umana. La tesi può trasferire questo processo a sintomi collegati agli otto domini PHQ-8, storia clinica e indizi dell'intervistatore, misurando accordo e precisione del riconoscimento contestuale. L'uso nuovo da verificare è **esterno al predittore**: selezionare span e coppie per E3–E4 e confrontare la resa con selezione casuale abbinata e attribuzioni generiche in E5. Il loro lessico del rischio suicidario e la sua validità di contenuto non convalidano automaticamente un lessico per la depressione.

**Limite di trasferimento:** un lessico di rischio suicidario non è automaticamente valido per PHQ-8 e interviste sulla depressione. La generazione dell'LLM non sostituisce la revisione del costrutto e del contesto.

**Verifica:** abstract/record e documentazione degli autori; in questa ricerca non è stato verificato integralmente il PDF editoriale.

### R06 — Milintsevich, Dias e Sirts (2024): lessici dentro il modello

**Kirill Milintsevich, Gaël Dias e Kairit Sirts.** *Evaluating Lexicon Incorporation for Depression Symptom Estimation*. ClinicalNLP 2024, pp. 322–328. DOI: `10.18653/v1/2024.clinicalnlp-1.28`.

[Articolo](https://aclanthology.org/2024.clinicalnlp-1.28/) · [PDF](https://aclanthology.org/2024.clinicalnlp-1.28.pdf)

**Contributo:** marca nell'input parole identificate da AFINN, NRC e SDD, usando BERT/MentalBERT per la stima dei sintomi. Considera DAIC-WOZ e PRIMATE; i benefici dipendono da lessico e task.

**Per la tesi:** fornisce una baseline e un controllo metodologico per E5: verificare che eventuali benefici non derivino semplicemente dall'aver selezionato o marcato un certo numero di parole. Il loro lessico entra nell'input di BERT/MentalBERT; nel protocollo principale della tesi il predittore riceve il dialogo invariato e il lessico serve al ricercatore per scegliere interventi. Il confronto corretto è quindi sulla qualità degli span e sugli effetti trovati **a parità di budget**, non una promessa che il lessico aumenti la F1 del LLM. Le marcature casuali del paper motivano un controllo casuale abbinato.

**Controllo da riprendere:** confronta anche marcature casuali, che possono produrre benefici. L'effetto di una manipolazione non dimostra automaticamente il valore semantico del lessico.

**Verifica:** metodi, lessici, risultati e controlli nel PDF consultati.

### R07 — Villatoro-Tello et al. (2021): lessico interpretabile su DAIC-WOZ/E-DAIC

**Esaú Villatoro-Tello, Gabriela Ramírez-de-la-Rosa, Daniel Gatica-Perez, Mathew Magimai-Doss e Héctor Jiménez-Salazar.** *Approximating the Mental Lexicon from Clinical Interviews as a Support Tool for Depression Detection*. ICMI 2021. DOI: `10.1145/3462244.3479896`.

[Record istituzionale](https://publications.idiap.ch/publications/show/4652) · [PDF degli autori](https://publications.idiap.ch/attachments/papers/2021/VILLATORO-TELLO_ICMI2021-2_2021.pdf)

**Contributo:** approssima il lessico mentale tramite la teoria della disponibilità lessicale, usa il vocabolario rilevante come feature e valuta classificazione e interpretabilità nei due corpus.

**Per la tesi:** è un confronto necessario per la componente lessicale: esiste già un vocabolario rilevante ricavato da interviste DAIC-WOZ/E-DAIC e usato come feature di classificazione. Se disponibile in forma riutilizzabile, confrontarne copertura e precisione con il lessico costruito con LLM sugli stessi passaggi annotati; in caso contrario, confrontare almeno definizioni e tipi di evidenza. La tesi deve mostrare un'utilità ulteriore per la **scelta di perturbazioni e siti interni**, non rivendicare come nuovo il semplice fatto di avere un lessico interpretabile per questi corpus. Le loro prestazioni restano un riferimento solo con split e task compatibili.

**Differenza:** non studia componenti interne di un LLM generativo e non costruisce il lessico attraverso il processo proposto da Low.

**Verifica:** metadati istituzionali, abstract e apertura del PDF; lettura integrale ancora da svolgere.

### R08 — Zhang e Poellabauer (2025): mitigare il bias mantenendo il contesto

**Enshi Zhang e Christian Poellabauer.** *Mitigating Interviewer Bias in Multimodal Depression Detection: An Approach with Adversarial Learning and Contextual Positional Encoding*. Findings of EMNLP 2025, pp. 12169–12188. DOI: `10.18653/v1/2025.findings-emnlp.650`.

[Articolo](https://aclanthology.org/2025.findings-emnlp.650/) · [PDF](https://aclanthology.org/2025.findings-emnlp.650.pdf)

**Contributo:** propone un transformer multimodale del dialogo con informazioni contestuali sulle domande e un classificatore adversarial con gradient reversal, per ridurre la dipendenza dalla tipologia delle domande.

**Per la tesi:** mostra perché togliere indiscriminatamente le domande non è un buon test: esse possono dare alla risposta un referente necessario, per esempio quando il partecipante dice solo «yes». Questo orienta E3 verso parafrasi e neutralizzazioni controllate con verifica di coerenza. Se E3–E4 individuano una dipendenza indesiderata, il paper offre una mitigazione adversarial da discutere o confrontare nell'estensione E7, misurando sia la riduzione della dipendenza sia la prestazione predittiva. L'addestramento di un modello meno dipendente dalle domande non sostituisce l'audit dei meccanismi interni di uno specifico LLM.

**Differenza:** il nucleo è l'addestramento di rappresentazioni meno dipendenti dalle domande, non la verifica meccanicistica di un LLM generativo. Una riduzione dell'informazione decodificabile sul tipo di domanda non equivale da sola a dimostrare l'eliminazione di tutti gli shortcut.

**Verifica:** metadati, abstract e sezioni delle ablation consultati.

## 5. LLM e spiegazioni nella depressione: confronti vicini ma distinti

### R09 — Zhang et al. (2025): RED e recupero di evidenze

**Linhai Zhang, Ziyang Gao, Deyu Zhou e Yulan He.** *Explainable Depression Detection in Clinical Interviews with Personalized Retrieval-Augmented Generation*. Findings of ACL 2025, pp. 9927–9944. DOI: `10.18653/v1/2025.findings-acl.517`. Versione preliminare arXiv:2503.01315.

[Versione pubblicata](https://aclanthology.org/2025.findings-acl.517/) · [Preprint consultato](https://arxiv.org/html/2503.01315v1)

**Contributo:** RED recupera evidenze dalle interviste attraverso query personalizzate e conoscenza di supporto, per produrre predizioni spiegabili su DAIC-WOZ.

**Per la tesi:** offre un confronto per l'identificazione di passaggi testuali che sembrano sostenere una predizione. Nel progetto principale, questi passaggi possono servire come selezione alternativa agli span del lessico in E5, se recuperati su dati e budget comparabili. Nel piano di riserva, RED motiva la domanda più precisa: il passaggio citato cambia davvero lo score quando viene modificato, e un intervento interno conferma il suo contributo? Il recupero di evidenze rende una spiegazione leggibile, ma non stabilisce da solo la fedeltà causale della spiegazione per il LLM studiato.

**Attenzione al protocollo:** alcune ablation sono condotte sull'insieme combinato train+development. Non equiparare quelle metriche a un risultato sul test ufficiale; ciò non implica automaticamente leakage, ma richiede di confrontare protocolli omogenei.

**Verifica:** pubblicazione in Findings of ACL 2025 confermata su ACL Anthology; il dettaglio delle ablation qui annotato proviene dal preprint consultato e va ricontrollato nella versione pubblicata prima di una replica.

### R10 — Moon et al. (2025): DepressLLM

**Sehwan Moon et al.** *DepressLLM: Interpretable domain-adapted language model for depression detection from real-world narratives*. Preprint arXiv:2508.08591.

[Record](https://arxiv.org/abs/2508.08591) · [Testo](https://arxiv.org/html/2508.08591v1)

**Contributo:** addestra e valuta modelli su narrazioni autobiografiche, propone una stima basata sulle probabilità dei token di punteggio e usa anche DAIC-WOZ come valutazione esterna. Include revisione psichiatrica di errori ad alta confidenza.

**Per la tesi:** suggerisce di conservare uno **score continuo**, per esempio da log-probabilità dei token di risposta, così le perturbazioni possono essere misurate anche quando la classe finale resta uguale. La revisione di errori ad alta confidenza offre un modello per scegliere casi qualitativi da discutere senza trasformarli in evidenza statistica generale. L'uso di DAIC-WOZ come valutazione esterna motiva controlli sulle differenze tra corpora e task; il paper non fornisce già i siti interni responsabili di una decisione nelle interviste.

**Attenzione:** verificare PHQ-9 contro PHQ-8, soglie e copertura dei campioni quando si confrontano prestazioni o risultati filtrati per confidenza.

**Verifica:** abstract e sezioni del testo sui dati. Autori verificati sul record primario: alcune citazioni secondarie riportano nomi differenti.

### R11 — Lyu et al. (2026): Dep-LLM e ragionamento multifattoriale

**Yiqing Lyu, Xianbing Zhao, Buzhou Tang e Ronghuan Jiang.** *Dep-LLM: Training-Free Depression Diagnosis via Evidence-Guided Structured Multi-factor with Reliable LLM Reasoning*. Preprint arXiv:2606.10796, giugno 2026.

[Record e PDF](https://arxiv.org/abs/2606.10796)

**Contributo:** struttura l'analisi delle interviste in fattori, genera razionali e combina segnali modulati da una misura di confidenza; riporta esperimenti su DAIC-WOZ ed E-DAIC senza fine-tuning aggiuntivo.

**Per la tesi:** propone un'organizzazione dei segnali in fattori e una procedura senza fine-tuning che può informare il pilot dei prompt e una baseline di spiegazioni strutturate. Per il progetto principale, si può verificare se fattori nominati in una spiegazione coincidono con le categorie di domande o risposte che hanno effetto nelle perturbazioni. Per il piano di riserva, il confronto diventa tra fattori citati dal LLM e passaggi del partecipante causalmente influenti. Una catena di ragionamento convincente non dimostra, senza interventi, che quei fattori abbiano guidato la predizione.

**Limite:** un razionale strutturato e una misura di incertezza non dimostrano fedeltà causale delle spiegazioni. Il termine «diagnosis» appartiene al titolo, non è una conclusione di validazione clinica adottata da questa bibliografia.

**Verifica:** abstract e metadati; dettagli di split e implementazione da approfondire.

### R12 — Ling e Chorney (2026): fattibilità dei modelli locali

**Sing Hui Ling e Wesley Chorney.** *The use of large language models in automated depression detection*. Acta Psychologica, 269, 107602, settembre 2026. DOI: `10.1016/j.actpsy.2026.107602`.

[Articolo dell'editore](https://www.sciencedirect.com/science/article/pii/S0001691826014034)

**Contributo:** valuta modelli eseguibili localmente, inferiori a 15 miliardi di parametri, su DAIC-WOZ; riporta prestazioni variabili e scarsa concordanza tra modelli nelle configurazioni esaminate.

**Per la tesi:** è un vincolo di fattibilità per E1: prima di spendere risorse sul patching, verificare se un LLM locale produce score ripetibili e prestazioni superiori a baseline banali sullo split disponibile. La scarsa concordanza riportata tra modelli suggerisce di dichiarare checkpoint, prompt e regola di aggregazione, senza generalizzare il risultato di un singolo modello a tutti gli LLM. Se il pilot fallisce, il piano di riserva sulle evidenze del partecipante non è automaticamente praticabile con lo stesso predittore: occorre prima risolvere il problema del modello o del task.

**Limite:** non dimostra che ogni piccolo LLM, qualunque fine-tuning o ogni futuro sistema siano inadeguati. È un risultato dipendente dal protocollo sperimentale.

**Verifica:** pagina editoriale e abstract; dettagli dei prompt e dei modelli da leggere prima di scegliere una baseline.

## 6. Precedenti meccanicistici e metodi da trasferire

### R13 — Wu, Wu e Sun (2024): dizionari di feature cliniche interne

**John Wu, David Wu e Jimeng Sun.** *Beyond Label Attention: Transparency in Language Models for Automated Medical Coding via Dictionary Learning*. EMNLP 2024, pp. 8848–8871. DOI: `10.18653/v1/2024.emnlp-main.500`.

[Articolo](https://aclanthology.org/2024.emnlp-main.500/) · [PDF alternativo](https://aclanthology.org/anthology-files/pdf/emnlp/2024.emnlp-main.500.pdf)

**Contributo:** usa dictionary learning e rappresentazioni sparse per spiegare predizioni di codici ICD e intervenire sul comportamento del modello; l'implementazione descritta usa attivazioni di un encoder clinico RoBERTa.

**Per la tesi:** mostra un possibile livello di analisi più fine del residual stream: ottenere feature interne interpretabili e intervenire su di esse per misurarne il contributo a una decisione. Se si dispone di un dizionario o SAE compatibile con il checkpoint scelto, si può verificare se feature candidate rispondono a categorie del lessico e se l'intervento cambia lo score. È un'estensione, non un prerequisito: il lessico di frasi e le feature latenti sono oggetti diversi e non vanno equiparati per somiglianza dei nomi.

**Distinzione essenziale:** un dizionario di feature latenti ottenuto con SAE non è un lessico di parole e frasi generato da LLM. Il collegamento tra i due sarebbe un esperimento, non un'equivalenza.

**Verifica:** abstract e sezioni del PDF indicizzate; utile soprattutto se si aggiunge l'estensione SAE.

### R14 — Yeo, Satapathy e Cambria (2025): fedeltà delle spiegazioni tramite patching

**Wei Jie Yeo, Ranjan Satapathy ed Erik Cambria.** *Towards Faithful Natural Language Explanations: A Study Using Activation Patching in Large Language Models*. EMNLP 2025, pp. 10425–10447. DOI: `10.18653/v1/2025.emnlp-main.529`. Preprint iniziale del 2024.

[Articolo](https://aclanthology.org/2025.emnlp-main.529/) · [PDF](https://aclanthology.org/2025.emnlp-main.529.pdf) · [Codice indicato nella versione pubblicata](https://github.com/SenticNet/causal-faithfulness)

**Contributo:** propone una misura di *Causal Faithfulness* basata sulla corrispondenza delle attribuzioni causali per risposta e spiegazione, ottenute tramite activation patching.

**Per la tesi:** fornisce il precedente metodologico più diretto per il piano di riserva: confrontare l'evidenza che il LLM cita nella spiegazione con l'evidenza che influisce sulla sua risposta sotto intervento. Nel progetto principale può diventare una verifica facoltativa delle spiegazioni verbali su Ellie; nel piano di riserva è il cuore della domanda sulla fedeltà dei passaggi del partecipante. Riprendere la logica di confronto tra attribuzioni causali, ma costruire controlli specifici per negazione, temporalità e attribuzione a sé o ad altri nelle interviste. La misura proposta dal paper non rende automaticamente valide coppie di input mal costruite.

**Limite:** la misura non è una certificazione universale di fedeltà e non elimina la necessità di controllare gli input controfattuali. Nel nucleo minimo della tesi non è indispensabile generare spiegazioni verbali.

**Verifica:** abstract e sezione metodologica del PDF.

### R15 — Zhang e Nanda (2024): progettare bene il patching

**Fred Zhang e Neel Nanda.** *Towards Best Practices of Activation Patching in Language Models: Metrics and Methods*. ICLR 2024. Preprint arXiv:2309.16042 del 2023.

[Atti ICLR](https://proceedings.iclr.cc/paper_files/paper/2024/hash/06a52a54c8ee03cd86771136bc91eb1f-Abstract-Conference.html) · [Preprint e PDF](https://arxiv.org/abs/2309.16042)

**Contributo:** mostra che scelte di metrica e corruzione degli input possono cambiare i risultati di localizzazione.

**Per la tesi:** guida una scelta da fissare **prima** del test finale: quale score usare (per esempio differenza di log-probabilità delle classi), come costruire la variante e quale attivazione sostituire. Poiché metrica e corruzione possono cambiare la localizzazione, E4 deve riportare effetti non normalizzati, confronti con siti di controllo e sensibilità a più formulazioni delle domande. Nel piano di riserva lo stesso principio vale per i passaggi del partecipante: l'effetto di una frase eliminata brutalmente può riflettere un input innaturale, non il concetto clinico cercato.

**Trasferimento proposto:** confrontare più perturbazioni semanticamente controllate degli indizi di Ellie, verificando che la conclusione non dipenda da una sola formulazione.

**Verifica:** metadati degli atti, abstract e indicazioni metodologiche già consultate nel brainstorming.

### R16 — Heimersheim e Nanda (2024): interpretare correttamente gli interventi

**Stefan Heimersheim e Neel Nanda.** *How to use and interpret activation patching*. Preprint arXiv:2404.15255.

[Record e testo](https://arxiv.org/abs/2404.15255)

**Contributo:** presenta indicazioni pratiche su varianti del patching, metriche e significato dell'evidenza ottenuta sui circuiti.

**Per la tesi:** serve per progettare e descrivere correttamente E4: eseguire patching originale→variante e variante→originale, riportare l'effetto sullo score e includere controlli di pari dimensione. Aiuta a formulare conclusioni locali — un sito contribuisce alla differenza tra coppie testate — senza chiamarlo circuito generale della depressione. Se non c'è differenza comportamentale tra le coppie, non va interpretato come «patching negativo»: manca l'effetto che si vorrebbe mediare. Questi criteri valgono anche per il piano di riserva.

**Trasferimento proposto:** verificare entrambe le direzioni degli interventi, usare controlli e limitare le conclusioni alle condizioni testate. Il recupero dello score non identifica necessariamente l'unico percorso interno disponibile.

**Verifica:** record e testo HTML consultato durante la preparazione della proposta; trattato come guida metodologica in forma di preprint.

### R17 — Geiger et al. (2024): dai concetti ai sottospazi interni

**Atticus Geiger, Zhengxuan Wu, Christopher Potts, Thomas Icard e Noah D. Goodman.** *Finding Alignments Between Interpretable Causal Variables and Distributed Neural Representations*. CLeaR 2024, PMLR 236, pp. 160–187. Preprint iniziale del 2023.

[Articolo e PDF negli atti](https://proceedings.mlr.press/v236/geiger24a.html)

**Contributo:** introduce Distributed Alignment Search, cercando allineamenti tra variabili interpretabili e rappresentazioni distribuite attraverso interventi, senza richiedere che ogni concetto coincida con un insieme disgiunto di neuroni.

**Per la tesi:** chiarisce che il passaggio da una categoria umana, come «storia di trattamento», a una rappresentazione interna richiede un'ipotesi causale e interventi coerenti, non una semplice correlazione tra parole e neuroni. Un'estensione potrebbe definire variabili contrafattuali esplicite, cercare un sottospazio che le codifichi e verificare se scambiarne il valore cambia la predizione come previsto. Per il nucleo magistrale questo sarebbe più impegnativo dell'activation patching su pochi siti; il paper aiuta soprattutto a delimitare ciò che E4 **non** dimostra.

**Limite:** richiede una specifica delle variabili e dei comportamenti controfattuali attesi. La presenza di una parola in una categoria lessicale non basta a definire una buona astrazione causale. È un'estensione più impegnativa del patching su siti selezionati.

**Verifica:** metadati e abstract degli atti; dettagli implementativi da approfondire.

### R18 — Marks et al. (2025): circuiti interpretabili e feature spurie

**Samuel Marks, Can Rager, Eric J. Michaud, Yonatan Belinkov, David Bau e Aaron Mueller.** *Sparse Feature Circuits: Discovering and Editing Interpretable Causal Graphs in Language Models*. ICLR 2025.

[Atti ICLR](https://proceedings.iclr.cc/paper_files/paper/2025/hash/3ba4d47a83e498c2b1a0868cba20f6de-Abstract-Conference.html) · [PDF degli autori](https://belinkov.com/assets/pdf/iclr2025-sfc.pdf)

**Contributo:** identifica circuiti di feature interpretabili e presenta SHIFT, che interviene su feature giudicate irrilevanti per migliorare la generalizzazione di un classificatore.

**Per la tesi:** offre un modello per un'estensione E7: dopo aver individuato componenti causalmente rilevanti per un indizio di protocollo, attenuarle e misurare sia la riduzione della dipendenza sia la tenuta della prestazione su dati disgiunti. Il paper mostra anche perché «feature interpretabile» e «feature spuria» non sono sinonimi: occorre definire perché l'indizio sia inappropriato per il target. La scoperta di grafi di feature o l'uso di SAE compatibili richiederebbero risorse aggiuntive e non sono necessari per E3–E5.

**Limite:** non studia DAIC-WOZ né dimostra che una feature clinicamente nominabile sia uno shortcut. L'applicazione richiederebbe risorse SAE compatibili e verifica della fedeltà della decomposizione.

**Verifica:** abstract degli atti e apertura del PDF. Da leggere dopo aver stabilizzato il nucleo del progetto, non come prerequisito obbligatorio.

## 7. Due lavori recenti da tenere nel perimetro

### R19 — Gao et al. (2026): PADE

**Ziyang Gao, Linhai Zhang, Yulan He e Deyu Zhou.** *From Personal to Clinical: Personalisation and Depersonalisation for Explainable Depression Detection*. DASFAA 2026, LNCS 16537, pp. 133–148. DOI: `10.1007/978-981-92-0369-7_9`.

[Pagina editoriale](https://doi.org/10.1007/978-981-92-0369-7_9)

PADE combina query personalizzate, recupero di evidenze e trasformazione in indizi clinicamente interpretabili. È vicino alla parte di organizzazione semantica dell'evidenza, ma l'abstract non documenta activation patching. La pagina consultata offre un'anteprima: verificati metadati e abstract, non il metodo integrale. Non assumere che sia semplicemente la versione pubblicata di RED: sono riferimenti distinti finché non se ne confrontano i testi.

**Per la tesi:** suggerisce un confronto qualitativo fra categorie clinicamente nominate e passaggi effettivamente citati nelle spiegazioni, utile soprattutto per il piano di riserva. Nel progetto principale può orientare la descrizione delle evidenze del partecipante, senza sostituire i controlli sugli indizi dell'intervistatore. Prima di usarlo come baseline quantitativa occorre leggere il metodo integrale e verificare dati, split e metrica: dall'abstract non si può dedurre che i passaggi recuperati siano causalmente fedeli al predittore.

### R20 — Keeman (2026): controlli senza parole-emozione

**Michael Keeman.** *AIPsy-Affect: A Keyword-Free Clinical Stimulus Battery for Mechanistic Interpretability of Emotion in Language Models*. Preprint arXiv:2604.23719.

[Record e PDF](https://arxiv.org/abs/2604.23719)

Propone stimoli narrativi emotivi senza parole esplicite dell'emozione e controlli neutrali abbinati, destinati ad analisi delle rappresentazioni e a interventi. L'idea utile è distinguere sensibilità alla parola da sensibilità al concetto. Non è una validazione del lessico depressivo né un risultato su DAIC-WOZ. Verificati abstract e metadati; l'effettiva qualità degli stimoli va valutata prima di adottarli.

**Per la tesi:** ispira un controllo semantico per E5: includere parafrasi o passaggi che esprimono un costrutto senza una parola chiave ovvia, oltre a falsi positivi lessicali con negazione o riferimento al passato. Se il matcher riconosce solo la keyword ma il LLM reagisce al concetto, il lessico va migliorato o la conclusione sulla sua utilità ridimensionata. È uno spunto per costruire stimoli controllati, non una prova che quelle frasi siano clinicamente valide nel corpus DAIC-WOZ.

## 8. Che cosa cambierei nel posizionamento della tesi

### Rivendicazioni da evitare

| Rivendicazione troppo ampia | Precedenti da discutere |
|---|---|
| «Prima analisi degli shortcut in DAIC-WOZ/E-DAIC» | R01, R02 |
| «Primo uso di lessici per la depressione nelle interviste» | R06, R07 |
| «Primo uso di LLM interpretabili su questi dataset» | R04, R09–R11 |
| «Prima interpretabilità meccanicistica in sanità o sul rischio depressivo» | R03, R13 |
| «Primo uso di interventi interni per limitare feature spurie» | R18 |

### Contributo candidato, da sottoporre al relatore

**Valutare se un lessico assistito da LLM e revisionato consente di identificare e testare più efficacemente le componenti interne che mediano la dipendenza di un LLM dagli indizi del protocollo d'intervista, distinguendoli dalle evidenze del partecipante.**

Questa formulazione rende centrale una domanda verificabile: il lessico aiuta davvero a produrre spiegazioni causali più stabili o efficienti rispetto a selezione casuale abbinata e attribuzioni non guidate dal lessico?

Il contributo non dovrebbe ridursi a concatenare strumenti. Servono almeno:

1. **Un oggetto causale preciso:** manipolazioni degli indizi del protocollo che conservino, per quanto possibile, il contenuto informativo del partecipante e la coerenza del dialogo.
2. **Una separazione tra scoperta e conferma:** lessico, siti e prompt scelti senza utilizzare gli esempi di conferma finale.
3. **Un confronto sul valore aggiunto del lessico:** stesso budget di span, interventi e componenti per i metodi confrontati.
4. **Una verifica semantica:** parafrasi senza la keyword, negazioni e cambiamenti del parlante, per distinguere una categoria concettuale da una semplice corrispondenza lessicale.
5. **Una replica esplicitamente delimitata:** su altri partecipanti e, dove i dati lo consentono, sulle sessioni autonome E-DAIC disgiunte; eventuali differenze di ASR vanno trattate come possibili confondenti.

Un titolo più focalizzato, ancora neutro sui risultati, potrebbe essere:

> **Audit meccanicistico guidato da lessici della dipendenza dal protocollo d'intervista negli LLM per la predizione della depressione.**

È una proposta di posizionamento coerente con il [piano della tesi]({{ '/' | relative_url }}); il titolo finale dipenderà dagli esperimenti.

## 9. Ordine di lettura e utilizzo concreto

| Ordine | Lavori | Decisione che aiutano a prendere |
|---|---|---|
| 1 | R01 + R02 | Quali effetti sono già noti e quali protocolli/split replicare |
| 2 | R03 | Come impostare interventi interni in un task clinico vicino |
| 3 | R05 + R06 | Come costruire il lessico e quali controlli servono per valutarlo |
| 4 | R04 | Quale baseline a fattori interpretabili confrontare |
| 5 | R15 + R16 | Come progettare metrica, coppie di input e patching |
| 6 | R08 + R12 | Se puntare solo sull'audit o anche su mitigazione; sostenibilità del modello |
| 7 | R14, poi R13/R17/R18 | Quale eventuale estensione scegliere: fedeltà verbale, SAE o allineamento a concetti |

Prima di fissare il protocollo definitivo restano da approfondire: dettagli delle release e degli split dei lavori principali; disponibilità del codice e delle trascrizioni ricostruite; risorse hardware effettive; accesso a revisori clinici; ricerca delle citazioni successive dei precedenti più vicini. La bibliografia sostiene una proposta circoscritta e plausibile, ma non certifica ancora la sua novità assoluta.
