---
title: Piano della tesi
permalink: /
description: Obiettivi, fasi e diario di avanzamento della tesi.
---

# Piano della tesi

<p class="lead">Uno spazio di lavoro pubblico per rendere visibili obiettivi, decisioni e avanzamenti settimana per settimana.</p>

> Aggiorna questa pagina quando cambiano obiettivi, tempi o perimetro del lavoro. I dettagli operativi vivono nel [diario settimanale]({{ '/settimane/' | relative_url }}).

## Obiettivo

_Descrivi qui in poche righe il problema affrontato, il contributo atteso e a chi può essere utile._

## Domande di ricerca

Le domande seguenti sono proposte alternative e combinabili. Il filo comune è distinguere la **validità formale** dell'output (una scelta appartiene alle opzioni consentite) dalla sua **affidabilità epistemica** (la scelta è corretta, stabile e accompagnata da un'incertezza utilizzabile). I modelli di decisione tipizzata, come Jev e le alternative open source, costituiscono il caso di studio; il piano sperimentale deve comunque includere almeno un classificatore/LLM convenzionale come baseline.

### RQ1 — Le probabilità restituite da un modello decisionale tipizzato sono calibrate e utili per decidere quando automatizzare?

**A cosa risponde.** Una probabilità pari a 0,90 dovrebbe corrispondere, su molti casi analoghi, a circa il 90% di decisioni corrette. La domanda verifica se la confidence può essere usata come segnale operativo per: eseguire automaticamente una decisione, richiedere revisione umana o inoltrare il caso a un modello più capace. Risponde quindi a una lacuna dell'explainability pratica: non chiede solo *che cosa* il modello scelga, ma se dichiari correttamente *quanto è affidabile* la propria scelta.

**Parte quantitativa.** Su uno o più dataset con etichette umane (ad esempio classificazione di ticket, intent detection, policy compliance o documenti), confrontare modelli tipizzati e baseline con:

- accuratezza, macro-F1 e log-loss;
- Expected Calibration Error (ECE), Brier score e reliability diagram, prima e dopo un'eventuale calibrazione della temperatura;
- curve rischio-copertura / selective risk: errore residuo quando il sistema si astiene nei casi sotto una soglia di confidence;
- costo e latenza medi per decisione, per misurare il compromesso tra qualità, velocità e quota di casi inviati alla revisione.

L'analisi dovrebbe riportare intervalli di confidenza bootstrap al 95% e confronti appaiati tra modelli (ad esempio McNemar per l'accuratezza; bootstrap sulle differenze per ECE e Brier). Il risultato atteso non è necessariamente che un modello sia “migliore”, ma identificare soglie di confidence che mantengano un rischio massimo esplicito.

### RQ2 — Quanto sono robuste decisione e confidence rispetto a variazioni semanticamente irrilevanti dell'input e delle opzioni?

**A cosa risponde.** Un sistema spiegabile deve produrre decisioni coerenti quando il significato non cambia. Questa domanda verifica se il modello è sensibile a fattori accidentali: ordine delle alternative, loro denominazione, parafrasi del testo, informazioni ridondanti o stile linguistico. È particolarmente importante per i modelli “type-safe”: il tipo può essere corretto pur selezionando un'opzione diversa per motivi non giustificabili.

**Parte quantitativa.** Costruire per ciascun esempio un insieme di perturbazioni controllate: permutazione delle label, parafrasi preservando il significato, rimozione/aggiunta di testo irrilevante e rinomina delle opzioni senza cambiarne la definizione. Misurare:

- tasso di invariance: percentuale di casi in cui la decisione resta uguale dopo una perturbazione semanticamente neutra;
- variazione assoluta media della confidence e divergenza Jensen-Shannon tra le distribuzioni sulle opzioni;
- performance sulle versioni originali e perturbate, con relativo degrado percentuale;
- tasso di inversione della decisione per tipo di perturbazione e per numero di classi.

Un disegno appaiato consente test di McNemar sulle decisioni e test di permutazione/bootstrap sulla variazione di confidence. L'output della ricerca è una *robustness profile* che indica non soltanto se il modello sbaglia, ma in quali condizioni la sua spiegazione probabilistica perde stabilità.

### RQ3 — Un sistema ibrido “decisione + evidenze + spiegazione” produce spiegazioni più fedeli e più utili di una spiegazione generata direttamente da un LLM?

**A cosa risponde.** La scelta tipizzata e la sua confidence non equivalgono a una motivazione leggibile. La domanda valuta se una pipeline che (i) prende una decisione, (ii) recupera o seleziona evidenze nel documento e (iii) genera una spiegazione vincolata a tali evidenze sia più auditabile di un LLM che genera decisione e spiegazione nello stesso passaggio. Risponde direttamente al problema delle spiegazioni persuasive ma non fedeli.

**Parte quantitativa.** Preparare un campione annotato con decisione corretta ed evidenze/rationale di riferimento. Confrontare almeno: LLM con spiegazione libera, LLM con output strutturato e pipeline ibrida. Valutare:

- qualità della decisione: accuracy e macro-F1;
- fedeltà dell'evidenza: precision, recall e F1 rispetto ai passaggi annotati; sufficiency e comprehensiveness, cioè quanto la predizione cambia mantenendo o rimuovendo l'evidenza proposta;
- factual consistency / citation entailment: quota di affermazioni della spiegazione supportate dall'evidenza citata;
- utilità per l'utente: valutazione cieca di annotatori su chiarezza, azionabilità e fiducia appropriata, con accordo inter-annotatore (Cohen's kappa o Krippendorff's alpha).

Le differenze possono essere stimate con bootstrap stratificato e, per i giudizi umani ripetuti, con un modello a effetti misti che separi l'effetto del metodo da quello del documento e dell'annotatore.

### RQ4 — Quando conviene delegare a un modello generativo o a una revisione umana invece di fidarsi della decisione tipizzata?

**A cosa risponde.** Questa domanda trasforma l'incertezza in una politica di controllo. Vuole stabilire se una regola semplice basata su confidence, margine tra prima e seconda opzione e segnali di fuori-distribuzione possa ridurre gli errori senza annullare il vantaggio di costo e latenza. Il contributo è una strategia di *human/LLM-in-the-loop* verificabile, non una promessa generica di sicurezza.

**Parte quantitativa.** Simulare o implementare politiche di routing: soglia di confidence, soglia sul margine, detector out-of-distribution e combinazioni di esse. Per ogni politica, riportare:

- accuracy finale e tasso di errore sui casi automatizzati;
- coverage: percentuale di casi risolti dal modello veloce senza escalation;
- tasso di escalation a revisore/modello generativo, costo atteso e latenza end-to-end;
- area sotto la curva rischio-copertura e costo per decisione corretta;
- risultati separati per sottogruppi rilevanti del dominio, per verificare che l'automazione non concentri gli errori su una categoria.

La politica dovrebbe essere scelta su un validation set e valutata una sola volta su un test set tenuto separato. Un'analisi di sensibilità sulle soglie rende esplicito il trade-off e permette di proporre una configurazione adeguata a un rischio target (ad esempio errore inferiore al 5% sui casi automatizzati).

### Possibile domanda principale e perimetro consigliato

Una formulazione compatta e realistica è: **“In che misura i modelli di decisione tipizzata forniscono segnali di incertezza calibrati e spiegazioni verificabili per l'automazione selettiva di decisioni testuali?”**

Per una tesi magistrale il perimetro più solido è RQ1 + RQ2, con RQ4 come dimostrazione applicativa. RQ3 è molto interessante ma richiede un dataset con rationale/evidenze annotate e una valutazione umana: conviene includerla solo se tempo e disponibilità di annotatori lo consentono.

## Piano di lavoro

| Fase | Risultato atteso | Stato |
| --- | --- | --- |
| Definizione del problema | Obiettivi, domande di ricerca e perimetro | Da definire |
| Revisione della letteratura | Fonti selezionate e sintesi critica | Da definire |
| Progettazione della metodologia | Metodo, dati e criteri di valutazione | Da definire |
| Sviluppo / analisi | Esperimenti, prototipo o analisi completati | Da definire |
| Scrittura e revisione | Bozza, feedback e versione finale | Da definire |

## Prossimo passo

Completa le sezioni precedenti e registra il primo avanzamento nel [diario delle settimane]({{ '/settimane/' | relative_url }}).
