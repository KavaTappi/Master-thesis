---
title: Piano della tesi
permalink: /
description: Quattro proposte di tesi sull'audit causale di piccoli LLM per la stima sperimentale del PHQ-8.
---

# Piano della tesi

<p class="lead">Quattro percorsi alternativi, dallo studio dei metodi di interpretabilità all'audit di testo, voce e volto. La scelta finale dipende da accesso ai dati, compatibilità degli strumenti e risorse di calcolo.</p>

> Il PHQ-8 qui è un target self-report **a livello di sessione**, non una diagnosi. Le analisi spiegano il comportamento di un modello: non attribuiscono uno stato clinico a un singolo turno, una persona o una feature interna. Audio, video e trascrizioni devono rispettare le licenze del corpus e non comparire in artefatti pubblici riconoscibili.

## Le quattro proposte

| Proposta | Domanda principale | Evidenza decisiva | Dipendenza principale | Perimetro minimo |
| --- | --- | --- | --- | --- |
| [00 — From verbalizable concepts to causal circuits](../argomenti-tesi/00-from-verbalizable-concepts-to-causal-circuits.md) | I concetti letti dal Jacobian Lens e i grafi di Circuit Tracer individuano siti che influenzano davvero il logit PHQ-8? | Il ranking di ciascun metodo, e del loro accordo, anticipa l'effetto di **uno stesso activation patching** sul modello originale | Lens e transcoders compatibili con il medesimo checkpoint; prestazione del modello non banale | un LLM testuale, `P-only`, una classe binaria, pochi concetti prespecificati |
| [01 — Audit meccanicistico di shortcut del protocollo](../argomenti-tesi/01-audit-meccanicistico-protocolli-intervista.md) | Un LLM usa le risposte o segnali della struttura dell'intervista per predire il target? | Controlli `P-only`/`E-only`/`E+P` e controfattuali del protocollo mostrano una dipendenza; solo allora si cercano componenti interne causalmente valide | dimostrare **prima** che lo shortcut esiste nel LLM studiato | audit comportamentale con possibilità di approfondimento meccanicistico |
| [02 — Audit causale testo-voce con acoustic landmarks](../argomenti-tesi/02-audit-causale-landmark-acustici-llm.md) | I landmark acustici allineati al testo aggiungono informazione realmente usata dal piccolo LLM? | Il vantaggio della fusione resiste a controlli di lunghezza/qualità e cala con shuffle, mask o swap matched della voce | accesso ad audio e allineamento; costruzione riproducibile dei landmark | testo, landmark, controllo di allineamento, un output binario |
| [03 — Meccanismi multimodali specifici per item](../argomenti-tesi/03-meccanismi-multimodali-item-phq8-llm.md) | Voce e volto contribuiscono a item PHQ-8 diversi, e le spiegazioni prevedono effetti selettivi? | Mascheramenti, swap e interventi interni alterano l'item bersaglio più degli altri, superando controlli di qualità | disponibilità di score per item, segnali audio/video e potenza statistica sufficiente | pochi item prespecificati come studio pilota, poi estensione agli otto |

Le quattro proposte sono **alternative**, non capitoli obbligatori di una sola tesi. La 00 valuta soprattutto la *fedeltà degli strumenti*; la 01 un possibile *shortcut del protocollo*; la 02 l'*uso causale della voce* codificata in token; la 03 la *selettività per item* della fusione testo–voce–volto. Nelle 02 e 03 il modello principale resta un **LLM testuale** che legge rappresentazioni multimodali strutturate; una variante nativamente multimodale in 03 è facoltativa e richiede un audit diverso.

## Modelli candidati per la proposta 00

I cataloghi pubblici mostrano sia un Jacobian Lens pre-fittato sia transcoders utilizzabili da Circuit Tracer per i seguenti checkpoint sotto 3B. È una verifica di **disponibilità**, non ancora una prova che la pipeline funzioni sulle interviste o che il modello predica bene il PHQ-8.

| Priorità | Checkpoint | Perché considerarlo | Verifica ancora necessaria |
| --- | --- | --- | --- |
| 1 | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) | [Lens pubblicato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b), [transcoders](https://huggingface.co/mntss/gemma-scope-transcoders) e [demo di tracing/intervento](https://github.com/decoderesearch/circuit-tracer); è la scelta iniziale più documentata | accesso alla licenza Gemma, identità dei pesi/tokenizer, output di classe e fedeltà del replacement model |
| 2 | [Qwen3 1.7B](https://huggingface.co/Qwen/Qwen3-1.7B) | [Lens pubblicato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/qwen3-1.7b) e [PLT pubblicati](https://huggingface.co/mwhanna/qwen3-1.7b-transcoders-lowl0); candidato a replica fra architetture | compatibilità effettiva del backend e costo del tracing sui prompt scelti |
| 3 | [Gemma 3 1B base](https://huggingface.co/google/gemma-3-1b-pt) | [Lens pubblicato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-3-1b) e [transcoders GemmaScope 2](https://huggingface.co/collections/mwhanna/gemma-scope-2-transcoders-circuit-tracer); più leggero | Circuit Tracer richiede `nnsight`, descritto come sperimentale; verificare capacità predittiva, probabilmente più fragile |

[Gemma 2 2B-IT](https://huggingface.co/google/gemma-2-2b-it) ha un [Lens dedicato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b-it) e una demo di Circuit Tracer, ma quest'ultima riusa transcoders del modello base: trattarlo come **variante da validare**, non come equivalente già dimostrato. Gemma 3 1B-IT dispone invece di [Lens](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-3-1b-it) e [transcoders IT](https://huggingface.co/collections/mwhanna/gemma-scope-2-transcoders-circuit-tracer), con la stessa cautela sul backend `nnsight`. L'elenco e le limitazioni dei backend provengono dalla [documentazione di Circuit Tracer](https://github.com/decoderesearch/circuit-tracer); l'[implementazione di riferimento del Jacobian Lens](https://github.com/anthropics/jacobian-lens) permette anche di rifittare un Lens, ma ciò aumenta il lavoro sperimentale.

**Raccomandazione operativa:** provare per primo Gemma 2 2B base su pochi prompt brevi; passare a Qwen3 1.7B solo dopo un test completo di Lens, grafo e patching. Un modello instruction-tuned può classificare meglio, ma ogni adattamento o LoRA cambia il checkpoint e impone di ricontrollare Lens e transcoders. Nessuna di queste risorse dimostra da sola accuratezza clinica.

## Come scegliere fra le quattro

| Se l'interesse principale è… | Prima scelta | Condizione di avvio |
| --- | --- | --- |
| confrontare rigorosamente due metodi meccanicistici e costruire un protocollo estendibile al dottorato | **00** | gate tecnico sullo stesso checkpoint e target predittivo credibile |
| capire se un LLM sfrutta una scorciatoia dell'intervista | **01** | dipendenza comportamentale riproducibile da prompt/struttura; senza di essa la parte circuitale non è giustificata |
| studiare l'integrazione voce–testo con interventi relativamente chiari | **02** | accesso all'audio, qualità dell'allineamento e baseline `text-only` solida |
| studiare spiegazioni multimodali fini, una per item | **03** | score per item e segnali facciali/audio disponibili, con campione sufficiente per otto esiti; è la più ampia |

Non esiste una graduatoria assoluta di fattibilità: **00** riduce il lavoro sui dati ma aumenta quello sugli strumenti; **02** ha interventi naturali ma dipende dall'audio; **01** ha un gate empirico sullo shortcut; **03** concentra il maggior numero di modalità e target. Per una magistrale conviene definire un esperimento minimo indipendente per ogni opzione e lasciare le repliche a un possibile dottorato.

## Protocollo comune di qualità

1. **Dati e accessi.** Verificare licenza e asset realmente disponibili di DAIC-WOZ/E-DAIC prima di fissare il disegno. DAIC-WOZ ed E-DAIC possono contenere sessioni sovrapposte: non sono automaticamente due test indipendenti.
2. **Unità di analisi.** Fissare split per partecipante prima di creare finestre o turni. Il target PHQ-8 è session-level; modificare un turno per un test controfattuale non crea una nuova label clinica.
3. **Baseline.** Confrontare sempre il piccolo LLM con TF-IDF + classificatore lineare e una baseline encoder o di fusione semplice pertinente alla proposta. Riportare balanced accuracy, macro-F1, AUROC/AUPRC e calibrazione, con incertezza per partecipante.
4. **Input e lunghezza.** Definire una regola di segmentazione/aggregazione sul development: le interviste possono superare la finestra utile del modello. Evitare che troncamento, numero di turni o token di qualità diventino spiegazioni spurie.
5. **Spiegazioni e interventi.** Prespecificare target, budget, controlli matched e metrica primaria. Valutare i metodi sul **cambiamento osservato** nel modello originale; saliency, Lens e grafi restano ipotesi finché non anticipano interventi controllati.
6. **Risultati nulli.** Riportare anche quando il modello non supera le baseline, la modalità non aggiunge valore, lo shortcut non si manifesta o gli strumenti non superano il random. Ogni esito limita il claim ma può rispondere a una RQ.
7. **Privacy e interpretazione.** Pubblicare solo codice, protocolli e statistiche aggregate autorizzate. Nessun transcript, audio, frame, ID o esempio riconoscibile; niente indicazioni diagnostiche individuali.

## Piano di lavoro decisionale

| Fase | Deliverable | Decisione |
| --- | --- | --- |
| 1. Scelta e accessi | una proposta primaria, asset del corpus, licenze, capacità GPU e target definiti | confermare o ridurre il perimetro |
| 2. Pilota predittivo | split per partecipante, baseline semplici, un piccolo LLM e intervalli di incertezza | esiste un comportamento abbastanza solido da auditare? |
| 3. Gate specifico | 00: compatibilità Lens/Tracer; 01: shortcut comportamentale; 02: landmark/allineamento; 03: score per item e qualità multimodale | procedere, cambiare variante o fermarsi al risultato pilota |
| 4. Audit preregistrato | coppie controfattuali, ranking delle spiegazioni, patching/mascheramenti e controlli | le spiegazioni anticipano effetti selettivi? |
| 5. Robustezza e scrittura | bootstrap per partecipante, failure cases anonimizzati, limiti e protocollo riproducibile | claim finale proporzionato all'evidenza |

## Prossimo passo

Scegliere la proposta primaria con un pilota breve, non sulla sola promessa teorica: verificare prima dati, checkpoint e un intervento rappresentativo. Registrare la scelta e i risultati nel [diario delle settimane]({{ '/settimane/' | relative_url }}).
