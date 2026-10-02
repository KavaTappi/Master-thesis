# Audit meccanicistico di shortcut indotti dai protocolli d'intervista

## Titolo provvisorio

*Mechanistic interpretability as an audit for protocol-induced shortcuts in interview-based screening models*

## Proposta di tesi

La tesi studia se un piccolo LLM open-weight, quando viene usato per classificare un esito di screening derivato dal PHQ-8 a partire da un'intervista, impari davvero segnali nelle risposte del partecipante oppure sfrutti shortcut introdotti dalla struttura dell'intervista. 

Il caso di studio è DAIC-WOZ: un corpus di interviste semi-strutturate in cui le domande di Ellie, la loro presenza e la loro posizione possono variare in funzione dello svolgimento del colloquio.

Il problema è già documentato a livello comportamentale: **un modello può ottenere una buona prestazione usando le sole domande dell'intervistatore.** 

La proposta non si limita a replicare tale osservazione con un LLM. Usa prima trasformazioni controfattuali del transcript (ad esempio rimozione, permutazione e parafrasi controllata dei prompt, mantenendo fisse le risposte) per stabilire se il piccolo LLM dipenda realmente dal protocollo. 
Successivamente confronta spiegazioni post-hoc e tecniche di interpretabilità meccanicistica per verificare se le componenti interne da esse indicate prevedano l'effetto reale di ablation e activation/path patching sul logit della classe di screening.

La domanda centrale è quindi:

> **Le tecniche di interpretabilità meccanicistica individuano componenti interne la cui perturbazione spiega causalmente la dipendenza di un piccolo LLM da shortcut indotti dal protocollo, meglio delle spiegazioni post-hoc?**

### Obiettivo metodologico esplicito

> **Valutare se strumenti di interpretabilità meccanicistica forniscono spiegazioni causalmente più fedeli delle tecniche post-hoc nell'identificare shortcut indotti da protocolli d'intervista in piccoli LLM per screening clinico.**

In termini sperimentali, l'obiettivo è verificare se l'interpretabilità meccanicistica aggiunga evidenza causale rispetto a entrambe le baseline necessarie:

1. l'audit controfattuale del comportamento del modello, che stabilisce **se** lo shortcut è usato;
2. le spiegazioni post-hoc, che propongono **quali** input o componenti siano importanti ma non dimostrano da sole un meccanismo.

La formulazione estesa è:

> **Valutare se l'interpretabilità meccanicistica aggiunge evidenza causale oltre a un audit controfattuale del comportamento del modello, e oltre alle spiegazioni post-hoc, nel rilevare shortcut indotti da protocolli d'intervista.**

L'output non è una diagnosi: il task è la classificazione sperimentale `PHQ8 < 10` contro `PHQ8 >= 10`. Il contributo atteso è un protocollo di audit che separi tre livelli: il segnale disponibile nel dataset, l'uso effettivo di quel segnale da parte di un modello e la sua possibile implementazione nelle rappresentazioni interne.


## Dal caso DAIC-WOZ a un audit per altri protocolli

DAIC-WOZ non è presentato come l'unico protocollo possibile, ma come un banco di prova in cui lo shortcut è documentato e le trasformazioni controfattuali sono realizzabili. Il prodotto metodologico della tesi è una procedura trasferibile, non un circuito universale.

Un altro sistema basato su interviste può essere sottoposto allo stesso audit se mette a disposizione:

- transcript attribuiti a intervistatore e partecipante, con ordine dei turni;
- un esito session-level che il modello deve stimare;
- una procedura con variabilità in presenza, ordine, formulazione o routing delle domande;
- una semantica dei prompt sufficiente per costruire controfattuali controllati;
- accesso white-box al piccolo LLM da auditare, se si vuole eseguire il livello meccanicistico.

La procedura riutilizzabile è:

```text
1. Mappare le variazioni del protocollo (domanda, posizione, routing, durata).
2. Verificare con test comportamentali se il modello le sfrutta per predire l'esito.
3. Usare metodi post-hoc e meccanicistici per formulare ipotesi sulle componenti rilevanti.
4. Validare o respingere tali ipotesi mediante interventi causali e controlli casuali.
5. Decidere se la mitigazione debba avvenire sui dati/protocollo o, solo se validato, nel modello.
```

Questo schema può essere applicato, con un nuovo label e nuove trasformazioni specifiche del dominio, a interviste per PTSD, ansia, screening cognitivo, anamnesi automatizzate e chatbot sanitari adattivi. 

Non dimostra che tutti questi protocolli abbiano leakage: fornisce un modo riproducibile per verificarlo modello per modello e protocollo per protocollo.

### Sviluppi futuri: scenari in cui le evidenze diventano utili

L'evidenza ottenuta dall'audit è applicabile soltanto come supporto alla progettazione e alla valutazione di sistemi automatici.

Gli usi concreti dipendono dal risultato dell'audit.

| Evidenza trovata | Decisione o applicazione possibile | Scenario esemplificativo |
| --- | --- | --- |
| Il modello predice bene dalle sole domande o dalla loro posizione | Escludere i turni dell'intervistatore dal modello di screening, oppure dichiarare esplicitamente che il sistema replica in parte il routing dell'intervista | Un chatbot di pre-screening per ansia viene valutato solo sulle risposte dell'utente, evitando che il punteggio dipenda dal ramo di domande già selezionato dal chatbot |
| Il problema è concentrato in poche famiglie di prompt | Mascherare tali prompt, bilanciare la loro distribuzione, o usare la loro comparsa come variabile di audit obbligatoria | In un protocollo PTSD, i follow-up su ricoveri o terapia passata sono presenti soprattutto nei casi complessi: il modello viene stress-testato con e senza tali follow-up |
| Il segnale dipende dalla posizione/ordine | Randomizzare o controbilanciare, in un nuovo studio, l'ordine delle domande non vincolate clinicamente | Un'intervista cognitiva pone un blocco su memoria e autonomia sempre nella seconda metà; lo studio futuro alterna l'ordine tra partecipanti per separare posizione e stato cognitivo |
| L'effetto non si trasferisce a un secondo protocollo | Evitare claim di generalizzazione e valutare il modello solo nel suo contesto di raccolta | Un modello addestrato su interviste da telemedicina non viene riutilizzato su colloqui ambulatoriali senza un nuovo audit del diverso copione |
| Un circuito supera necessità, sufficienza e controlli casuali | Usarlo come monitor esplorativo durante il fine-tuning o come candidato per una mitigazione interna, sempre rivalutando il comportamento esterno | Prima di rilasciare una nuova versione di un LLM, si controlla se la componente associata a un prompt di routing riappare o cresce di effetto |
| Nessun circuito supera i controlli, ma il test d'input mostra dipendenza | Non modificare internamente il LLM; mitigare a livello di input, dati o protocollo | Si usa `P-only` e si riportano risultati per famiglia di domanda, anziché ablare componenti interne non validate |

Le evidenze possono inoltre essere utili per la **documentazione del modello**: un model card o protocol card può dichiarare quali turni sono ammessi in input, quali trasformazioni hanno causato instabilità e su quali popolazioni/protocolli è stata verificata la robustezza.

---

### Come trasferire l'audit a un altro protocollo

Il trasferimento non consiste nel riusare direttamente un circuito scoperto in DAIC-WOZ. Ogni nuovo protocollo richiede un audit ex novo, perché prompt, popolazione, label e strategia di routing possono cambiare. Si riusa invece la struttura sperimentale.

| Passo | Adattamento al nuovo protocollo | Esempio: chatbot di anamnesi per disturbi del sonno |
| --- | --- | --- |
| Definire l'esito | Identificare un label di ricerca o screening separato dalla decisione del sistema | punteggio validato per insonnia raccolto al termine della sessione |
| Mappare il protocollo | Annotare speaker, famiglie di domanda, rami, posizione, fallback e durata | domande su sonno, caffeina, lavoro a turni, farmaci e follow-up adattivi |
| Costruire controfattuali validi | Modificare una sola proprietà del protocollo preservando le risposte e il significato | sostituire una domanda di follow-up con una formulazione equivalente o permutare due blocchi indipendenti |
| Eseguire l'audit comportamentale | Confrontare `utente-only`, `sistema-only`, completo e struttura-only | verificare se il modello classifica bene dal solo ramo selezionato dal chatbot |
| Decidere se svolgere l'audit interno | Procedere solo se il LLM presenta sensibilità robusta ai controfattuali | cercare componenti interne solo dopo aver osservato `delta-logit` non banale |
| Validare e documentare | Replicare su nuovi partecipanti/protocolli e pubblicare limiti e mitigazioni | indicare che il modello è utilizzabile solo con la versione auditata del copione |

Un protocollo con domande identiche e non adattive per tutte le persone non elimina tutti i problemi di bias, ma rende improbabile lo shortcut specifico basato su presenza e ordine dei prompt. Al contrario, protocolli con routing dinamico, follow-up discrezionali o più intervistatori sono candidati prioritari per questo tipo di audit.

## Che cosa viene studiato (e che cosa no)

Il protocollo non calcola automaticamente il PHQ-8. Il PHQ-8 è il questionario self-report da cui deriva il label di benchmark; il modello automatico riceve una trascrizione e predice la classe di screening `A = PHQ8 < 10` o `B = PHQ8 >= 10`.

```text
segnali/risposte iniziali del partecipante
          ↓
scelta adattiva dell'intervistatore (quale follow-up porre e quando)
          ↓
testo, identità e posizione delle domande nel transcript
          ↓
predizione del modello
```

Il lavoro non intende:

- diagnosticare autonomamente la depressione;
- stabilire se il protocollo clinico umano sia corretto o scorretto;
- presumere che ogni domanda clinicamente mirata sia uno shortcut invalido.

L'audit riguarda invece la validità di una pipeline automatica che usa una trascrizione completa per predire un **esito di screening derivato da PHQ-8**.


## Relazione con Burdisso et al. e Watawana et al.

[Burdisso et al. (2024)](https://aclanthology.org/2024.clinicalnlp-1.8.pdf) sono la baseline empirica e il controllo positivo: mostrano che, in DAIC-WOZ, GCN e Longformer possono ottenere prestazioni elevate dalle sole domande di Ellie e localizzano il segnale in prompt legati alla storia di salute mentale. [Watawana et al. (LREC 2026)](https://aclanthology.org/2026.lrec-1.185/) estendono l'osservazione ad altri corpora e split.

Di conseguenza, **non** è un contributo sufficiente ripetere `P-only`/`E-only` con un LLM. La tesi usa quei risultati per porre una domanda diversa: una spiegazione proposta per il LLM riesce a prevedere quale intervento cambierà davvero la sua decisione?

## Domande di ricerca

### RQ0 — Il piccolo LLM sfrutta un segnale di protocollo?

Il LLM, addestrato o calibrato per la stessa classe binaria PHQ-8, dipende da identità, posizione o presenza dei prompt dell'intervistatore? 

La domanda non assume che l'effetto osservato da Burdisso compaia necessariamente nello stesso modo in un decoder-only.

### RQ1 — Gli strumenti meccanicistici localizzano componenti candidate dello shortcut?

Tra gli stati interni del LLM, esistono feature, edge, head o componenti MLP la cui attivazione è associata alla presenza di un prompt/posizione ad alta informazione di protocollo e al logit `B-A`?

### RQ2 — Le componenti indicate sono causalmente valide?

L'ablation o l'activation/path patching delle componenti candidate modifica il logit `B-A` più di componenti casuali o componenti selezionate da una spiegazione post-hoc, su coppie controfattuali appaiate?

### RQ3 — I metodi meccanicistici sono più fedeli delle spiegazioni post-hoc?

La classifica di componenti prodotta da Circuit Tracer/Jacobian Lens anticipa meglio l'effetto reale degli interventi rispetto ad attention, integrated gradients, gradient saliency o perturbation importance?

La domanda è sui **metodi di spiegazione**, non sulla superiorità clinica del LLM rispetto a GCN o Longformer.


## Struttura dei dati e condizioni di input

Lo split è sempre per `participant_id`, prima di qualunque segmentazione; il label della sessione non deve mai creare split a livello di turno.

| Condizione | Input per sessione | Domanda a cui risponde |
| --- | --- | --- |
| `P-only` | soli turni del partecipante | il contenuto del partecipante basta? |
| `E-only` | soli prompt/interventi di Ellie | il protocollo contiene un proxy predittivo? |
| `E+P` | dialogo completo con speaker tag | quanto il modello trae vantaggio dal contesto dell'intervistatore? |
| `E-structure` | famiglia prompt, posizione, numero e lunghezza, senza testo delle risposte | il segnale è puramente strutturale? |
| `E-counterfactual` | prompt rimossi, permutati o parafrasati, con risposte fissate | la decisione è invariabile a modifiche del protocollo? |

Per il decoder, il prompt di classificazione e la coppia di token-etichetta `A`/`B` restano fissi; la quantità valutata è il logit gap `z_B-z_A`. Per encoder e GCN si usa l'analogo margine fra le due logits della classifier head.

## Audit a due livelli

### Livello 1 — Audit comportamentale, generalizzabile fra modelli

Questo livello non richiede circuiti ed è applicabile a ogni classificatore.

1. Confrontare macro-F1, AUROC, AUPRC, balanced accuracy e calibrazione fra `P-only`, `E-only`, `E+P` ed `E-structure`.
2. Applicare interventi appaiati che non cambiano le risposte: rimozione, permutazione, length matching e parafrasi controllata dei prompt.
3. Misurare `delta-logit`, tasso di class flip, prediction agreement e differenza di metrica con bootstrap appaiato per partecipante.
4. Usare controlli: perturbazioni casuali della stessa lunghezza, prompt non predittivi, componenti random e trasformazioni applicate anche a `P-only`.

Questo livello può affermare soltanto che **uno specifico modello**, su uno specifico corpus, dipende o non dipende da una proprietà del protocollo.

### Livello 2 — Audit meccanicistico, specifico dell'architettura

Questo livello è svolto sul piccolo LLM che mostra un effetto comportamentale robusto.

1. Usare Jacobian Lens/Circuit Tracer, e uno o più metodi post-hoc, per ordinare componenti interne candidate.
2. Definire prima dell'intervento le coppie originale/controfattuale e il budget di componenti da testare.
3. Ablare o patchare feature/edge/head candidate e misurare l'effetto reale su `z_B-z_A` e sui class flip.
4. Confrontare l'effetto con componenti casuali e con componenti selezionate dai metodi post-hoc allo stesso budget.

Il risultato meccanicistico è necessariamente dipendente dal modello: un circuito trovato in Gemma non è automaticamente un circuito in Qwen o Longformer. Ciò che è riutilizzabile è la **procedura** — ipotesi, controfattuali, controlli, metriche e soglia per accettare o rifiutare un claim causale.


## Stack di tecniche applicate

Lo stack non parte dai circuiti. L'evidenza primaria è il comportamento del modello dopo una trasformazione controllata dell'input; l'analisi meccanicistica deve aggiungere potere esplicativo a tale evidenza, non sostituirla.

**Terminologia.** *Mechanistic interpretability* è il programma/metodo generale per studiare i calcoli interni di un LLM. *Activation patching* è una sua tecnica di intervento: sostituisce un'attivazione interna con quella proveniente da un input di riferimento e misura il cambiamento dell'output. Non sono due alternative; in questa tesi il patching è il test causale con cui validare o respingere le ipotesi prodotte dall'analisi meccanicistica.

| Fase | Tecniche | Output verificabile | Ruolo nel claim |
| --- | --- | --- | --- |
| 0. Audit del corpus e del protocollo | speaker split; raggruppamento delle famiglie di prompt; `E-structure`; analisi di presenza/posizione/lunghezza; split per partecipante | quali proprietà del protocollo sono associate al label | identifica un potenziale shortcut, non prova che un modello lo usi |
| 1. Audit comportamentale controfattuale | `P-only`, `E-only`, `E+P`, `E-structure`; rimozione, permutazione, length matching e parafrasi controllata dei prompt | `delta-logit`, class flip, macro-F1/AUROC, bootstrap appaiato | **evidenza primaria** che un modello sfrutta o non sfrutta il segnale di protocollo |
| 2. Baseline post-hoc | integrated gradients; gradient saliency; perturbation/leave-one-out importance; attention soltanto come descrizione | ranking di token/turni/componenti candidati | confronto necessario: una spiegazione plausibile non viene trattata come prova causale |
| 3. Scoperta rappresentazionale | probe lineari per famiglia/posizione del prompt; logit/Jacobian Lens; Circuit Tracer o sparse-autoencoder features, se compatibili con il modello scelto | layer, feature, edge o head candidati associati a `z_B-z_A` | genera ipotesi meccanicistiche, ancora correlazionali |
| 4. Validazione meccanicistica | ablation di head/MLP/feature; activation o path patching tra esempi originale/controfattuale; controlli random matched per layer e budget | effetto reale su `z_B-z_A`, `flip@k`, specificità e danno collaterale | trasforma un candidato in un claim causale limitato al modello studiato |
| 5. Valutazione della fedeltà | `effect@k`, `random-gap@k`, `posthoc-gap@k`, correlazione ranking-effetto; test di permutazione | confronto quantitativo tra meccanicistico, post-hoc e casuale | risponde a RQ3, non la sola visualizzazione del circuito |
| 6. Robustezza e riuso | replica su un secondo piccolo decoder, se le risorse lo consentono; leave-question-family-out; eventuale secondo corpus/protocollo | stabilità del risultato rispetto a modello, prompt family e distribuzione | distingue un effetto locale da una procedura di audit riutilizzabile |

### Paper per ciascuna fase dello stack

| Fase | Paper utile | Uso concreto nella tesi |
| --- | --- | --- |
| 0–1 | [Burdisso et al. (2024)](https://aclanthology.org/2024.clinicalnlp-1.8.pdf) | controllo positivo `P-only`/`E-only`, ablation e definizione del rischio specifico di DAIC-WOZ |
| 0–1 | [Watawana et al. (LREC 2026)](https://aclanthology.org/2026.lrec-1.185/) | dimostra che il fenomeno può attraversare dataset e architetture; evita claim eccessivi su DAIC-WOZ |
| 1 | [Ribeiro et al. (2020), *CheckList*](https://aclanthology.org/2020.acl-main.442/) | cornice per behavioural testing basato su trasformazioni controllate |
| 1–2 | [Lyu, Apidianaki e Callison-Burch (2024), *Towards Faithful Model Explanation in NLP*](https://direct.mit.edu/coli/article/50/2/657/119158/Towards-Faithful-Model-Explanation-in-NLP-A-Survey) | giustifica la distinzione fra plausibilità e fedeltà e l'uso di controfattuali |
| 2 | [Sundararajan, Taly e Yan (2017), *Integrated Gradients*](https://arxiv.org/abs/1703.01365) | baseline di attribuzione differenziabile |
| 2 | [Jain e Wallace (2019), *Attention is not Explanation*](https://aclanthology.org/N19-1357/) | impedisce di trattare attention come evidenza causale |
| 3–4 | [Geiger et al. (2025), *Causal Abstraction for Mechanistic Interpretability*](https://www.jmlr.org/papers/volume26/23-0058/23-0058.pdf) | formalizzazione di patching, ablation e fedeltà causale |
| 3–4 | [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching in Language Models*](https://arxiv.org/abs/2309.16042) | scelta di corruption, metrica e controlli nell'activation patching |
| 3–4 | [Ameisen et al. (2025), *Circuit Tracing*](https://www.transformer-circuits.pub/2025/attribution-graphs/methods.html) | scoperta di grafi di feature/edge candidati, da validare e non assumere fedeli |
| 4–5 | [Zaman e Srivastava (EMNLP 2025), *A Causal Lens for Evaluating Faithfulness Metrics*](https://aclanthology.org/2025.emnlp-main.1496/) | motivazione e disegno per valutare metriche di fedeltà con interventi invece di confrontare solo visualizzazioni |
| 4–6 | [Ahsan et al. (Findings EMNLP 2025), *Elucidating Mechanisms of Demographic Bias in LLMs for Healthcare*](https://aclanthology.org/2025.findings-emnlp.789.pdf) | precedente più vicino nel sanitario: rappresentazioni di bias, patching e controlli in LLM open-weight |
| 4–6 | [Eshuijs, Wang e Fokkens (CoNLL 2025), *Short-circuiting Shortcuts*](https://aclanthology.org/2025.conll-1.8.pdf) | precedente metodologico più vicino: shortcut controllabile nella classificazione testuale, teste meccanicistici e mitigazione mirata |

## Metriche per la fedeltà causale delle spiegazioni

| Misura | Definizione operativa | Interpretazione |
| --- | --- | --- |
| `effect@k` | media di delta(z_B-z_A) ablating le prime k componenti indicate | quanto una spiegazione seleziona componenti influenti |
| `flip@k` | percentuale di casi in cui l'intervento cambia classe | effetto decisionale, non solo numerico |
| `random-gap@k` | `effect@k` meno effetto di set casuali matched per layer/token | necessità di superare un controllo |
| `posthoc-gap@k` | `effect@k` meccanicistico meno effetto del post-hoc allo stesso budget | test diretto di RQ3 |
| `rank correlation` | correlazione tra score attribuito e effetto osservato ablating componenti | capacità predittiva della spiegazione |
| `specificity` | effetto sullo shortcut target meno effetto su esempi senza il prompt target | evita ablazioni che degradano tutto indiscriminatamente |

Le differenze vanno riportate con intervalli bootstrap appaiati e, dove il confronto è per esempio, con un test di permutazione. Un heatmap o una rationale senza almeno un controllo causale non è sufficiente per rivendicare un circuito.

---
Importante perchè non pottrebbero essere vere le assunzioni

## Piano decisionale e claim che ne consegue

| Biforcazione osservata | Che cosa si può affermare | Che cosa **non** si può affermare | Passo successivo |
| --- | --- | --- | --- |
| `E-only`/`E-structure` è predittivo e gli interventi sui prompt cambiano il LLM | Il LLM sfrutta uno shortcut indotto dal protocollo in DAIC-WOZ | Che tutti i LLM o tutti i protocolli abbiano lo stesso problema | Procedere al livello meccanicistico |
| GCN/Longformer mostrano lo shortcut, ma il LLM no | Lo shortcut è dipendente dall'architettura o dall'addestramento; nel LLM valutato non è rilevabile con questi test | Che il protocollo sia “sicuro” in assoluto | Concludere con audit comportamentale comparativo; ***niente claim sui circuiti del LLM*** |
| Nessun modello supera controlli e controfattuali | Non vi è evidenza, con il protocollo sperimentale scelto, di shortcut sfruttabile | Che lo shortcut non esista nel corpus in assoluto o che Burdisso sia confutato | Audit dei dati/implementazione, report del risultato nullo, nessuna fase circuitale |
| Lo shortcut comportamentale esiste e circuiti candidati superano random e post-hoc | I metodi meccanicistici testati localizzano componenti causalmente influenti e più fedeli delle baseline post-hoc, nel LLM studiato | Un meccanismo universale o una validità clinica | Claim principale della tesi; replica su un secondo LLM se le risorse lo consentono |
| Lo shortcut esiste, i circuiti cambiano il logit ma non superano il post-hoc | Esiste una localizzazione causale, ma nessuna evidenza di superiorità meccanicistica | Che il metodo meccanicistico sia migliore | Tesi sul confronto di fedeltà senza claim di superiorità |
| Lo shortcut esiste, ma i circuiti candidati non superano i controlli causali | Le spiegazioni meccanicistiche testate non localizzano fedelmente una componente causale; l'audit affidabile resta comportamentale | Che il circuito proposto sia reale | Risultato negativo metodologicamente valido; non applicare mitigazioni interne basate sul circuito |
| L'effetto è grande soltanto con ablazioni estese e diffuse | Il segnale è distribuito o i metodi non hanno risoluzione sufficiente per un circuito piccolo | L'esistenza di un circuito localizzato | Passare da claim “circuito” a claim su dipendenza distribuita; testare feature/subspazi o solo mitigazioni d'input |

---

## Claim finale: formulazioni consentite

La formulazione della tesi deve seguire l'esito, non essere fissata in anticipo.

**Se RQ0 e RQ2/RQ3 sono positive:**

> In un piccolo LLM valutato su DAIC-WOZ, strumenti meccanicistici hanno localizzato componenti la cui perturbazione causa una riduzione della dipendenza da shortcut associati al protocollo, e hanno previsto tali effetti meglio delle spiegazioni post-hoc considerate.

**Se RQ0 è positiva ma RQ2/RQ3 è negativa:**

> Sebbene il piccolo LLM dipenda da shortcut indotti dal protocollo, gli strumenti meccanicistici valutati non hanno fornito una localizzazione causalmente più fedele delle spiegazioni post-hoc; le perturbazioni d'input rimangono l'audit affidabile.

**Se RQ0 è negativa nel LLM, ma positiva nelle baseline:**

> La dipendenza da shortcut di protocollo non si trasferisce automaticamente fra architetture: nel LLM valutato, i controlli controfattuali non hanno evidenziato lo stesso comportamento osservato nelle baseline.

**Se RQ0 è negativa per tutti i modelli valutati:**

> Con il setup e gli split riprodotti non abbiamo trovato evidenza robusta di dipendenza sfruttabile dal protocollo; il risultato non costituisce una validazione clinica né una confutazione generale di precedenti lavori.

## Esperimento minimo e criteri go/no-go

1. Riprodurre `P-only`, `E-only`, `E+P` e `E-structure` su almeno una baseline semplice, ModernBERT e un piccolo decoder LLM.
2. Applicare rimozione e permutazione di prompt su coppie appaiate; riportare intervalli bootstrap.
3. Avviare l'analisi meccanicistica solo se il decoder mostra sia un segnale `E-only`/strutturale sia sensibilità agli interventi d'input superiore ai controlli.
4. Confrontare un metodo meccanicistico, almeno due metodi post-hoc e controlli casuali a parità di budget di ablazione.
5. Se i metodi meccanicistici non superano i controlli, fermare il claim al livello comportamentale e riportare il risultato nullo.

Questo piano rende il progetto robusto a esiti negativi: la parte meccanicistica non è una promessa di trovare un circuito, ma un test della sua effettiva fedeltà.

## Lavori analoghi e spazio di novità

Non ho trovato un lavoro che combini tutte e tre le componenti: **shortcut indotto da un protocollo di intervista clinica**, **LLM open-weight come classificatore** e **confronto causale fra meccanicistico e post-hoc**. Esistono però lavori molto vicini su due componenti alla volta.

| Lavoro | Che cosa condivide | Differenza che lascia spazio alla tesi |
| --- | --- | --- |
| [Burdisso et al. (2024)](https://aclanthology.org/2024.clinicalnlp-1.8.pdf) | DAIC-WOZ, prompt dell'intervistatore, shortcut e ablation a livello di input | Non usa decoder LLM né localizza/valida circuiti interni |
| [Watawana et al. (LREC 2026)](https://aclanthology.org/2026.lrec-1.185/) | Audit multi-corpus di prompt e posizioni dell'intervistatore | Non confronta spiegazioni post-hoc e meccanicistiche, né fa patching interno |
| [Eshuijs, Wang e Fokkens (CoNLL 2025)](https://aclanthology.org/2025.conll-1.8.pdf) | Shortcut nella classificazione testuale, logit attribution, interventi su head e mitigazione | Shortcut sintetico/controllabile in recensioni cinematografiche, non clinico né conversazionale |
| [Ahsan et al. (Findings EMNLP 2025)](https://aclanthology.org/2025.findings-emnlp.789.pdf) | Meccanicistica, LLM open-weight, bias e downstream task sanitario | Bias demografico in vignette/note cliniche, non artefatto di protocollo né PHQ-8 |
| [Marinescu, Gruber e Fajardo Vargas (ICLR 2026)](https://proceedings.iclr.cc/paper_files/paper/2026/hash/17aa70697d6cd35835f201c6fb0a2fd5-Abstract-Conference.html) | Lesioning e activation patching su conoscenza medica in LLM | Mappa conoscenza medica generale; non valuta shortcut o fedeltà rispetto a post-hoc |
| [Zaman e Srivastava (EMNLP 2025)](https://aclanthology.org/2025.emnlp-main.1496/) | Valutazione causale della fedeltà di spiegazioni | Benchmark di ragionamento generale, non un protocollo clinico o un circuito di shortcut |

La novità difendibile non è dire che i prompt di Ellie sono un problema — questo è già documentato — né dire che l'activation patching esiste. È verificare, in un caso clinico reale con shortcut documentato, **se e quando** una spiegazione meccanicistica predica meglio delle baseline post-hoc gli effetti di interventi causali sul modello.

## Riferimenti fondamentali

- [Burdisso et al. (2024), *DAIC-WOZ: On the Validity of Using the Therapist's Prompts in Automatic Depression Detection from Clinical Interviews*](https://aclanthology.org/2024.clinicalnlp-1.8.pdf) — controllo positivo per il rischio di shortcut nelle prompt.
- [Watawana et al. (LREC 2026), *When Consistency Becomes Bias: Interviewer Effects in Semi-Structured Clinical Interviews*](https://aclanthology.org/2026.lrec-1.185/) — verifica multi-corpus e multi-architettura del fenomeno.
- [Ahsan et al. (2025), *Elucidating Mechanisms of Demographic Bias in LLMs for Healthcare*](https://aclanthology.org/2025.findings-emnlp.789.pdf) — esempio metodologico di activation patching per bias in compiti sanitari, non su interviste PHQ-8.
- [Eshuijs, Wang e Fokkens (2025), *Short-circuiting Shortcuts: Mechanistic Investigation of Shortcuts in Text Classification*](https://aclanthology.org/2025.conll-1.8.pdf) — precedente più vicino per la parte di shortcut + meccanicistica, ma fuori dal dominio clinico.
- [Lyu, Apidianaki e Callison-Burch (2024), *Towards Faithful Model Explanation in NLP*](https://direct.mit.edu/coli/article/50/2/657/119158/Towards-Faithful-Model-Explanation-in-NLP-A-Survey) — distinzione fra plausibilità e fedeltà delle spiegazioni.
- [Zaman e Srivastava (2025), *A Causal Lens for Evaluating Faithfulness Metrics*](https://aclanthology.org/2025.emnlp-main.1496/) — disegno di valutazione causale per metodi di spiegazione.
- [Geiger et al. (2025), *Causal Abstraction for Mechanistic Interpretability*](https://www.jmlr.org/papers/volume26/23-0058/23-0058.pdf) — formalizzazione degli interventi causali e della fedeltà.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching in Language Models*](https://arxiv.org/abs/2309.16042) — guida metodologica per patching e metriche.
- [Ameisen et al. (2025), *Circuit Tracing*](https://www.transformer-circuits.pub/2025/attribution-graphs/methods.html) — strumento meccanicistico da sottoporre a validazione, non da assumere fedele.
