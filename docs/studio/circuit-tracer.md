# Circuit Tracer

## Scopo

**Circuit Tracer** e' una procedura per trovare e visualizzare un grafo sparso di attribuzione interno a un modello. Il grafo collega:

- token e posizioni dell'input;
- feature sparse ottenute da transcoders;
- error nodes e componenti interne;
- logits o una funzione di logits scelta come target.

Nel caso DAIC, il target piu' semplice e' la differenza tra i logits delle classi `at_or_above_10` e `below_10`. Il risultato e' un **circuito candidato**: una spiegazione strutturale di quali componenti hanno contribuito alla decisione del modello.

## Perche' tracciare una differenza di logits

Un LLM produce, nella posizione di output, un vettore di logits `z(x)`, uno per token del vocabolario. I logits sono valori non normalizzati: la probabilita' di un token `v` e' data dalla softmax

```text
p(v | x) = exp(z_v(x)) / sum_j exp(z_j(x))
```

Per una classificazione binaria e' piu' informativo definire un contrasto tra due classi che tracciare il token piu' probabile. Se `A` codifica `PHQ8 < 10` e `B` codifica `PHQ8 >= 10`, il target e':

```text
g(x) = z_B(x) - z_A(x)
```

Questa scelta ha tre proprieta' utili:

1. **E' una misura di evidenza relativa.** Vale esattamente

   ```text
   g(x) = log( p(B | x) / p(A | x) )
   ```

   quindi un `g` positivo favorisce B rispetto ad A, uno negativo favorisce A. La normalizzazione della softmax si cancella.
2. **Ha un segno interpretabile.** Ogni nodo e ogni edge del grafo puo' spingere `g` verso B, spingerlo verso A oppure non avere effetto. Tracciare solo `z_B` confonderebbe evidenza per B con componenti che alzano entrambi i logits.
3. **E' allineata alla decisione.** Se il classificatore sceglie la classe con logit maggiore, il confine decisionale e' `g(x)=0`. Un intervento che cambia il segno di `g` e' un class flip misurabile.

In pratica, i token testuali come `below_10` possono essere spezzati dal tokenizer in piu' token. Per un audit rigoroso usare un output a **singolo token**, ad esempio `A` e `B`, e dichiarare nel prompt la mappa `A = PHQ8 < 10`, `B = PHQ8 >= 10`. Solo dopo si puo' rendere il risultato leggibile in linguaggio naturale. Se si tracciano etichette multi-token, il target deve diventare una somma di log-probabilita' condizionate su piu' posizioni, che e' piu' complessa e meno pulita.

### Esempio DAIC

Supponiamo due versioni della stessa risposta del partecipante: una originale e una in cui la domanda di Ellie e' stata permutata. Il modello produce:

```text
g(originale) = +2.4    -> odds B:A circa 11:1
g(domanda permutata) = +0.1
```

La predizione e' ancora B, ma il margine e' quasi scomparso. Circuit Tracer consente di chiedere se la perdita di `2.3` proviene da feature che codificano la domanda dell'intervistatore, da contenuto della risposta, oppure da entrambi. Questo e' molto piu' informativo di dire soltanto che la probabilita' e' cambiata.

## Da attivazioni dense a feature interpretabili

Un neurone singolo e' spesso polisemantico: puo' contribuire a concetti diversi in contesti diversi. Circuit Tracer usa quindi un **transcoder**, un dizionario sparso che approssima l'output MLP a partire dal suo input.

Per il layer `l`, in forma semplificata:

```text
h_l          = attivazione densa in ingresso al MLP
a_l = W_enc h_l + b_enc
s_l = f(a_l) = feature sparse, non negative
m_hat_l = W_dec s_l + b_dec = output MLP ricostruito
```

Dove:

- `h_l` e `m_hat_l` sono vettori densi nel residual stream;
- `s_l,i` e' l'attivazione della feature sparsa `i`;
- `f` e' in genere ReLU, JumpReLU o Top-k, quindi poche feature sono attive;
- `W_dec[:, i]` e' la direzione che la feature `i` scrive nell'output MLP.

Un transcoder non rende magicamente una feature "un concetto clinico". La sua descrizione viene inferita dai massimi esempi di attivazione e dai token che tende ad aumentare o diminuire. La sua utilita' e' rendere il calcolo abbastanza sparso da poter attribuire un percorso.

## Come viene costruito il grafo

Per un prompt `x` e un target scalare `g(x)`, il grafo contiene nodi di quattro tipi:

```text
token/input embedding -> feature transcoder -> feature downstream -> logit target
                                  \-> error node ----------->/
```

Gli **error nodes** rappresentano la parte dell'output MLP che il transcoder non ricostruisce. Sono necessari per fedelta', ma se dominano il grafo ne limitano l'interpretabilita'.

Concettualmente, l'effetto totale di una feature candidata `i` puo' essere definito come intervento:

```text
TE_i(x) = g(x) - g(x ; s_i <- 0)
```

Un grande `TE_i` indica che spegnere la feature cambia il contrasto di classe, ma calcolare questa quantita' con un forward pass per ogni feature e' costoso. Circuit Tracer calcola invece effetti diretti per gli edge del grafo. Condizionando sulle non-linearita' osservate del forward pass - ad esempio pattern di attenzione e fattori di LayerNorm - le pre-attivazioni delle feature diventano localmente funzioni lineari dei nodi precedenti. In questa linearizzazione condizionata, l'effetto diretto di un nodo sorgente `u` su un nodo target `v` e' del tipo:

```text
DE(u -> v | x) = coefficiente_lineare(u, v, x) * attivazione(u, x)
```

Il coefficiente viene ottenuto con una backward pass dal nodo target, fermando il gradiente attraverso le non-linearita' su cui si condiziona. Ripetendo il procedimento per i nodi tenuti, si ottiene una matrice di adiacenza con effetti diretti e segno. Il grafo visualizzato e' la versione potata di questa matrice, non l'intero calcolo del Transformer.

Questa e' una stima piu' strutturata del semplice `gradient x activation`, ma resta condizionata al forward pass osservato. Per questo la verifica con interventi reali e' indispensabile.

## Cosa significa leggere un edge

Se il grafo contiene un edge positivo:

```text
feature "sleep-related language"  -- +0.35 -->  logit gap g
```

la lettura corretta e': *in questo prompt, entro la decomposizione del transcoder, l'attivazione di questa feature contribuisce +0.35 al contrasto B contro A attraverso quell'edge o percorso*. Non significa: *il partecipante ha un disturbo del sonno*.

Un edge negativo riduce `g`, cioe' spinge relativamente verso A. Un circuito puo' contenere sia evidenza positiva sia contro-evidenza: e' proprio il motivo per cui il contrasto tra logits e' preferibile a una generica "importanza" positiva.

## Dall'attribuzione alla prova causale

Il flusso corretto ha tre livelli:

| Livello | Operazione | Cosa si puo' affermare |
|---|---|---|
| Attribuzione | Trovare nodi/edge con grande effetto diretto o totale stimato. | Sono candidati importanti per questo calcolo. |
| Intervento mirato | `s_i <- 0`, `s_i <- valore`, activation patching da un caso sorgente. | La feature puo' causare un cambiamento nel target in questo setting. |
| Replica e controlli | Ripetere su esempi, seed, split e feature abbinate. | Il meccanismo e' stabile e selettivo, non un artefatto locale. |

Per DAIC, il test piu' informativo e' comparare tre interventi:

1. ablation di una feature candidata del circuito;
2. ablation di una feature di controllo con stesso layer e attivazione simile;
3. ablation di una feature casuale.

Se solo il primo riduce in modo consistente `g(x)` sui casi target senza degradare un target linguistico non correlato, l'evidenza e' piu' forte. Il patching e' complementare: si sostituisce l'attivazione della feature o del residual stream con quella di un esempio matched e si verifica se `g` si sposta nella direzione predetta.

## Perche' e' utile specificamente nella tesi

Tracciare `g=z_B-z_A` permette di separare quattro spiegazioni possibili per una buona metrica di classificazione:

| Possibile meccanismo | Cosa mostrerebbe il grafo | Controllo da fare |
|---|---|---|
| Contenuto della risposta | Token del partecipante e feature semanticamente coerenti contribuiscono a `g`. | Permutare la domanda e ablate le feature. |
| Shortcut dell'intervista | Token/feature di Ellie, posizione o lunghezza contribuiscono molto a `g`. | `E-only`, `P-only`, domanda permutata, length matching. |
| Keyword superficiali | Poche parole esplicite dominano il circuito. | Mascheramento/parafrasi controllata e verifica della selettivita'. |
| Rumore del transcoder | Error nodes o feature instabili dominano. | Ridurre le conclusioni; cambiare dizionario o riportare limite. |

Il contributo scientifico non e' trovare un grafo esteticamente convincente. E' misurare quale di queste ipotesi e' supportata da effetti con segno, interventi e replica.

## Limiti matematici e pratici

- Il contrasto `g` riguarda la decisione del modello, non la verita' clinica del label.
- Il grafo e' condizionato a uno specifico prompt e forward pass; circuiti diversi possono essere validi per esempi diversi.
- Le non-linearita' di attenzione e LayerNorm sono fissate nel calcolo degli effetti diretti; il grafo non cattura automaticamente cosa avverrebbe se esse cambiassero molto dopo un intervento.
- I transcoders ricostruiscono in modo approssimato gli MLP. Molto flusso attraverso error nodes richiede prudenza.
- La potatura rende il grafo leggibile ma puo' eliminare percorsi piccoli e cumulativamente importanti. Salvare sempre il grafo completo e le soglie.

## Cosa non e'

Un grafo Circuit Tracer non e' di per se' una prova di causalita', ne' una spiegazione clinica. L'attribuzione seleziona componenti potenzialmente importanti; la prova viene dopo, con interventi controllati sulla feature o sul percorso indicato dal grafo.

## Perche' usare una differenza di logits

Per una classe binaria si puo' definire:

```text
target = logit(at_or_above_10) - logit(below_10)
```

Tracciare questo target e' piu' preciso che tracciare genericamente il token piu' probabile: ogni nodo/edge e' valutato rispetto alla decisione che interessa alla tesi. Una regressione continua del punteggio PHQ-8 resta utile come baseline, ma e' meno immediata da investigare con un grafo di next-token attribution.

## Prerequisiti tecnici

Circuit Tracer richiede:

- un modello open-weight compatibile con il backend scelto;
- un transcoder o un insieme di transcoders compatibili per i layer MLP del modello;
- GPU sufficiente per modello e transcoders;
- una versione precisa di pesi, tokenizer, transcoders, libreria e configurazione di pruning.

La disponibilita' dei modelli e dei transcoders evolve rapidamente. Va verificata al momento dell'implementazione; non progettare la tesi assumendo che un qualunque LLM fine-tuned sia automaticamente tracciabile. Un adapter LoRA puo' cambiare il comportamento e ridurre la validita' interpretativa di transcoders addestrati sul modello base.

Neuronpedia offre un'interfaccia per generare/esplorare grafi per modelli supportati e per caricare grafi propri. Per risultati di tesi, conservare anche JSON del grafo, prompt, seed, soglie, versione dei pesi e risultati numerici delle verifiche causali.

## Protocollo per DAIC

1. **Formulare il target.** Iniziare con classificazione binaria PHQ-8 >= 10 oppure con categorie ordinali; non usare una risposta libera non vincolata.
2. **Fissare gli esempi.** Campionare casi bilanciati, includendo predizioni corrette e sbagliate, domande condivise e risposte con lunghezza comparabile.
3. **Tracciare il target.** Generare grafi per il logit gap, fissando in anticipo la procedura di pruning. Salvare il grafo intero prima della visualizzazione potata.
4. **Annotare senza circolarita'.** Descrivere una feature con massimi esempi di attivazione e token upweighted, senza guardare il label del partecipante. Usare categorie: contenuto sintomatologico, domanda/protocollo, stile, non interpretabile.
5. **Aggregare, non selezionare.** Misurare ricorrenza e peso di feature/edge su molti esempi; un grafo individuale e' solo uno studio di caso.
6. **Intervenire.** Eseguire feature ablation, activation patching e, se giustificato, insertion. Confrontare l'effetto sul logit gap con feature casuali o abbinate per layer e frequenza di attivazione.
7. **Testare selettivita'.** Verificare che l'intervento modifichi la decisione di PHQ-8 piu' di un target linguistico di controllo; altrimenti potrebbe semplicemente danneggiare il modello.

## Metriche consigliate

| Proprietà | Misura |
|---|---|
| Fedelta' causale | delta-logit e class flip rate dopo ablation/patching |
| Selettivita' | effetto sul target PHQ-8 meno effetto su target di controllo |
| Stabilita' | overlap pesato tra feature/edge su seed, split e casi comparabili |
| Specificita' semantica | quota di feature annotate come sintomatologiche rispetto a protocollo/scorciatoia |
| Robustezza | variazione del circuito dopo permutazione domanda e normalizzazione lunghezza |

## Multimodalita': come integrarla davvero

Circuit Tracer puo' analizzare un modello multimodale soltanto se le componenti rilevanti sono tracciabili dal backend e coperte da un dizionario sparse/transcoder. Questo porta a tre configurazioni diverse:

| Configurazione | Cosa puo' tracciare | Limite |
|---|---|---|
| Testo + token acustici discreti | token del transcript e token/proxy acustici nel circuito LLM | non il segnale audio grezzo |
| Encoder audio/video + LLM multimodale | in principio token multimodali, proiettore e LLM | richiede supporto esplicito e transcoders per l'architettura |
| Modello di fusione tardiva | ciascun ramo separatamente, se strumentato | un Circuit Tracer del solo LLM non spiega la fusione |

La prima configurazione e' la piu' realistica. Esempio: la pipeline acustica estrae pause, F0, intensita', jitter e COVAREP; li discretizza con soglie fissate sul training set; il LLM riceve sia la trascrizione sia token descrittivi. Il circuito puo' allora mostrare se `LONG_PAUSE` contribuisce al target, se interagisce con contenuto testuale e se l'effetto sopravvive al controllo per durata della risposta.

Per fare affermazioni sul vero audio/video encoder, servirebbe estendere la strumentazione al suo spazio latente e al proiettore multimodale. Questa e' una possibile estensione di dottorato o, in una magistrale, un capitolo limitato con un'architettura piccola e completamente riproducibile.

## Esperimento minimo raccomandato

- Modello piccolo, text-only e supportato; target binario PHQ-8.
- 50-100 esempi bilanciati e un piccolo insieme di domande comuni.
- Grafi per il logit gap, feature aggregate e ablation con controlli abbinati.
- Un secondo esperimento con token acustici interpretabili, non con audio grezzo.
- Conclusione limitata a: "il modello usa/non usa questi proxy nel circuito per questa predizione", non a inferenze cliniche.

## Riferimenti essenziali

- Hanna e Piotrowski, *circuit-tracer: A Library for Finding Feature Circuits in Language Models*: [paper](https://openreview.net/pdf/3a81db93b4b3d6710d495dee99f516f70e6e2de2.pdf).
- Neuronpedia, *Circuit Tracer*: [documentazione/annuncio](https://www.neuronpedia.org/blog/circuit-tracer).
