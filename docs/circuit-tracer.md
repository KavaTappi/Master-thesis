# Circuit Tracer

## Scopo

**Circuit Tracer** e' una procedura per trovare e visualizzare un grafo sparso di attribuzione interno a un modello. Il grafo collega:

- token e posizioni dell'input;
- feature sparse ottenute da transcoders;
- error nodes e componenti interne;
- logits o una funzione di logits scelta come target.

Nel caso DAIC, il target piu' semplice e' la differenza tra i logits delle classi `at_or_above_10` e `below_10`. Il risultato e' un **circuito candidato**: una spiegazione strutturale di quali componenti hanno contribuito alla decisione del modello.

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
