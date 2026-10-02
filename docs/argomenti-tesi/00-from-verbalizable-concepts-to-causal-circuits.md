# 00 — From verbalizable concepts to causal sites: evaluating Jacobian Lens explanations in small language models

## Idea e perimetro

La tesi valuta se il **Jacobian Lens** (J-Lens) individui posizioni e layer le cui attivazioni influenzano davvero una decisione di un piccolo LLM. Il test causale principale è l'**activation patching** sul modello originale. **Logit Lens** e siti casuali comparabili sono le baseline. Circuit Tracer è una possibile estensione qualitativa, successiva ai risultati principali.

Il progetto ha due livelli:

1. **Livello A, obbligatorio e controllato:** coppie di brevi enunciati costruite attorno agli **otto domini tematici del PHQ-8**. Il compito riguarda ciò che è espresso nel testo, non il punteggio di un questionario.
2. **Livello B, applicativo e condizionale:** stima binaria del punteggio PHQ-8 a livello di intervista DAIC-WOZ/E-DAIC, solo se il modello congelato mostra prestazioni sufficienti e i dati sono accessibili.

> **Domanda centrale:** una lettura J-Lens relativa a un dominio del PHQ-8 aiuta a scegliere siti `layer × posizione` il cui patching modifica la risposta del modello più di Logit Lens e di siti casuali, a parità di intervento?

## Perché usare i singoli punti del PHQ-8 nel Livello A

È una scelta utile: collega il benchmark controllato all'applicazione futura e permette di verificare se il metodo funziona in domini semanticamente diversi. Gli otto domini sono interesse/piacere, umore, sonno, energia, appetito, autovalutazione, concentrazione e rallentamento/agitazione.

La distinzione decisiva è questa:

| Livello A | Livello B |
| --- | --- |
| Un enunciato sintetico **riporta**, **nega** o non chiarisce un contenuto relativo a un dominio? | Quale punteggio PHQ-8 self-report è associato all'intera sessione? |
| Etichetta semantica costruita e controllata per l'esperimento | Etichetta raccolta dal questionario, tipicamente riferita alle ultime due settimane |
| Una frase può bastare per valutare la comprensione del testo | Una frase non determina il punteggio 0–3 dell'item né il totale PHQ-8 |

Per esempio, `I keep waking up at night` contro `I sleep through the night` costituisce una coppia per il **dominio sonno**. Consente di chiedere se il modello distingua due resoconti testuali e quali attivazioni sostengano la differenza. Non consente di attribuire a una persona uno score PHQ-8 del sonno. Gli enunciati ambigui (`My schedule changed`) formano una categoria di controllo: non vengono forzati in un sì/no.

Esempi di **coppie illustrative**, da ampliare con parafrasi e revisione delle etichette:

| Dominio | Contenuto riferito | Contenuto esplicitamente assente |
| --- | --- | --- |
| Interesse/piacere | `I used to enjoy painting, but I no longer look forward to it.` | `I still look forward to painting each weekend.` |
| Umore | `I have felt down most mornings lately.` | `My mood has been steady lately.` |
| Sonno | `I keep waking up during the night.` | `I usually sleep through the night.` |
| Energia | `I run out of energy by midday.` | `I have felt energetic through the day.` |
| Appetito | `Meals have stopped appealing to me.` | `My appetite has not changed.` |
| Autovalutazione | `I keep thinking I have let everyone down.` | `I do not feel I have let anyone down.` |
| Concentrazione | `I lose track of what I read after a few lines.` | `I can follow a book without losing track.` |
| Rallentamento/agitazione | `Others have noticed that I move and speak more slowly.` | `Others say my usual pace has not changed.` |

Queste coppie controllano **l'informazione riportata**, non la frequenza richiesta per assegnare uno score 0–3. Per l'analisi finale servono esempi indiretti, negazioni, casi insufficienti e controlli di lunghezza: una sola frase per dominio non costituisce un benchmark.

Si costruisce una piccola batteria bilanciata per **tutti e otto** i domini, così da misurare la copertura del metodo. Per il patching intensivo si prespecificano **tre domini** sul development (per esempio sonno, energia, interesse), prima di osservare gli effetti causali. La scelta è motivata da competenza del modello, varietà linguistica e fattibilità; i risultati degli altri cinque domini restano nel report comportamentale e J-Lens, senza rivendicare una valutazione causale completa per essi.

## Cosa aggiunge alla letteratura

[Gurnee et al. (2026)](https://transformer-circuits.pub/2026/workspace/index.html) introducono il Jacobian Lens e verificano anche effetti causali di interventi sulle sue direzioni in vari compiti. La tesi non rivendica di scoprire che il J-space possa influenzare il comportamento. Chiede una cosa più circoscritta: **il readout osservato durante una decisione di classificazione predice quali siti del residual stream sono influenti per quella decisione?**

La novità potenziale è un protocollo quantitativo, su un checkpoint piccolo e congelato, che confronta tre *selettori degli stessi siti* (J-Lens, Logit Lens, casuale) e verifica tutti con **il medesimo patching**. Il benchmark a coppie per gli otto domini rende misurabili successi, falsi positivi, negazioni, parafrasi e assenza di informazione. L'applicazione session-level, quando possibile, verifica se il risultato si trasferisce a una decisione più difficile. La priorità e l'eventuale novità rispetto a lavori pubblicati dopo la stesura richiedono una revisione bibliografica finale.

Un readout semanticamente plausibile può risultare **descrittivo**: il modello rende leggibile `sleep`, ma l'intervento sul sito non modifica la risposta. Anche questo è un risultato della tesi.

## Disegno del Livello A

### Dati e coppie

- Testi sintetici o autorizzati e de-identificati, scritti in inglese come il corpus applicativo.
- Per ciascun dominio: esempi di presenza riferita, negazione esplicita e informazione insufficiente; coppie con una variazione semantica circoscritta.
- Parafrasi che preservano il significato, esempi indiretti senza la parola canonica del dominio e sostituzioni neutre di lunghezza/tokenizzazione simili.
- Split per **template e fonte di generazione**, prima di produrre varianti, così che un template quasi identico non finisca in train e test.
- Revisione umana cieca alle predizioni su un campione: l'etichetta semantica deve essere abbastanza chiara da costituire un riferimento credibile.

I prompt restano identici salvo l'enunciato modificato. Le due etichette di risposta, per esempio `A = contenuto riferito` e `B = contenuto negato`, devono essere tokenizzate in modo verificato. I casi insufficienti vengono valutati separatamente con una terza risposta o un'analisi di astensione; non vengono usati come source/target del patching binario.

Per limitare il semplice eco lessicale, il punteggio primario del J-Lens è letto in **posizioni successive all'enunciato**, fissate in anticipo. Si confrontano i readout delle due versioni della coppia; il nome del dominio presente nell'istruzione non costituisce da solo evidenza. Le formulazioni indirette e i controlli di negazione misurano la capacità del readout di andare oltre la parola letterale.

### Un esperimento esemplificativo

```text
Dominio: sonno
x:  For several nights, I have been awake for hours after going to bed.
x': For several nights, I have fallen asleep soon after going to bed.

Task: in questo resoconto è riferita una difficoltà del sonno?
```

Il modello produce un contrasto di output `g(x) = z_A(x) - z_B(x)`. Se il J-Lens legge, in un certo sito, token preregistrati come `insomnia` o `awake` in `x` più che in `x'`, quel sito diventa candidato. L'ipotesi direzionale è che sostituire l'attivazione di `x` con quella di `x'` **riduca** `g(x)`.

Prima del patch si controlla che il modello risponda correttamente a entrambe le versioni e che la tokenizzazione permetta di allineare le posizioni scelte. Si usano principalmente posizioni ancora confrontabili dopo il testo, come il delimitatore finale; per interventi su span interni serve una mappa esplicita delle posizioni corrispondenti. Le coppie non allineabili non vengono forzate in un confronto token per token.

## Domande di ricerca

1. **RQ1 — Lettura semantica.** Il J-Lens varia con presenza, negazione e parafrasi nei domini PHQ-8 più di quanto faccia su modifiche neutre? Quanti domini supera, con criteri fissati prima dell'analisi?
2. **RQ2 — Fedeltà causale.** I siti selezionati dal J-Lens producono, dopo lo stesso activation patching, effetti più grandi e con segno più spesso corretto rispetto a siti casuali comparabili?
3. **RQ3 — Valore rispetto a Logit Lens.** Il J-Lens migliora la selezione rispetto al Logit Lens agli stessi layer, posizioni, esempi e budget `k`?
4. **RQ4 — Trasferimento.** Se il modello supera un gate predittivo su interviste reali, il metodo conserva parte della sua utilità nella stima binaria del PHQ-8 session-level?

RQ1–RQ3 costituiscono la tesi completa. RQ4 è un'estensione applicativa con un esito anche negativo, non una condizione necessaria per consegnare il lavoro.

## Pipeline minima di explainability

| Passo | Operazione | Output |
| --- | --- | --- |
| 1. Competenza | Valutare il modello congelato sulle coppie e sui template tenuti fuori, separatamente per dominio | accuratezza/coverage del task semantico; esempi idonei all'audit |
| 2. Readout | Registrare score/rank J-Lens e Logit Lens per concetti preregistrati, ai layer e nelle posizioni ammissibili | ranking di siti `layer × posizione` per ciascuna coppia |
| 3. Ipotesi | Selezionare top-`k` e segno previsto senza guardare il patching | lista bloccata di interventi |
| 4. Intervento | Sostituire nel modello originale l'attivazione del sito di `x` con quella di `x'`; ripetere in direzione inversa | cambiamento del logit gap e della classe |
| 5. Controlli | Siti random matched per layer/posizione e ampiezza della differenza fra attivazioni; modifiche testuali neutre | effetto specifico rispetto al rumore e al danno generico |
| 6. Report | Effetto per dominio, robustezza a parafrasi, costi, risultati nulli e intervalli | valutazione della fedeltà e della copertura |

La quantità primaria è:

```text
g(x) = z_A(x) - z_B(x)
effect(s, x, x') = g(x con h_s presa da x') - g(x)
```

Si riportano `effect@k`, accuratezza del segno, associazione fra rank e ampiezza dell'effetto, class flip e copertura. I confronti J-Lens/Logit Lens/random usano **gli stessi esempi e lo stesso patching**. L'ampiezza di un effetto su un intero residual stream non dimostra da sola che il token leggibile sia la descrizione esatta della causa: il patch sostituisce più informazione di quella rappresentata dal singolo token J-Lens.

## Modelli adatti

| Priorità | Checkpoint | Uso e cautela |
| --- | --- | --- |
| 1 | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) + [Lens pre-fittato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b) | prima scelta per il nucleo congelato; verificare checkpoint, tokenizer, indici dei layer, accesso alla licenza e qualità del Lens sui prompt scelti |
| 2 | [Gemma 2 2B IT](https://huggingface.co/google/gemma-2-2b-it) + [Lens IT](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b-it) | fallback se il base non segue stabilmente il formato di risposta; è un modello distinto da valutare dall'inizio |
| 3 | [Gemma 3 1B base](https://huggingface.co/google/gemma-3-1b-pt) + [Lens pre-fittato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-3-1b) | opzione più leggera, utile per un pilota; confermare che distingua le coppie nei domini prescelti |
| 4 | [Qwen3 1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) + [Lens pre-fittato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/qwen3-1.7b) | replica opzionale; fissare il formato del prompt e la modalità di generazione |

Si usa **un solo checkpoint** nella tesi minima. Un LoRA o una modifica degli embedding cambia le attivazioni: il Lens pre-fittato non si trasferisce automaticamente al modello adattato. Per semplicità, il nucleo usa un modello congelato e non introduce nuovi token.

Stack: Python, PyTorch, Hugging Face Transformers, [implementazione Jacobian Lens](https://github.com/anthropics/jacobian-lens), hook per il residual stream, `pandas` e statistiche con bootstrap sulle coppie/template. Neuronpedia serve per esplorare esempi pubblici; i risultati quantitativi e gli eventuali transcript si elaborano localmente.

## Livello B — Applicazione session-level

Solo dopo RQ1–RQ3 si verifica una classificazione `PHQ-8 < 10` contro `PHQ-8 >= 10` usando i **soli turni del partecipante**. Occorrono accesso legittimo al corpus, split per partecipante, policy di troncamento fissata sul development e confronto con TF-IDF/lineare o un encoder testuale. Il label deriva dal self-report dell'intera sessione: una frase modificata non riceve un nuovo label clinico.

Se il modello non supera il gate predittivo o gli artefatti del corpus non sono disponibili, si riporta il limite e si conclude sul Livello A. I punteggi degli otto item non sono richiesti: nella release E-DAIC studiata da [Mandal et al. (2025)](https://aclanthology.org/2025.clpsych-1.4.pdf), il test ufficiale contiene il totale ma non gli score dei singoli item.

## Gate e piano in 6–9 mesi

| Quando | Gate verificabile | Decisione |
| --- | --- | --- |
| Settimane 1–2 | checkpoint e Lens pre-fittato caricati; readout e patching riproducibili su poche coppie sintetiche | scegliere uno dei checkpoint della shortlist prima di fissare il benchmark |
| Mese 1 | coppie dei domini, template di test separati e metrica comportamentale verificati | fissare tre domini per il patching intensivo e bloccare i criteri |
| Mesi 2–4 | J-Lens, Logit Lens, random matched e patching comune completati | rispondere a RQ1–RQ3 anche se gli effetti sono nulli |
| Mesi 5–6 | analisi di robustezza, intervalli e scrittura | tesi minima completa |
| Mesi 7–9, se disponibili | pilot DAIC/E-DAIC e, solo dopo, eventuale Circuit Tracer su pochi casi | estensione senza cambiare i risultati principali |

La tesi deve riportare anche i domini nei quali il modello non è competente e quelli per cui J-Lens non anticipa gli effetti. Un risultato positivo o negativo sulle coppie controllate è una risposta completa alla domanda metodologica.

## Riferimenti essenziali

- [Gurnee et al. (2026), *Verbalizable Representations Form a Global Workspace in Language Models*](https://transformer-circuits.pub/2026/workspace/index.html) — metodo J-Lens e interventi sul J-space.
- [Repository di riferimento Jacobian Lens](https://github.com/anthropics/jacobian-lens) — caricamento di Lens pre-fittati e confronto con Logit Lens.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching*](https://arxiv.org/abs/2309.16042) — scelta di interventi, controlli e metriche.
- [Mandal et al. (2025), *Enhancing Depression Detection via Question-wise Modality Fusion*](https://aclanthology.org/2025.clpsych-1.4.pdf) — natura degli score per item e disponibilità delle label E-DAIC.
