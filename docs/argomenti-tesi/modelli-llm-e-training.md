# Modelli LLM e strategie di training per la tesi su DAIC-WOZ

*Ricognizione del 5 ottobre 2026. Questo documento accompagna [il piano di tesi](brainstorming00.md); è una selezione per un pilot, non il risultato di benchmark eseguiti su DAIC-WOZ. Dimensioni, licenze e disponibilità dei checkpoint vanno ricontrollate prima degli esperimenti.*

## 1. Scelta in breve

**Proposta principale:** usare [Qwen3-4B-Instruct-2507](https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507) come predittore testuale, valutarlo prima senza adattamento e, se non produce uno score abbastanza utile e stabile, addestrare un adapter **QLoRA** sul solo training set. Usare [SmolLM2-1.7B-Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct) per collaudare prompt, tokenizzazione, interventi e memoria. Un eventuale secondo modello di altra famiglia è [Mistral-7B-Instruct-v0.3](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3), da aggiungere solo dopo aver chiuso il protocollo sul modello principale.

Il modello riceve **il dialogo**, con i ruoli dei parlanti. Il lessico serve a selezionare span e interventi per l'audit, e può alimentare una baseline distinta; non è una feature aggiuntiva del predittore principale. Il target operativo resta l'etichetta binaria ufficiale collegata al PHQ-8, non una diagnosi clinica indipendente. Vedi [piano, sezione 3](brainstorming00.md#3-dataset-e-task).

## 2. Criteri di scelta

1. **Accesso alle attivazioni.** Servono pesi e implementazione che consentano forward pass, hook e activation patching. Una API remota o una quantizzazione per sola inferenza non basta.
2. **Task e formato.** Il modello deve produrre uno score ripetibile tra due risposte di classe predefinite; la qualità delle spiegazioni generate non è la misura principale.
3. **Costo reale.** La lunghezza della sequenza, le attivazioni salvate e i passaggi ripetuti del patching incidono sulla memoria almeno quanto il numero di parametri. La lunghezza massima pubblicizzata dal modello non equivale alla lunghezza sostenibile su Colab.
4. **Confronto scientifico.** Tenere fissi prompt, finestre, verbalizzazioni delle classi, precisione numerica e split nei confronti prima/dopo LoRA. Se si usa QLoRA, il controllo iniziale va caricato con **la stessa quantizzazione** del modello adattato.
5. **Trasferibilità del dominio.** Un modello addestrato su post social, frasi isolate o risposte terapeutiche non è automaticamente adatto a colloqui DAIC-WOZ di 7–33 minuti. Il corpus ufficiale contiene 189 sessioni: il limite statistico è severo anche se si ricavano molte finestre di testo. [Fonte corpus](https://dcapswoz.ict.usc.edu/).

## 3. Modelli generali candidati

| Modello | Ruolo suggerito | Vantaggi per questa tesi | Limite pratico | Training da provare |
|---|---|---|---|---|
| [SmolLM2-1.7B-Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct) | Pilot e possibile baseline LLM piccola | 1,7B; pesi aperti, licenza Apache 2.0; più agevole per verificare hook, tokenizzazione e patching | La capacità predittiva va misurata; non presumere che sia sufficiente per E1 | Prima prompt fisso; poi LoRA se E1 è debole |
| [Qwen3-4B-Instruct-2507](https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507) | **Modello principale proposto** | 4B, Apache 2.0, modalità non-thinking: adatto a uno score di classe semplice e a un audit ripetibile | Il contesto nominale di 262.144 token non implica che training o patching di un'intera intervista siano fattibili sulla GPU disponibile | Prompt fisso; poi QLoRA 4-bit su T4, LoRA/QLoRA da verificare su L4/A100 |
| [Mistral-7B-Instruct-v0.3](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3) | Replica su una famiglia diversa | 7B, Apache 2.0; confronto più indipendente che usare un altro Qwen | Su T4 il training e soprattutto il patching di finestre lunghe sono impegnativi | QLoRA con finestre corte; LoRA solo dopo prova di memoria su GPU più ampia |
| [Qwen2.5-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct) | Alternativa al secondo modello | 7,61B, Apache 2.0; altra scelta se Mistral si comporta male sul task | Aumenta il costo senza aggiungere una famiglia architetturale molto distante dal Qwen principale | QLoRA; non è necessario studiarlo insieme a Mistral |

Le informazioni sui contesti nominali vanno lette insieme ai file di configurazione: [SmolLM2](https://huggingface.co/HuggingFaceTB/SmolLM2-1.7B-Instruct/blob/main/config.json), [Qwen3-4B](https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507/blob/main/config.json), [Mistral-7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3/blob/main/config.json), [Qwen2.5-7B](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct/blob/main/config.json). Sono limiti del modello, non preventivi di memoria.

## 4. Modelli già adattati alla salute mentale: quanto sono utili?

**Sì, esistono checkpoint sotto 8B, ma non ne ho identificato uno che sia contemporaneamente un predittore testuale PHQ-8 di DAIC-WOZ, ben documentato, facilmente riproducibile e adatto come scelta automatica per l'audit meccanicistico.** I più interessanti restano candidati *secondari* da misurare sullo stesso protocollo.

| Checkpoint | Adattamento dichiarato | Cosa può dare alla tesi | Perché non assumerlo come modello principale |
|---|---|---|---|
| [MentaLLaMA-chat-7B-hf](https://huggingface.co/klyang/MentaLLaMA-chat-7B-hf), con [paper del progetto](https://arxiv.org/abs/2309.13567) | LLaMA 2-chat 7B adattato per analisi di salute mentale con istruzioni IMHI, soprattutto **testi social** | È il confronto di dominio più motivato scientificamente: si può chiedere se una specializzazione pregressa cambi prestazioni e sensibilità agli indizi di Ellie | Target, stile e lunghezza dei testi sono diversi da DAIC-WOZ; il contesto configurato è 4.096 token. Le spiegazioni che genera non sono prova di fedeltà causale. La scheda indica MIT, ma il modello deriva da LLaMA 2: verificare anche le condizioni del modello base prima di ridistribuire pesi o adapter. |
| [dsuram/mistral-mental-health-lora](https://huggingface.co/dsuram/mistral-mental-health-lora) | Adapter LoRA su Mistral-7B-Instruct-v0.1 per classificare **frasi** in sette categorie, tra cui depressione | Mostra un esempio vicino al formato di classificazione e può essere un controllo esplorativo | La categoria «Depression» non coincide con l'etichetta PHQ-8; la scheda non documenta abbastanza la provenienza dei dati e la separazione dei partecipanti per trasferire le metriche dichiarate a questa tesi. Verificare prima anche che i file pubblicati si carichino realmente con il modello base indicato. |
| [kasunw/mental-health-mistral-7b-instructv0.2-finetuned-V2](https://huggingface.co/kasunw/mental-health-mistral-7b-instructv0.2-finetuned-V2) | Mistral-7B-Instruct-v0.2 adattato a conversazioni di counseling | Può servire a studiare se l'adattamento conversazionale cambia il comportamento rispetto al relativo modello base | È addestrato a rispondere in un contesto di counseling; la scheda non fornisce una valutazione pertinente al PHQ-8 o a colloqui DAIC-WOZ. |
| [chillymiao/Mistral-7B-DSM5](https://huggingface.co/chillymiao/Mistral-7B-DSM5) | Mistral-7B-Instruct-v0.2 adattato a domande e risposte basate su criteri DSM-5 | Esempio di specializzazione psichiatrica e di LoRA documentata | La conoscenza dei criteri diagnostici non equivale a predire l'etichetta PHQ-8 da un dialogo; meglio come confronto di dominio, se accessibile e pertinente, che come predittore iniziale. |

**Caso vicino al dataset ma fuori dal perimetro testuale:** [maimai11/woz](https://huggingface.co/maimai11/woz) pubblica un adapter LoRA per esperimenti DAIC-WOZ di depressione, ma richiede *Step-Audio2-mini-Think* ed è un modello audio-testo. Non è intercambiabile con i quattro predittori testuali sopra; può essere citato come lavoro correlato, non assunto come checkpoint pronto per il patching testuale.

**Criterio di ammissione di un checkpoint specialistico al confronto principale:** poter ricostruire esattamente base + adapter + tokenizer + licenza; verificare che il task in uscita sia confrontabile; escludere sovrapposizioni note con DAIC-WOZ/E-DAIC; eseguire E1 e i controlli di sensibilità sul nostro split. In caso contrario riportarlo come confronto esplorativo, senza usare le prestazioni della sua scheda come prestazioni su DAIC-WOZ.

## 5. Tipologie di adattamento e decisione sperimentale

### A. Nessun training sul corpus — prima condizione

Prompt fisso, ruoli espliciti, due etichette di risposta predefinite. Come score usare la differenza fra log-probabilità delle due risposte; se le etichette sono token singoli verificati, può bastare la differenza dei logits. Calcolare la metrica **per sessione**, la stabilità su parafrasi innocue e la reazione a modifiche pertinenti delle risposte del partecipante. È il controllo necessario per capire se l'eventuale dipendenza dalle domande era presente **prima** dell'adattamento. L'assenza di fine-tuning non impedisce la contaminazione pregressa del modello; se documentabile, controllare anche quella.

### B. Supervised fine-tuning con LoRA — seconda condizione preferita se serve

Congelare il modello base e apprendere solo gli adapter LoRA sul training set DAIC-WOZ, con output breve e vincolato alla classe. Se si usa un modello generativo, addestrare la loss sulla **risposta di classe**, mascherando prompt e trascrizione; evitare di chiedere una spiegazione generata come parte del target, perché l'audit riguarda ciò che determina lo score e non la qualità retorica della spiegazione. La libreria [TRL SFTTrainer](https://huggingface.co/docs/trl/main/sft_trainer) documenta PEFT e il training sui soli token di risposta.

Un pilot ragionevole è LoRA di rango 8 o 16, dropout basso, pochi epoch con selezione sul validation set e seed fissati. Questi sono **punti di partenza da tarare**, non iperparametri validati per DAIC-WOZ. Salvare adapter e configurazione separatamente: così il checkpoint iniziale resta confrontabile. [Documentazione LoRA/PEFT](https://huggingface.co/docs/peft/main/package_reference/lora).

### C. QLoRA — variante di B quando la memoria è il limite

Caricare il modello base in 4-bit (per esempio NF4) e allenare gli adapter. Su T4 è la scelta da provare per 4B e 7B; su L4/A100 si può confrontare con LoRA a precisione maggiore. La [guida ufficiale PEFT](https://huggingface.co/docs/peft/developer_guides/quantization) descrive `prepare_model_for_kbit_training`, configurazione 4-bit e adapter. Per il confronto prima/dopo, usare la **stessa versione quantizzata del base** a adapter disattivato: altrimenti una differenza potrebbe derivare dalla quantizzazione. Per l'audit, descrivere esplicitamente che le componenti studiate appartengono al modello quantizzato con adapter, e verificare almeno su un sottocampione se gli effetti resistono a una precisione più alta quando fattibile.

### D. Specializzazione di dominio già esistente, eventualmente seguita da LoRA sul task

Confronto facoltativo: valutare MentaLLaMA o un adapter di classificazione nello stato pubblicato, poi eventualmente adattarlo con **un nuovo adapter separato** su DAIC-WOZ. Questo risponde alla domanda «partire da una specializzazione di salute mentale aiuta?»; non sostituisce il modello generale, perché introduce differenze di corpus, epoca, tokenizer e training precedente. Confronti causali prima/dopo hanno senso **all'interno dello stesso checkpoint di base**, non tra modelli diversi. L'audit completo su più checkpoint richiede molte più forward pass: fare il confronto comportamentale prima di promettere patching su tutti.

### E. Addestramenti da non includere nel percorso minimo

Il fine-tuning completo di tutti i pesi è sproporzionato rispetto al corpus e al budget. Un ulteriore pretraining generativo sulle trascrizioni non è necessario per la domanda principale e rischia di accentuare la memorizzazione di testi sensibili. DPO/RLHF richiedono preferenze o ricompense che DAIC-WOZ non fornisce. Un secondo fine-tuning con esempi «debiasati» ha senso solo come E7 facoltativo, **dopo** aver documentato una dipendenza da indizi di protocollo con E3/E4.

## 6. Vincolo fondamentale: l'unità di training è la sessione

Le finestre di un colloquio non sono nuovi partecipanti. Non assegnare automaticamente a ogni finestra la label dell'intera intervista come se quella finestra da sola esprimesse lo stato misurato. Le opzioni da confrontare nel pilot sono:

- usare un estratto deterministico e **predefinito senza guardare le label**, uguale per tutte le condizioni; è semplice, ma perde informazione;
- suddividere il dialogo in finestre fissate in anticipo, calcolare score per finestra e aggregarli in uno score **per sessione**, con loss e valutazione a livello di sessione; è più fedele al target ma più oneroso;
- usare l'intera sessione solo dove lunghezza e memoria lo consentono, dichiarando quali sessioni si perdono e verificando che la selezione non alteri la distribuzione del target.

La scelta va fissata prima di E2–E5. Separare train, validation e test per sessione e, dove identificabile, per partecipante; verificare duplicati e possibili sovrapposizioni fra DAIC-WOZ ed E-DAIC. Non scegliere finestre o iperparametri usando il test. Se si studia l'effetto delle domande di Ellie, la loro posizione e il contesto devono restare confrontabili nelle coppie perturbate. [Documentazione ufficiale DAIC-WOZ](https://dcapswoz.ict.usc.edu/wwwdaicwoz/DAICWOZDepression_Documentation_AVEC2017.pdf).

## 7. Fattibilità su Colab Pro e piano di lavoro

| Risorsa disponibile | Pilot consigliato | Esperimento principale realistico da verificare |
|---|---|---|
| **T4, 16 GB** | SmolLM2 con LoRA e patching su finestre brevi | Qwen3-4B con QLoRA, batch 1, checkpointing dei gradienti, sequenze inizialmente brevi; 7B QLoRA solo come verifica limitata |
| **L4, 24 GB** | Qwen3-4B LoRA/QLoRA | Qwen3-4B con più margine per audit; Mistral-7B QLoRA da misurare |
| **A100, 40/80 GB** | Profilare 4B a precisione maggiore | LoRA e patching più ampi; valutare secondo 7B se i tempi lo permettono |

Questa tabella è una **stima di fattibilità**, non un benchmark: numero di token, implementazione degli hook, versioni di PyTorch/Transformers e memoria effettiva cambiano il risultato. Un fine-tuning 7B con QLoRA è stato [mostrato su T4 con sequenze di 512 token](https://huggingface.co/blog/falcon), ma ciò non prova che un'intervista intera o il patching siano sostenibili sulla stessa scheda. [NVIDIA T4](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/tesla-product-literature/T4%20Product%20Brief.pdf), [NVIDIA L4](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/l4/PB-11316-001_v01.pdf), [NVIDIA A100](https://www.nvidia.com/en-us/data-center/a100/). [Colab](https://research.google.com/colaboratory/faq.html) dichiara che tipi di GPU e limiti d'uso variano: il piano non deve dipendere dall'ottenere sempre un'A100.

**Sequenza minima di decisioni:**

1. Misurare lunghezze token delle sessioni e memoria di un forward con score e hook sul 4B. Salvare un report per 512, 1.024, 2.048 e 4.096 token, senza supporre che tutte le lunghezze entrino.
2. Chiudere prompt, rappresentazione delle classi, strategia per le finestre, metriche per sessione e split.
3. Eseguire E1 sul base non adattato. Se E1 è sufficiente, procedere all'audit senza training obbligatorio.
4. Se E1 è debole, addestrare **un** adapter LoRA/QLoRA sul training set; selezionarlo sul validation set e confrontarlo col base in condizioni identiche. Ripetere E1 e i controlli di sensibilità.
5. Solo sul modello che supera E1, eseguire E2–E5. Fare un secondo modello o un checkpoint specialistico soltanto se aggiunge una verifica precisa alla domanda di ricerca.

## 8. Esito atteso della scelta del modello

Non serve che LoRA migliori necessariamente l'accuracy per giustificare la tesi: serve un predittore abbastanza stabile e informativo da rendere interpretabili le perturbazioni. Se LoRA migliora la predizione ma aumenta l'uso di indizi di Ellie, questa differenza diventa un risultato scientifico; se non lo aumenta, è ugualmente informativa. Se né il base né l'adapter superano controlli predittivi minimi, il piano di explainability sul target va ridimensionato prima di trarre conclusioni sulle scorciatoie.

Le predizioni e gli adapter restano artefatti di ricerca su dati sensibili. Conservare trascrizioni e checkpoint secondo le condizioni del corpus; non pubblicare esempi identificabili o adapter addestrati su interviste senza aver verificato i permessi di condivisione.
