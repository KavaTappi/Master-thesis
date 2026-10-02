# 00 — From verbalizable concepts to causal circuits: evaluating Jacobian Lens explanations in small language models for PHQ-8 estimation

## Proposta di tesi

### Idea in breve

La tesi valuta se i concetti resi leggibili dal **Jacobian Lens** durante una stima sperimentale del PHQ-8 corrispondano a parti del calcolo che influenzano davvero la decisione di un piccolo modello linguistico. **Circuit Tracer** fornisce una seconda vista, basata su feature e percorsi di attribuzione. Interventi sulle attivazioni del modello verificano le previsioni formulate dalle due viste.

Il caso di studio è un modello open-weight entro 3B parametri che riceve i turni testuali di un partecipante a un'intervista DAIC/E-DAIC e sceglie tra `A = PHQ-8 < 10` e `B = PHQ-8 >= 10`. Il PHQ-8 è il punteggio self-report associato all'intera sessione. Il modello non diagnostica una persona e la tesi non attribuisce un item clinico a un singolo turno.

> **Domanda centrale:** quando il Jacobian Lens rende leggibile un concetto durante la predizione, quel concetto aiuta a individuare componenti o posizioni il cui intervento cambia causalmente il logit di screening? La convergenza con Circuit Tracer migliora questa capacità rispetto ai due metodi separati?

La risposta può essere negativa: un concetto verbalizzabile può essere presente nelle attivazioni, ma non contribuire al target PHQ-8. Anche un grafo di attribuzione può non prevedere l'effetto di un intervento sul modello originale. La tesi quantifica proprio queste possibilità.

## 1. Obiettivi, uno per uno

1. **Stabilire un compito predittivo credibile.** Verificare che il piccolo LLM produca una stima non banale su partecipanti non visti, rispetto a TF-IDF con regressione logistica e a un encoder testuale. La prestazione è un requisito per interpretare una decisione, non il contributo principale.
2. **Misurare cosa legge il Jacobian Lens.** Registrare, per layer e posizione, score di concetti prespecificati come sonno, energia, interesse, negazione, contesto neutro e possibili proxy superficiali. Verificare se la lettura reagisce a variazioni semantiche controllate.
3. **Costruire grafi del medesimo calcolo.** Applicare Circuit Tracer allo stesso checkpoint, agli stessi esempi e allo stesso contrasto di output `g(x) = z_B(x) - z_A(x)`; identificare feature, percorsi ed eventuali error nodes che contribuiscono alla predizione.
4. **Definire una corrispondenza verificabile.** Cercare convergenza tra le due viste a livello di *layer, posizione e direzione prevista dell'effetto*, senza equiparare automaticamente un token leggibile nel J-space a una feature del transcoder.
5. **Validare le ipotesi con interventi.** Sostituire attivazioni fra coppie di input, ablare componenti candidate e, dove tecnicamente supportato, intervenire su direzioni del Lens e feature del grafo. Misurare l'effetto sul modello originale, non soltanto sul modello sostitutivo usato per il grafo.
6. **Confrontare fedeltà e costo.** Valutare Jacobian Lens, Circuit Tracer, la loro selezione congiunta e baseline semplici a pari budget di siti o interventi. Documentare quando uno strumento aggiunge informazione all'altro e quando genera candidati fuorvianti.

## 2. Cosa aggiunge alla letteratura

[Gurnee et al. (2026)](https://arxiv.org/abs/2607.15495) introducono il Jacobian Lens per leggere rappresentazioni verbalizzabili e ne testano proprietà funzionali. [Ameisen et al. (2025)](https://www.transformer-circuits.pub/2025/attribution-graphs/methods.html) introducono grafi di attribuzione di circuiti e procedure di verifica tramite interventi. Nessuno dei due risultati implica, da solo, che un concetto letto dal Lens durante un compito di classificazione sia una spiegazione causale *di quella classe* o che coincida con un nodo del grafo.

Il gap proposto è **metodologico e operativo**: valutare sullo stesso piccolo modello e sullo stesso output se una lettura verbalizzabile anticipi gli effetti di interventi, se un grafo aggiunga capacità predittiva e se il loro accordo selezioni meglio i siti influenti. Il contesto PHQ-8 rende il test interessante perché molte parole plausibili possono comparire in un'intervista senza essere usate dal modello per decidere. Il contributo resta condizionato alla revisione bibliografica finale: la tesi non rivendica di essere la prima applicazione di ciascuno strumento.

Risultati consegnabili:

- un protocollo riproducibile di confronto fra lettura di concetti, attribuzione di circuiti e interventi causali;
- un benchmark di coppie originali/controfattuali con unità statistica a livello di partecipante;
- una valutazione quantitativa dei casi di accordo e disaccordo fra metodi, inclusi i risultati nulli;
- criteri per capire quando un readout del Jacobian Lens è utile per selezionare un intervento e quando resta soltanto descrittivo.

Questa proposta è diversa dalla **01**: la 01 chiede se un LLM sfrutti shortcut del protocollo d'intervista; qui l'oggetto principale è la **fedeltà dei metodi di interpretabilità**. Gli shortcut possono comparire come controllo o caso di errore, ma non sono una premessa per la tesi. È diversa dalla **03** perché usa testo e una classe session-level, senza introdurre voce, volto o otto output per item.

## 3. Dataset e unità di analisi

| Elemento | Scelta proposta | Motivo |
| --- | --- | --- |
| Corpus principale | E-DAIC, se la release disponibile include transcript, score PHQ-8 e identificativi di sessione | offre più sessioni del sottoinsieme DAIC-WOZ e consente di mantenere il task testuale circoscritto |
| Corpus di confrontabilità | DAIC-WOZ, solo sulle sessioni pertinenti e con identificativi allineati | è un sottoinsieme/precedente della famiglia E-DAIC, quindi non costituisce test esterno indipendente |
| Input principale | soli turni del partecipante, con ordine preservato (`P-only`) | riduce il rischio che il segnale delle domande dell'intervistatore domini l'analisi dei concetti |
| Label primario | `PHQ-8 < 10` oppure `PHQ-8 >= 10` a livello di sessione | permette un contrasto di logits definito prima di produrre le spiegazioni |
| Unità statistica | partecipante/sessione | impedisce che frammenti della stessa persona entrino in train e test |

Gli [asset E-DAIC e il manuale del corpus](https://dcapswoz.ict.usc.edu/wwwedaic/E-DAIC%20Manual.pdf) vanno controllati all'inizio: disponibilità dei transcript, copertura delle label, split ufficiali e licenza d'uso. In assenza della release E-DAIC completa, DAIC-WOZ può diventare il corpus principale, dichiarando la minore numerosità. Nessuna trascrizione riconoscibile o ID di partecipante entra negli artefatti pubblici.

Lo split si fissa per partecipante prima di ogni segmentazione. Soglie, vocabolario dei concetti, controlli e budget di interventi si scelgono sul train/development. Il test resta chiuso fino alla valutazione finale. Le sessioni DAIC-WOZ presenti anche in E-DAIC non sono una replica indipendente.

## 4. Modello e verifica preliminare

### Shortlist verificata sulle risorse pubbliche

| Checkpoint sotto 3B | Jacobian Lens pre-fittato | Circuit Tracer / transcoder | Decisione pratica |
| --- | --- | --- | --- |
| [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) | [artefatto `gemma-2-2b`](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b) | [PLT GemmaScope](https://huggingface.co/mntss/gemma-scope-transcoders) e [demo ufficiale](https://github.com/decoderesearch/circuit-tracer) | **prima scelta**: è l'incrocio meglio documentato, con tutorial eseguibile anche su GPU Colab da circa 15 GB per esempi piccoli; accesso ai pesi soggetto alla licenza Gemma |
| [Gemma 2 2B instruction-tuned](https://huggingface.co/google/gemma-2-2b-it) | [artefatto `gemma-2-2b-it`](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b-it) | [demo IT](https://github.com/decoderesearch/circuit-tracer), che però riusa transcoders del modello base | variante utile se il base non segue bene il prompt; **richiede** misurare l'errore del replacement model, non assumere equivalenza base/IT |
| [Qwen3 1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | [artefatto `qwen3-1.7b`](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/qwen3-1.7b) | [PLT Qwen3 1.7B](https://huggingface.co/mwhanna/qwen3-1.7b-transcoders-lowl0) elencati dalla [libreria Circuit Tracer](https://github.com/decoderesearch/circuit-tracer) | seconda scelta o replica architetturale; verificare backend, tracing del target e costo prima di includerlo nella tesi |
| [Gemma 3 1B base](https://huggingface.co/google/gemma-3-1b-pt) / [IT](https://huggingface.co/google/gemma-3-1b-it) | [base](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-3-1b) / [IT](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-3-1b-it) | [PLT base e IT](https://huggingface.co/collections/mwhanna/gemma-scope-2-transcoders-circuit-tracer) | opzione più leggera, ma Circuit Tracer richiede il backend `nnsight`, dichiarato ancora sperimentale: non è la prima scelta per il confronto causale |

Questa è **compatibilità documentale degli artefatti**, non una prova end-to-end già eseguita sui transcript DAIC. Altre taglie presenti nei cataloghi, come Gemma 3 270M, possono servire per smoke test ma sono troppo deboli come scelta primaria per lo screening. Llama 3.2 1B ha una demo Circuit Tracer, ma non compare nella raccolta consultata di Lens pre-fittati: richiederebbe il fitting del Lens e non è quindi una scorciatoia di fattibilità.

La scelta raccomandata è partire da **Gemma 2 2B base**, congelato. Prima degli esperimenti bisogna verificare che *entrambi* gli strumenti funzionino sullo **stesso ID di checkpoint, revisione dei pesi, tokenizer e configurazione**. La presenza di file per una famiglia non basta. Qwen3 1.7B è la replica opzionale più interessante se supera lo stesso gate; Gemma 2 2B-IT è un'alternativa distinta, non un sostituto trasparente del base.

Si parte dal backbone congelato, con prompt di classificazione fisso e due etichette a singolo token verificate nel tokenizer. Si riportano balanced accuracy, macro-F1, AUROC/AUPRC e calibrazione, con intervalli per partecipante. Se il modello non mostra una competenza sufficiente a rendere interessante la decisione, si può provare un diverso prompt o un altro checkpoint **prima** di aprire il test. Un LoRA è possibile solo come variante separata: modifica il checkpoint e richiede di rivalidare o rifittare Lens e transcoders. Non si trasferiscono automaticamente i risultati del modello base al modello adattato.

Il Lens legge token del vocabolario: per concetti composti da più token si prespecifica un piccolo insieme di verbalizzazioni e una regola di aggregazione. Si controlla che le differenze osservate non dipendano solo dal numero di token o dalla scelta di una parola particolarmente favorevole.

Una piccola batteria di esempi semantici controllati, indipendente dalle label PHQ-8, verifica preliminarmente che il Lens e il protocollo di patching reagiscano a fenomeni semplici, per esempio `I sleep well` rispetto a `I cannot sleep`. Questa è una prova tecnica del metodo, non un campione aggiuntivo per la metrica clinica.

## 5. Domande di ricerca

### RQ0 — Esiste un comportamento da spiegare?

Il piccolo LLM mantiene una prestazione e una calibrazione non banali sul target session-level rispetto alle baseline, e il contrasto `g(x)` varia in modo interpretabile su coppie semantiche controllate? Se no, i claim sul suo meccanismo PHQ-8 vanno ridimensionati.

### RQ1 — I concetti del Jacobian Lens sono sensibili e specifici?

Quando si modifica un'informazione del partecipante preservando il resto del prompt, cambiano i readout attesi nei layer e nelle posizioni pertinenti? Per esempio, una negazione controllata cambia la lettura di `sleep`/`cannot sleep` più di una parafrasi neutra di lunghezza simile? Il test misura il comportamento del *readout*, non conferma che quel turno contenga un sintomo clinico.

### RQ2 — Jacobian Lens e Circuit Tracer convergono sul medesimo effetto?

Per lo stesso esempio, target e checkpoint, i siti che il Lens considera informativi sono anche quelli in cui il grafo assegna un contributo al logit gap? La corrispondenza viene verificata per layer, posizione, segno atteso e gruppo semantico, annotando i gruppi senza consultare l'esito dell'intervento. È possibile che i metodi descrivano aspetti diversi dello stesso calcolo: il disaccordo è un risultato da spiegare.

### RQ3 — Quale metodo anticipa meglio l'effetto causale?

Su un insieme di interventi fissato in anticipo, quale ranking predice meglio il cambiamento reale di `g(x)` dopo activation patching o ablation? Il confronto primario valuta gli **stessi siti layer × posizione**: si aggregano gli score del Lens e quelli del grafo su tali siti, poi si esegue lo stesso patching per i siti selezionati da ogni metodo. Questo evita di confrontare impropriamente l'ablation di una feature con la sostituzione di un intero residual stream. Gli interventi specifici sulle feature del grafo o sulle direzioni J-space sono analisi secondarie, con costi ed effetti riportati separatamente.

### RQ4 — L'accordo fra i metodi aggiunge valore?

Selezionare siti proposti da entrambi produce una migliore precisione nella scoperta di effetti reali rispetto al Lens o al grafo da soli, allo stesso numero di siti? L'accordo migliora anche la stabilità fra parafrasi, seed, split e sottogruppi di esempi? Una selezione congiunta può essere più precisa ma coprire meno casi; occorre riportare entrambe le quantità.

## 6. Metodo sperimentale, passo per passo

| Fase | Operazione | Evidenza prodotta |
| --- | --- | --- |
| 0. Pre-registrazione interna | fissare prompt, target `g`, split, concetti, top-k, controlli e metriche sul development | evita di scegliere spiegazioni dopo aver visto gli effetti sul test |
| 1. Baseline | valutare TF-IDF/lineare, encoder testuale e piccolo LLM su input `P-only` | determina se l'oggetto dell'audit è credibile |
| 2. Coppie controllate | parafrasi che conservano il contenuto; negazione o cambio semantico limitato; sostituzione di dettagli neutri; controlli di lunghezza/tokenizzazione | separa sensibilità semantica da effetto di forma e lunghezza |
| 3. Jacobian Lens | leggere score per concetti prespecificati su layer e posizioni del partecipante; confrontare originale e controfattuale | mappe di contenuto verbalizzabile e ranking dei siti |
| 4. Circuit Tracer | produrre grafi per `g(x)` sugli stessi esempi; salvare grafo completo, pruning, feature ed error nodes | ranking dei siti e candidati di percorso verso la classe |
| 5. Patching comune | sostituire attivazioni degli stessi siti layer × posizione tra coppie originali/controfattuali; testare siti scelti da Lens, grafo, accordo e controlli | effetto causale comparabile sul modello originale |
| 6. Interventi mirati | ablare/patchare feature del grafo e, se supportato, scambiare direzioni J-space; usare controlli matched per layer, norma e ampiezza | verifica più fine della spiegazione di ciascun metodo |
| 7. Statistica e robustezza | bootstrap appaiato per partecipante, test di permutazione, analisi per seed e per forza dell'effetto; report dei casi nulli | incertezza e limiti di generalizzazione |

Le modifiche testuali a un'intervista reale sono **interventi sul modello**, non nuovi casi clinici. Le negazioni non preservano la label self-report; perciò si usano per verificare sensibilità e direzione delle spiegazioni, non per calcolare l'accuratezza rispetto al label originale.

Se il backend dei grafi non consente di tracciare direttamente `z_B - z_A`, si tracciano i due logits separatamente e si compone il contrasto solo dopo avere verificato che i grafi e le attribuzioni siano confrontabili. Questo dettaglio viene fissato nel gate tecnico, prima di analizzare il test.

## 7. Come misurare la fedeltà

La quantità primaria è il contrasto fra i due token di classe:

```text
g(x) = z_B(x) - z_A(x)
effect(s, x, x') = g(x con attivazione del sito s presa da x') - g(x)
```

| Misura | Significato |
| --- | --- |
| `effect@k` | effetto assoluto medio intervenendo sui primi `k` siti selezionati; sempre affiancato dal segno atteso e osservato |
| `direction accuracy` | quota di interventi il cui effetto ha il segno previsto dal metodo |
| `rank–effect association` | quanto il ranking predice l'ampiezza dell'effetto misurato su un pannello di siti prespecificato |
| `random-gap@k` | vantaggio rispetto a siti casuali matched per layer, posizione e ampiezza dell'intervento |
| `joint gain@k` | vantaggio della selezione congiunta rispetto al migliore metodo singolo a pari `k` |
| `coverage` | quota di esempi per cui un metodo trova candidati valutabili; evita di riportare solo casi favorevoli |
| `collateral effect` | variazione su prompt linguistici di controllo, per individuare interventi che danneggiano genericamente il modello |

Si riportano anche correlazione fra score e effetto, class flip, stabilità delle classifiche e intervalli di confidenza. Un cambio del logit causato da un intervento interno prova un effetto nel modello e nell'intervento scelti; non dimostra da solo che il concetto umano attribuito alla componente sia la descrizione corretta del meccanismo.

## 8. Controlli essenziali e rischi

- **Confronto equo:** ranking e patching primari usano gli stessi siti, le stesse coppie e lo stesso budget; confronto feature-level e J-space directional restano distinti.
- **Selezione circolare:** concetti, soglie e criteri di accordo sono definiti senza guardare gli effetti finali; annotatori eventualmente ciechi ai label e agli score di patching.
- **Prompt e label:** etichette `A/B` a singolo token, template invariato, controllo di tokenizzazione; il label PHQ-8 non entra mai nel testo di input.
- **Ricostruzione del grafo:** misurare error nodes, fedeltà del replacement model e sensibilità al pruning. Un grafo poco fedele limita qualunque conclusione sulla sua convergenza con il Lens.
- **Validità del Lens:** confrontare il suo readout con Logit Lens e controlli semantici; su checkpoint adattato non riusare un lens pre-fittato senza verifica.
- **Potenza statistica:** corpus piccolo e classe sbilanciata; usare bootstrap per partecipante, pochi concetti preregistrati e nessuna ricerca post-hoc del migliore caso.
- **Interpretazione sanitaria:** risultati aggregati sul comportamento del modello; nessun claim su sintomi osservati in un turno o diagnosi individuali.

## 9. Stack e fattibilità

| Funzione | Strumenti proposti |
| --- | --- |
| Dati e baseline | Python, `pandas`, scikit-learn, PyTorch, Hugging Face Transformers |
| Lettura delle attivazioni | implementazione Jacobian Lens verificata sul checkpoint scelto; Logit Lens come baseline |
| Grafi | libreria pubblica `circuit-tracer`, transcoders compatibili e salvataggio dei grafi non potati |
| Interventi | hook PyTorch/TransformerLens o backend documentato; activation patching e ablation sul modello originale |
| Baseline XAI | gradient saliency/integrated gradients e leave-one-span-out per l'input; random matched per i siti interni |
| Analisi | bootstrap appaiato, test di permutazione, notebook e configurazioni versionati; tabelle di effetti e failure cases anonimizzati |

**Esperimento minimo per una magistrale:** un checkpoint <=3B, una condizione testuale `P-only`, un target binario, 3–4 gruppi di concetti definiti prima dell'analisi, un numero limitato di coppie controfattuali bilanciate, Jacobian Lens, Circuit Tracer, patching comune su siti layer × posizione, baseline Logit Lens e random matched. Una seconda architettura, item PHQ-8, audio e nuovo corpus indipendente sono estensioni.

**Gate tecnico iniziale:** entro le prime settimane verificare con pochi esempi che (a) il checkpoint predica il target meglio del caso e delle baseline più semplici in modo non trascurabile, (b) Lens e transcoders sono utilizzabili sullo *stesso* checkpoint, (c) il replacement model riproduce sufficientemente il logit gap del modello originale e (d) il patching restituisce effetti riproducibili. Se uno di questi punti fallisce, adattare checkpoint o perimetro prima di investire nell'analisi completa.

## 10. Interpretazione degli esiti

| Esito | Conclusione consentita |
| --- | --- |
| Lens, grafo e interventi concordano | in questo modello e task, il readout verbalizzabile aiuta a trovare siti causalmente influenti sulla predizione |
| Il grafo predice gli effetti, il Lens no | alcuni concetti leggibili non sono buoni selettori di siti causali per questo target |
| Il Lens predice gli effetti, il grafo no | il readout è utile nel setup, mentre la particolare decomposizione/pruning del grafo è insufficiente |
| La selezione congiunta è più precisa ma copre pochi esempi | l'accordo è un filtro selettivo, da riportare insieme alla sua copertura |
| Nessuno supera random e baseline | i metodi studiati non forniscono evidenza di spiegazioni causali fedeli nel setup; la prestazione del modello resta una questione separata |
| Il modello non supera il gate predittivo o tecnico | limitare il claim al benchmark metodologico controllato e non attribuire meccanismi a una stima PHQ-8 non affidabile |

## 11. Sviluppo possibile per un dottorato

La tesi può produrre un **protocollo di valutazione** trasferibile. Un dottorato potrebbe applicarlo a modelli audio-testo nativi, confrontare le direzioni verbalizzabili con rappresentazioni che non lo sono, sviluppare metriche di fedeltà stabili fra architetture e testare spiegazioni sotto shift di popolazione o qualità del segnale. Ogni estensione richiede nuovi interventi e una verifica dell'accesso ai componenti interni; i circuiti trovati nel piccolo LLM testuale non si trasferiscono automaticamente.

## Riferimenti chiave

- [Gurnee et al. (2026), *Verbalizable Representations Form a Global Workspace in Language Models*](https://arxiv.org/abs/2607.15495) — definizione del Jacobian Lens e test funzionali delle rappresentazioni verbalizzabili.
- [Ameisen et al. (2025), *Circuit Tracing: Revealing Computational Graphs in Language Models*](https://www.transformer-circuits.pub/2025/attribution-graphs/methods.html) — metodo dei grafi di attribuzione, replacement model e verifica con interventi.
- [Libreria `circuit-tracer`](https://github.com/decoderesearch/circuit-tracer) e [Neuronpedia, Jacobian Lens](https://www.neuronpedia.org/blog/jacobian-lens) — disponibilità degli strumenti, da verificare per il checkpoint preciso.
- [Zhang e Nanda (2024), *Towards Best Practices of Activation Patching in Language Models*](https://arxiv.org/abs/2309.16042) — scelte di corruption, metrica e controlli.
- [Geiger et al. (2025), *Causal Abstraction for Mechanistic Interpretability*](https://www.jmlr.org/papers/volume26/23-0058/23-0058.pdf) — cornice concettuale per la fedeltà causale di astrazioni meccanicistiche.
- [Burdisso et al. (2024), *On the Validity of Using the Therapist's Prompts in DAIC-WOZ*](https://aclanthology.org/2024.clinicalnlp-1.8/) — controllo di dominio sui rischi delle domande dell'intervistatore.
