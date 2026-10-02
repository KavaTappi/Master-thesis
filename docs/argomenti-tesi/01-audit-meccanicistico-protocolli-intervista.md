# 01 — Fedeltà causale delle spiegazioni di shortcut di protocollo in piccoli LLM

## Idea e domanda centrale

Un'intervista può rendere predittiva la **scelta delle domande dell'intervistatore**, anche quando il segnale che si vorrebbe studiare è nelle risposte del partecipante. [Burdisso et al. (2024)](https://aclanthology.org/2024.clinicalnlp-1.8/) documentano questa possibilità in DAIC-WOZ con GCN e Longformer e localizzano parole indicative in specifiche regioni dell'intervista.

La tesi valuta **quanto siano fedeli le spiegazioni** di tale dipendenza in un piccolo decoder LLM. Il progetto parte da uno shortcut di protocollo **controllato**, costruito in mini-interviste, e applica poi la stessa procedura ai prompt naturali di Ellie in DAIC-WOZ, se dati e comportamento del modello lo consentono.

> **Domanda centrale:** quando un LLM usa una regolarità nelle domande dell'intervistatore, quali spiegazioni individuano parti dell'input e siti interni che predicono gli effetti di interventi controllati sulla sua decisione?

La parte principale della tesi usa un fenomeno costruito e misurabile. La dipendenza spontanea da Ellie è una **verifica di trasferimento**, non il prerequisito che determina se la tesi può proseguire.

## Livello A — Benchmark controllato obbligatorio

### Come si costruisce

Ogni esempio è una breve mini-intervista sintetica in inglese:

```text
INTERVIEWER: Could you tell me a little more about that?
PARTICIPANT: I have been awake for hours most nights.
TASK: Does the participant report a sleep difficulty? A = yes, B = no.
```

La risposta corretta è definita dal contenuto del partecipante. Un'altra famiglia di prompt dell'intervistatore, semanticamente neutra rispetto al compito, viene associata ad `A` con una frequenza scelta **solo nel training**. Il modello può imparare la regola legittima (leggere la risposta) o la scorciatoia (guardare la forma della domanda). Le famiglie di prompt devono differire in modo controllato per lunghezza, posizione e tokenizzazione; si varia una proprietà per volta.

Le condizioni essenziali sono:

| Condizione | Scopo |
| --- | --- |
| Protocollo bilanciato, associazione 50/50 | controllo: la famiglia di domande non predice il label |
| Protocollo fortemente correlato, fino al 100% | controllo positivo: verificare che il modello possa acquisire lo shortcut |
| Test fattoriale con domanda e risposta incrociate | misurare l'uso della domanda quando contraddice il contenuto del partecipante |
| Coppie con sola domanda sostituita | stimare l'effetto del protocollo sullo **stesso** modello e sulla **stessa** risposta |

Esempio di coppia di valutazione:

```text
x:  INTERVIEWER: Could you tell me more about that?
    PARTICIPANT: I have been awake for hours most nights.

x': INTERVIEWER: What happened after that?
    PARTICIPANT: I have been awake for hours most nights.
```

Il contenuto del partecipante e il label semantico restano fissi. Se cambia `g = z_A - z_B`, il modello è sensibile alla domanda. Le due domande non devono essere trattate come clinicamente equivalenti a priori: si verifica che, nel mini-task, entrambe lascino intatta l'informazione necessaria per rispondere.

La batteria primaria può usare **due o tre domini testuali** (per esempio sonno, energia e interesse) con presenza, negazione e casi ambigui. Gli esempi ambigui sono controlli separati e non ricevono artificialmente un label binario. Queste etichette descrivono il testo sintetico: non sono punteggi degli item PHQ-8 né diagnosi.

Template, lessico del partecipante e varianti di intervistatore vengono divisi prima della generazione delle coppie, per evitare quasi duplicati fra train e test. La correlazione con il label è controllata sul train; il test include combinazioni concordanti e discordanti bilanciate. Un marcatore esplicito come `[ROUTE_A]` può servire da **smoke test tecnico**; l'esperimento principale usa famiglie di domande plausibili e neutrali.

### Come evitare il blocco dopo un mese

Entro le prime **due o tre settimane** si completa un pilota con un modello, una condizione 50/50, una condizione fortemente correlata e almeno un intervento interno. Si fissa prima una soglia minima di sensibilità sul development: il cambio della sola famiglia di prompt deve modificare il logit gap oltre la variabilità osservata su sostituzioni neutre.

Se l'effetto manca, si usano in sequenza due fallback **predefiniti**:

1. verificare con il marcatore esplicito che training, label, tokenizer e test fattoriale funzionino;
2. usare il secondo checkpoint della shortlist e ripetere il controllo positivo, senza toccare il test finale.

Se anche il controllo positivo non viene acquisito, il problema è tecnico o riguarda la capacità del modello; la ricerca di siti interni dello shortcut non viene avviata. Il calendario resta protetto perché tale decisione arriva nelle prime settimane. Nessun disegno può garantire a priori che un LLM impari un dato shortcut: qui il rischio è ridotto tramite dati controllati, un controllo positivo e un gate rapido.

## Livello B — Shortcut naturale di DAIC-WOZ

Si verifica, con accesso autorizzato ai transcript e split per partecipante, se le domande e la struttura di Ellie contengano segnale per la classe session-level `PHQ-8 < 10` contro `PHQ-8 >= 10`. Si confrontano `P-only`, `E-only`, `E+P` ed eventualmente `E-structure`, poi si modifica una famiglia di prompt alla volta mantenendo fisse le risposte quando la trasformazione è semanticamente sensata.

Questo livello risponde a due domande distinte:

1. Il protocollo contiene un segnale sfruttabile nel preprocessing adottato?
2. Lo **specifico LLM E+P** cambia decisione quando viene modificato solo il protocollo?

Se la seconda risposta è negativa, si riporta la robustezza osservata e non si cerca un circuito dello shortcut naturale. Il Livello A rimane la tesi metodologica completa. L'eventuale modello adattato sulle mini-interviste e quello valutato su DAIC sono esperimenti distinti: il trasferimento riguarda il **protocollo di audit**, non l'uguaglianza delle loro attivazioni.

DAIC-WOZ ed E-DAIC possono contenere sessioni sovrapposte e non costituiscono automaticamente train e test indipendenti. Le perturbazioni delle domande nelle interviste reali sono test del modello: non producono nuovi label clinici.

## Explainability: che cosa fa ogni passaggio

| Passaggio | Tecnica minima | Che cosa permette di dire |
| --- | --- | --- |
| Segnale disponibile | frequenza delle famiglie di prompt per label; baseline `interviewer-only` | lo shortcut è disponibile nel dataset sperimentale |
| Uso comportamentale | cambio della sola famiglia di prompt nello stesso input; logit gap e class flip | il modello usa o ignora quel segnale |
| Spiegazione sull'input | leave-one-prompt-out e Integrated Gradients | quali turni/token vengono indicati come importanti |
| Localizzazione interna | activation patching fra `x` e `x'` su un insieme prespecificato di siti `layer × posizione` | la differenza in quel sito media parte dell'effetto della domanda |
| Specificità | patch inverso, prompt neutri e siti casuali matched | l'effetto è selettivo e non semplice degrado generale |

**Integrated Gradients** è una spiegazione post-hoc dell'importanza dei token; **leave-one-prompt-out** è un intervento causale sull'input. Entrambi possono trovare una domanda influente senza identificare il calcolo interno. L'**activation patching** sostituisce un'attivazione interna prodotta dalla versione `x'` dentro la corsa di `x` e misura come cambia l'output: è il nucleo meccanicistico minimo.

```text
g(x) = z_A(x) - z_B(x)
effetto_input = g(x) - g(x')
effetto_patch(s) = g(x con h_s presa da x') - g(x)
```

Si valutano segno dell'effetto, quota dell'effetto d'input recuperata dal patch, class flip, robustezza fra template e interventi su prompt neutri. I siti si selezionano sul **development**; la loro capacità di prevedere effetti si valuta su esempi e template tenuti fuori. Se si confrontano metodi di selezione, si applica lo **stesso intervento sugli stessi tipi di sito** e lo stesso budget. Non si confronta direttamente un punteggio di token di Integrated Gradients con l'ablation di un'intera testa come se fossero la stessa unità.

Un patch sull'intero residual stream trasferisce più informazione di quella attribuibile a una singola parola. Un effetto positivo localizza causalmente una differenza interna, ma richiede ulteriori controlli per assegnarle un significato preciso.

## Domande di ricerca

1. **RQ1 — Acquisizione.** In quali condizioni di correlazione il modello usa la famiglia di prompt come scorciatoia? La stessa risposta del partecipante produce decisioni diverse al cambiare della sola domanda?
2. **RQ2 — Fedeltà sull'input.** Integrated Gradients e leave-one-prompt-out individuano le domande che producono il maggiore effetto controfattuale? Le attribuzioni resistono a parafrasi e controlli di lunghezza?
3. **RQ3 — Mediazione interna.** Esistono siti le cui attivazioni, sostituite fra coppie appaiate, trasferiscono una parte riproducibile dell'effetto del protocollo e superano siti casuali comparabili?
4. **RQ4 — Applicazione naturale.** Quali passaggi del protocollo di audit funzionano sulle vere domande di Ellie e su un LLM che predice il label session-level? Un risultato nullo limita il claim a DAIC, senza bloccare RQ1–RQ3.

## Rispetto alla letteratura

[Burdisso et al. (2024)](https://aclanthology.org/2024.clinicalnlp-1.8/) mostrano che le domande di Ellie possono predire il label con GCN/Longformer e usano la rappresentazione del GCN per localizzare keyword e regioni del dialogo. La tesi non ripresenta questa osservazione come scoperta. Aggiunge un piccolo decoder LLM, **interventi appaiati sullo stesso modello E+P** e un test di mediazione delle attivazioni interne. Il benchmark controllato fornisce una variabile di protocollo nota prima di esaminare il modello; DAIC-WOZ verifica quanto il protocollo di valutazione regga su dati reali.

La contribuzione difendibile è un confronto riproducibile fra spiegazioni dell'input e test causali interni, con risultati positivi o negativi. Non si sostiene una superiorità generale del meccanicistico: essa va dimostrata a parità di unità, intervento, budget e split. [Zhang e Nanda (2024)](https://arxiv.org/abs/2309.16042) mostrano perché metrica e costruzione del controfattuale possono cambiare molto il risultato del patching.

## Modelli adatti e stack minimo

| Priorità | Modello | Ruolo |
| --- | --- | --- |
| 1 | [Qwen3 1.7B Base](https://huggingface.co/Qwen/Qwen3-1.7B-Base) | modello principale per adattamento leggero e audit delle attivazioni; confermare formato della risposta e risorse di training |
| 2 | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) | fallback se il primo non supera il controllo positivo; vincoli di licenza e contesto più corto |
| Controllo non LLM | TF-IDF + regressione logistica o encoder piccolo | verifica che il benchmark controllato sia apprendibile e che il cue di protocollo sia predittivo |

Il modello principale può richiedere **LoRA**. Questo è compatibile con il nucleo della tesi, perché il patching si esegue sul checkpoint finale effettivamente usato. Jacobian Lens e Circuit Tracer pre-fittati non sono requisiti: dopo l'adattamento non si presume che restino validi. Eventuali probe, head ablation, path patching o Circuit Tracer sono estensioni successive.

Stack: Python, PyTorch, Hugging Face Transformers/PEFT, scikit-learn, Captum per Integrated Gradients, hook PyTorch per il residual stream, bootstrap appaiato su esempi/template e configurazioni versionate.

## Piano minimo in 6–9 mesi

| Periodo | Deliverable |
| --- | --- |
| Settimane 1–3 | mini-interviste, split per template, condizione bilanciata/fortemente correlata, baseline semplice e controllo positivo sul LLM |
| Mesi 2–3 | audit comportamentale fattoriale e spiegazioni sull'input, con risultati fissati sul test |
| Mesi 3–5 | patching su un numero limitato di siti e coppie, controlli random e inversi, replica fra template |
| Mese 6 | analisi quantitativa, risultati nulli e scrittura del nucleo |
| Mesi 7–9, se disponibili | audit delle domande naturali di Ellie; analisi interna solo se appare una dipendenza comportamentale robusta |

**Esito positivo del nucleo:** il modello acquisisce la scorciatoia controllata e alcuni siti trasferiscono selettivamente parte del suo effetto. **Esito negativo del nucleo:** il modello usa il cue ma le spiegazioni non localizzano siti fedeli, oppure la sensibilità appare soltanto per cue espliciti. Entrambi rispondono alla domanda metodologica; si riportano anche limiti, copertura e costo. Se il controllo positivo non funziona, il disegno va corretto entro il gate iniziale, prima di impegnare mesi nell'analisi interna.

## Riferimenti essenziali

- [Burdisso et al. (2024), *DAIC-WOZ: On the Validity of Using the Therapist's Prompts*](https://aclanthology.org/2024.clinicalnlp-1.8/) — fenomeno naturale e baseline storica.
- [Eshuijs et al. (2025), *Short-circuiting Shortcuts*](https://aclanthology.org/2025.conll-1.8/) — precedente su shortcut controllate e analisi meccanicistica in classificazione testuale.
- [Sundararajan et al. (2017), *Axiomatic Attribution for Deep Networks*](https://arxiv.org/abs/1703.01365) — Integrated Gradients.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching*](https://arxiv.org/abs/2309.16042) — scelta di interventi, metriche e controlli.
