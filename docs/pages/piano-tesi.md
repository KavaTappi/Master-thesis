---
title: Piano della tesi
permalink: /
description: Tre proposte di tesi sull'audit causale di piccoli LLM e spiegazioni per testo e voce.
---

# Piano della tesi

<p class="lead">Tre percorsi alternativi per una tesi di 6–9 mesi. La 00 e la 01 hanno un benchmark controllato come nucleo completo; la 02 richiede una pipeline audio verificata.</p>

> Il PHQ-8 è un questionario self-report riferito a una sessione e a un periodo di osservazione. Le coppie sintetiche ispirate ai suoi otto domini verificano la comprensione di enunciati e le spiegazioni del modello: non sono nuovi punteggi clinici. Interviste reali, audio e label restano soggetti agli accessi e alle licenze del corpus.

## Proposte attive

| Proposta | Esperimento principale | Estensione applicativa | Rischio da verificare subito |
| --- | --- | --- | --- |
| [00 — Jacobian Lens e siti causali](../argomenti-tesi/00-from-verbalizable-concepts-to-causal-circuits.md) | coppie controllate per gli otto domini PHQ-8; J-Lens contro Logit Lens e random, validati con lo stesso activation patching | stima binaria PHQ-8 su interviste, solo dopo un gate predittivo | checkpoint e Lens compatibili; modello competente sulle coppie |
| [01 — Shortcut di protocollo](../argomenti-tesi/01-audit-meccanicistico-protocolli-intervista.md) | mini-interviste con correlazione controllata fra famiglia di prompt e label; controfattuali e patching | audit della dipendenza naturale dalle domande di Ellie in DAIC-WOZ | acquisizione del cue controllato, verificata con controllo positivo nelle prime settimane |
| [02 — Testo e voce tramite acoustic landmarks](../argomenti-tesi/02-audit-causale-landmark-acustici-llm.md) | controfattuali su contenuto/allineamento/speaker, audit del label hint e patching limitato | robustezza su un secondo corpus non sovrapposto, se disponibile | accesso all'audio, estrazione dei landmark e allineamento |

Le proposte sono alternative. La 00 studia la **fedeltà del Jacobian Lens**; la 01 la **fedeltà delle spiegazioni di uno shortcut**; la 02 l'**uso effettivo di informazione vocale codificata in token**. La proposta trimodale per otto item è stata rimossa dal piano perché troppo ampia per questo orizzonte temporale.

Una [guida trasversale alle tecniche di explainability](../tecniche-explainability-llm.md) distingue metodi comportamentali, attribution sull'input, readout rappresentazionali, interventi interni e circuit discovery, indicando compatibilità e priorità per ciascuna proposta.

## Come scegliere

| Interesse prevalente | Scelta iniziale | Perimetro di sei mesi |
| --- | --- | --- |
| Metodi di interpretabilità meccanicistica  | **00** | un checkpoint congelato, J-Lens, Logit Lens e patching sui tre domini prespecificati |
| Shortcut learning e audit del protocollo | **01** | un modello adattato, controllo 50/50 e fortemente correlato, spiegazione dell'input e patching |
| Integrazione voce–testo | **02** | un modello, un estrattore di landmark, confronto con unimodali e tre interventi sull'audio |

In termini operativi, la 01 richiede più controllo sulla costruzione del training ma meno strumenti specializzati; la 00 evita il training e concentra il rischio su Lens e patching; la 02 concentra il rischio nell'audio. Nessuna proposta richiede un risultato positivo per essere completa: serve un esperimento eseguibile, controlli credibili e un claim proporzionato agli effetti osservati.

## Modelli candidati

| Proposta | Prima scelta | Alternativa | Perché |
| --- | --- | --- | --- |
| 00 | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) + [Lens pre-fittato](https://huggingface.co/neuronpedia/jacobian-lens/tree/main/gemma-2-2b) | [Gemma 2 2B IT](https://huggingface.co/google/gemma-2-2b-it) con Lens dedicato, oppure [Gemma 3 1B base](https://huggingface.co/google/gemma-3-1b-pt) | il nucleo usa un checkpoint congelato e non richiede transcoders |
| 01 | [Qwen3 1.7B Base](https://huggingface.co/Qwen/Qwen3-1.7B-Base) | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) | LoRA e patching sul checkpoint finale; Lens e Circuit Tracer pre-fittati non sono richiesti |
| 02 | [Qwen3 1.7B Base](https://huggingface.co/Qwen/Qwen3-1.7B-Base) | [Gemma 2 2B base](https://huggingface.co/google/gemma-2-2b) | token dei landmark e adattamento rendono il patching standard più semplice degli strumenti pre-fittati |

Disponibilità dei pesi, licenze, tokenizer e memoria vanno verificati prima di bloccare il checkpoint. Un LoRA modifica il sistema sotto audit: le attivazioni si misurano sul modello adattato, e un Lens pre-fittato sul base non viene riutilizzato senza nuova validazione.

## Protocollo comune di qualità

1. Fissare **split e unità statistica** prima di costruire varianti. Per i dati reali l'unità è il partecipante; per benchmark sintetici si separano template e fonti di generazione.
2. Bloccare prompt, target, coppie, siti ammissibili, budget `k`, controlli e metriche sul development. Il test non guida la scelta delle spiegazioni.
3. Verificare che il modello svolga il compito oggetto dell'audit. I domini o gli esempi nei quali non è competente restano nel report e non alimentano claim meccanicistici.
4. Valutare gli strumenti sull'**effetto osservato di interventi comparabili** nel modello originale. Un readout o una heatmap suggerisce un'ipotesi; il patching verifica la sua influenza nel setup scelto.
5. Pubblicare protocolli, codice e statistiche aggregate nel rispetto della licenza. Non pubblicare transcript, audio o identificativi riconoscibili.

## Piano decisionale

| Periodo | 00 | 01 | 02 |
| --- | --- | --- | --- |
| Prime 2–3 settimane | caricare Lens e verificare patching su coppie | verificare che il cue controllato venga appreso | ottenere audio e provare l'estrazione dei landmark |
| Mesi 2–4 | confronto J-Lens/Logit Lens/random sul benchmark | controfattuali del protocollo e localizzazione interna | baseline, controfattuali audio e confronto con/senza label hint |
| Mesi 5–6 | robustezza, risultati nulli e scrittura | test su template tenuti fuori e scrittura | spiegazioni e patching con controlli |
| Mesi 7–9, se disponibili | pilot PHQ-8 session-level e Circuit Tracer qualitativo | audit naturale su DAIC-WOZ | verifica su dati ulteriori non sovrapposti |

La scelta fra 00 e 01 può essere presa con due piloti brevi: un esempio J-Lens più patching per la 00; una mini-intervista con cue controllato più inversione del prompt per la 01. I rispettivi file descrivono le condizioni che rendono i risultati interpretabili.
