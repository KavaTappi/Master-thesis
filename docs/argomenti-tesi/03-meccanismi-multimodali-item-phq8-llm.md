# Meccanismi multimodali specifici per item PHQ-8 in piccoli LLM

## Titolo provvisorio

*Causal auditing of item-specific multimodal explanations in small language models for PHQ-8 estimation*

## Idea in breve

La tesi studia se un piccolo LLM open-weight (entro 3B parametri) usi testo, voce e segnali facciali in modo **diverso per ciascun item del PHQ-8**, e se le sue spiegazioni siano causalmente fedeli a tale uso.

Invece di predire soltanto un punteggio totale o la classe `PHQ8 >= 10`, il modello riceve lo stesso transcript del partecipante piu' token trasparenti da audio e volto, ma risponde a otto compiti condizionati: interesse/piacere, umore, sonno, stanchezza, appetito, autovalutazione, concentrazione e rallentamento/agitazione. Gli item sono score self-report del questionario, non diagnosi e non annotazioni di quale turno esprima un sintomo.

L'oggetto non e' il conflitto artificiale testo-audio. La domanda e': **una spiegazione che dice che una modalita' e' importante per uno specifico item predice un effetto selettivo su quell'item, senza alterare indistintamente tutti gli altri?**

## Domanda principale

> Per quali item PHQ-8 un piccolo LLM usa effettivamente testo, voce e segnali facciali, e i metodi post-hoc e meccanicistici localizzano componenti il cui intervento ha un effetto causale specifico per l'item?

## Perche' non e' *Who Wins the Conflict?*

Il lavoro di Cho et al. (2026) studia Audio LLM nativi in task generali in cui testo e audio sono resi deliberatamente contraddittori: cerca circuiti di **text dominance**, li abla e usa back-patching per attenuare tale dominanza.

Questa proposta e' differente per domanda, target e test di fedelta':

| *Who Wins the Conflict?* | Questa proposta |
| --- | --- |
| conflitto testo-audio artificiale come fenomeno centrale | fusione cooperativa su interviste reali, con mascheramenti solo come test causali |
| una risposta semantica per task generali | otto output ordinali, uno per item PHQ-8 |
| quale modalita' domina? | la dipendenza e' **specifica per item** oppure e' un effetto globale/aspecifico? |
| Audio LLM nativo e circuiti audio/testo | decoder <=3B con input multimodale strutturato, inclusi proxy facciali e di qualita' |
| efficacia di back-patching | confronto quantitativo fra attribution, perturbazione, probe e patching nel predire effetti selettivi |

La tesi non cerca di dimostrare che una modalita' debba vincere sull'altra. Verifica se una spiegazione di fusione e' valida quando l'output da spiegare e' un item preciso, e non un unico score aggregato.

## Relazione con lavori vicini e spazio di novita'

- **Mandal et al. (CLPsych 2025)** propongono QuestMF, una fusione multimodale con predizione question-wise e pesi/interpretabili per modalita'. Non e' un LLM piccolo e non verifica causalmente se le importanze riportate predicano gli effetti di interventi sugli input o sulle componenti interne.
- **Wang et al. (ACL 2026)** mostrano che modellare subscore e' utile su E-DAIC; riportano inoltre che Qwen3-14B in direct scoring non e' una baseline automaticamente vincente. Questo rende necessario verificare la competenza di un LLM piccolo, non assumerla.
- **Zhao et al. (2025)** integra audio, testo e visuale in un modello da 7B su DAIC-WOZ e confronta ablazioni di modalita', ma il target resta la classificazione globale e non analizza la fedelta' causale di spiegazioni per-item.

Il contributo difendibile non e' una nuova architettura di fusione o una claim clinica. E':

> un protocollo di valutazione causale della **specificita' per item** delle spiegazioni multimodali in un piccolo LLM, con controlli su qualita' del segnale, modalita' e componenti interne.

## Input e modello

### Input P-only multimodale

Il modello principale usa solo turni del partecipante, cosi' la scelta delle domande dell'intervistatore non diventa una scorciatoia alternativa. Ogni turno riceve testo e un piccolo insieme di token derivati da feature gia' disponibili:

```text
[TURN 07]
TEXT: I wake up several times during the night.
VOICE: PAUSE_HIGH F0_VARIABILITY_LOW VOICED_RATIO_NORMAL
FACE: AU12_LOW GAZE_VARIABILITY_HIGH HEAD_MOTION_LOW
QUALITY: AUDIO_VALID FACE_TRACKING_HIGH
```

I token vocali possono derivare da COVAREP/Formanti; quelli facciali da Action Units, pose e gaze OpenFace. Le soglie sono stimate soltanto sul training set. `QUALITY` distingue dati realmente osservati da zeri introdotti da scrubbing o fallimenti di tracking.

Gemma 2 2B o Qwen3 1.7B sono candidati. Per ogni sessione il modello e' interrogato con uno speciale token di compito, ad esempio `<TARGET=SLEEP>`, e deve generare `0`, `1`, `2` o `3`. Il target meccanicistico e' un contrasto fra logit ordinali, predefinito per ogni item; usare una procedura identica per tutti gli item evita di selezionare spiegazioni dopo aver visto i risultati.

### Variante con modello nativamente multimodale

La configurazione principale resta quella con token trasparenti da feature audio e facciali: con sole 275 sessioni offre controlli piu' puliti e interventi piu' economici. Se l'obiettivo diventa usare waveform e frame/video raw, il candidato piu' vicino al vincolo e' **Qwen2.5-Omni-3B**, che accetta testo, audio, immagini e video end-to-end. Il suo nome ``3B`` descrive la variante rilasciata, ma il checkpoint completo contiene anche encoder e il modulo Talker: per l'audit va usato il solo percorso di comprensione/``Thinker`` e vanno misurati memoria e parametri effettivamente caricati, non solo il nome del modello.

Per una tesi magistrale: cercare un backbone di **circa 3B** e trattare **7B** come replica/estensione soltanto se si dispone di piu' GPU e di tempo. Sotto 2B il rischio e' che il modello non integri bene segnali raw; oltre 7B aumentano molto costo del patching e numero di possibili componenti, senza risolvere il limite statistico del corpus. Un modello nativo non elimina il go/no-go: deve prima superare baseline semplici e controlli di modalita'.

## Domande di ricerca

### RQ1 — Il modello piccolo e' competente nel task fine-grained?

Confrontare, per item, `text-only`, `voice-only`, `face-only`, `text+voice+face` e una fusione tardiva regolarizzata. Usare MAE, quadratic weighted kappa, Brier score e calibrazione ordinale; il totale PHQ-8 e la classe binaria restano solo outcome secondari. Un LLM che non supera baseline semplici non e' un oggetto adeguato per il livello meccanicistico.

### RQ2 — Il contributo di una modalita' e' selettivo per item?

Applicare interventi appaiati all'input, mantenendo fissi transcript e partecipante:

| Intervento | Funzione |
| --- | --- |
| `voice-mask` / `face-mask` di uguale lunghezza | misura dipendenza da una modalita' |
| permutazione intra-sessione dei token non testuali | separa contenuto/alignment da semplice presenza o lunghezza |
| mascheramento quality-aware | controlla che il modello non sfrutti scrubbing e tracking failure |
| swap matched per durata e qualita' | verifica se il contenuto della modalita' modifica davvero il logit |

Per ogni intervento si stima una matrice `modalita' × item`: l'elemento e' il cambiamento del logit dell'item. Un effetto che cambia tutti gli otto output allo stesso modo non e' evidenza di contributo item-specifico.

### RQ3 — Le spiegazioni predicono gli effetti selettivi?

Confrontare quattro famiglie di metodi:

1. **Post-hoc sull'input:** integrated gradients, leave-one-turn-out e leave-one-token-group-out.
2. **Importanza condizionale della modalita':** permutation importance e Shapley approssimato, sempre condizionati su testo e flag di qualita'.
3. **Rappresentazioni interne:** probe lineari per item/modalita' e analisi layer-wise; logit o Jacobian lens solo se rivalidato sul checkpoint effettivamente usato.
4. **Test causali interni:** activation patching tra input originale e modalita'-masked, ablation di head/MLP/feature candidate e controlli random matched per layer e budget.

Una componente e' candidata per l'item `sleep`, per esempio, solo se il suo intervento modifica piu' il contrasto logit di `sleep` rispetto alla media degli altri sette item. Le metriche chiave sono `effect@k`, `random-gap@k`, correlazione ranking-effetto e:

```text
item-specificity@k = effect(target item) - mean effect(non-target items)
```

### RQ4 — Le spiegazioni restano affidabili quando la qualita' dei dati cambia?

Confrontare le classi `AUDIO_VALID`/`FACE_TRACKING_HIGH` con input parzialmente mascherati. La domanda non e' se una persona abbia o no un sintomo in quel turno: e' se la spiegazione individui dipendenza dal segnale disponibile, invece di confondere dati mancanti con evidenza psicologica.

## Dati: DAIC-WOZ ed E-DAIC

**E-DAIC deve essere il corpus principale.** Comprende 275 sessioni e costituisce l'estensione quality-controlled di DAIC-WOZ; DAIC-WOZ e' un suo sottoinsieme, percio' i due non possono essere trattati come train e test indipendenti o come una validazione esterna.

DAIC-WOZ puo' essere usato per due ruoli limitati e dichiarati:

- replica del preprocessing sulle sessioni comuni, per confrontabilita' con letteratura storica;
- studio di sensibilita' agli errori/alle differenze di trascrizione e feature fra release.

La valutazione primaria deve usare gli split E-DAIC ed escludere ogni duplicato di partecipante. Se si vuole una prova di trasferimento esterno, serve un terzo corpus non sovrapposto; non bisogna improvvisarla con DAIC-WOZ.

## Stack tecnologico e di explainability

| Livello | Tecnologie / tecniche | Cosa si valuta |
| --- | --- | --- |
| Dati e segnali | E-DAIC; Python, `pandas`, `librosa`/`torchaudio`, COVAREP/Formanti per voce, OpenFace per Action Units/pose/gaze; flag di qualita' audio e tracking | disponibilita' reale dei segnali e distinzione fra assenza di evidenza e dato mancante |
| Modello e baseline | PyTorch + Hugging Face Transformers; decoder 1.7--2B con token multimodali; baseline text-only, voice-only, face-only e fusione tardiva regolarizzata | competenza predittiva per item prima di interpretare il modello complesso |
| Variante raw nativa | Qwen2.5-Omni-3B con audio e frame/video campionati per turno; processamento ufficiale del modello; inferenza sul ramo Thinker | integrazione end-to-end da segnali raw; e' una variante comparativa, non un prerequisito della tesi |
| Explainability post-hoc | Captum/integrated gradients, gradient x input, leave-one-turn-out, leave-one-token-group-out, permutation importance e Shapley approssimato condizionato su testo e qualita' | quale modalita', turno o gruppo di feature viene proposto come rilevante per un item |
| Audit causale e interno | voice/face mask, permutazione intra-sessione, swap matched per durata e qualita'; PyTorch/NNsight o hook equivalenti; activation/path patching; ablation head/MLP/feature; probe lineari | se l'effetto sul logit dell'item bersaglio e' maggiore dell'effetto medio sugli altri sette item |
| Validazione quantitativa | `item-specificity@k`, `effect@k`, `random-gap@k`, correlazione ranking-effetto, class flip, bootstrap appaiato e test di permutazione | fedelta' e selettivita' della spiegazione, non soltanto accuratezza o una visualizzazione convincente |

Circuit Tracer/Jacobian Lens possono essere aggiunti solo dopo una prova di compatibilita' sul checkpoint finale; non vanno assunti validi dopo LoRA o su un backbone omni proprietario. Il minimo realizzabile e' il modello con token trasparenti, controlli di input, due baseline post-hoc e patching/ablation con controlli casuali matched.

## Piano minimo realizzabile

1. Verificare che gli otto item PHQ-8 e i tre gruppi di feature siano disponibili nella release E-DAIC posseduta; documentare item senza supporto sufficiente.
2. Addestrare baseline testuali, vocali, facciali e una fusione tardiva; scegliere gli item principali in base alla distribuzione **del solo training set**.
3. Addestrare un piccolo decoder per il formato `<TARGET=item>` su input `text-only` e `text+token multimodali`.
4. Eseguire mascheramento per modalita', quality-aware masking e una permutazione intra-sessione come audit comportamentale.
5. Confrontare integrated gradients e perturbation importance contro activation patching/ablation sulle componenti candidate, con set random matched.
6. Riportare la matrice `modalita' × item`, la specificita' per-item e i risultati nulli; usare DAIC-WOZ solo per il controllo di sensibilita' dichiarato.

## Esiti e claim consentiti

| Esito | Claim corretto |
| --- | --- |
| Il modello usa una modalita' e gli effetti sono selettivi per pochi item | Nel modello e nel corpus studiati, la modalita' contribuisce a specifici output PHQ-8 |
| La modalita' cambia tutti gli item o dipende da flag di qualita' | Non e' supportata un'interpretazione item-specifica; possibile artefatto di fusione |
| La fusione non supera il testo | Non c'e' evidenza di valore multimodale per il piccolo LLM scelto |
| Il post-hoc e il patching divergono | La spiegazione post-hoc non e' sufficientemente fedele per un claim sul meccanismo |
| Il patching supera controlli e mostra specificita' | Evidenza causale limitata di componenti interne associate a un output per-item |

Non e' consentito affermare che un token o una componente identifichi un sintomo clinico in un preciso turno: le label sono self-report a livello di sessione.

## Rischi e mitigazioni

- Otto target ordinali su 275 sessioni sono difficili: usare training multi-task, regolarizzazione e pochi item primari pre-specificati; non inseguire il massimo score per ogni item.
- Un LLM <=3B potrebbe non superare le baseline. Questo e' un go/no-go esplicito, non un fallimento da nascondere.
- I token sono proxy di audio/video, non segnali clinici diretti; la tesi non spiega una waveform o un video raw end-to-end.
- Modelli e feature multimodali possono incorporare bias demografici e artefatti tecnici: mantenere metadata fuori dall'input principale e usarli solo per audit, se la numerosita' lo consente.

## Casi d'uso e decisioni a cui puo' servire

La tesi non produce un classificatore diagnostico per singoli pazienti. Produce evidenza su quando e come una modalita' e' usata da un modello sperimentale.

| Evidenza dell'audit | Decisione o caso d'uso appropriato |
| --- | --- |
| Voce/volto migliorano pochi item e l'effetto e' selettivo | Progettare studi di ricerca per monitoraggio remoto che usino la modalita' aggiuntiva solo per quegli output e mantengano il testo come fallback |
| Il contributo della modalita' e' globale su tutti gli item | Evitare dashboard che presentano spiegazioni per-item; trattare il segnale come distress aspecifico o artefatto finche' non viene chiarito |
| L'effetto scompare con quality-aware masking | Stabilire requisiti minimi di acquisizione e un comportamento di astensione/fallback per sessioni con audio o tracking insufficiente |
| Testo e segnali non testuali divergono | Usare il conflitto come trigger di revisione umana o di raccolta migliore nei protocolli di ricerca, non come regola automatica sul partecipante |
| Patching e ablation superano post-hoc e controlli random | Inserire nel model card un audit della dipendenza per modalita' e usare le componenti trovate solo come monitor esplorativi di regressioni dopo aggiornamenti del modello |

## Riferimenti chiave

- [Mandal et al. (CLPsych 2025), *Enhancing Depression Detection via Question-wise Modality Fusion*](https://aclanthology.org/2025.clpsych-1.4/) — baseline concettuale question-wise; non valuta fedelta' causale delle spiegazioni.
- [Wang et al. (ACL 2026), *Rethinking Depression Prediction from a Fine-Grained Subscore Modeling Perspective*](https://aclanthology.org/2026.acl-long.1841/) — precedente recente su subscore E-DAIC e limite del direct scoring con Qwen3-14B.
- [Cho et al. (2026), *Who Wins the Conflict?*](https://arxiv.org/abs/2606.18924) — lavoro distinto su conflitto testo-audio in Audio LLM.
- [Zhao et al. (2025), *It Hears, It Sees too*](https://arxiv.org/abs/2511.19877) — LLM tri-modale da 7B su DAIC-WOZ; target globale e nessuna valutazione causale per-item.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching*](https://arxiv.org/abs/2309.16042) — guida per corruption, metriche e controlli di patching.
