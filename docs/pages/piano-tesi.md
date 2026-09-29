---
title: Piano della tesi
permalink: /
description: Possibili tesi, domande di ricerca e piano di lavoro.
---

# Piano della tesi

<p class="lead">Uno spazio di lavoro per scegliere il perimetro della tesi e rendere visibili decisioni, esperimenti e avanzamenti.</p>

> Il progetto riguarda modelli su dati sensibili di salute mentale. Ogni risultato va interpretato come audit di un modello o supporto sperimentale allo screening: il PHQ-8 non è una diagnosi clinica e le spiegazioni interne non descrivono stati mentali delle persone.

## Obiettivo comune

Studiare se piccoli modelli a pesi aperti possono stimare un target di screening da interviste DAIC-WOZ in modo **spiegabile, causale e robusto**. Il contributo non è soltanto una metrica predittiva: occorre verificare quali segnali usa il modello, se gli interventi sulle sue rappresentazioni cambiano davvero la decisione e se il comportamento sopravvive a controlli contro confondenti.

I modelli devono restare entro 3B parametri. Il punto di partenza più solido è Gemma 2 2B, con Qwen3 1.7B come replica; entrambi dispongono di strumenti pubblici per Jacobian Lens e Circuit Tracer. Le baseline restano necessarie: TF-IDF + modello lineare e almeno un encoder Transformer.

## Possibili tesi

| # | Argomento | Domanda centrale | RQ da risolvere | Fattibilità |
| --- | --- | --- | --- | --- |
| 1 | [Circuiti causali per PHQ-8](../argomenti-tesi/01-circuiti-causali-phq8.md) | Il modello usa contenuto delle risposte o scorciatoie del protocollo? | segnale predittivo; circuiti; causalità degli interventi | Medio-alta |
| 2 | [Shortcut della struttura d'intervista](../argomenti-tesi/02-shortcut-struttura-intervista.md) | Quanto predicono domanda, lunghezza e struttura invece della risposta? | contributo di domanda/risposta; invarianza; circuiti; generalizzazione a domande nuove | Alta — consigliata |
| 3 | [Rappresentazioni latenti dei domini PHQ-8](../argomenti-tesi/03-rappresentazioni-latenti-domini-phq8.md) | Il modello separa concetti relativi ai diversi item PHQ-8? | predizione per item; separazione; causalità; confronto XAI | Media |
| 4 | [Spiegazioni multimodali testo-voce](../argomenti-tesi/04-spiegazioni-multimodali-testo-voce.md) | Voce e testo offrono informazione complementare e verificabile? | guadagno audio; dominanza di modalità; interazioni causali | Media-bassa |
| 5 | [Stabilità sotto domain shift](../argomenti-tesi/05-stabilita-spiegazioni-domain-shift.md) | Le spiegazioni restano valide cambiando gruppo, protocollo o dominio? | degradazione; trasferimento J-space; riuso di circuiti; shortcut di community | Media |

## RQ della tesi 1 — Circuiti causali per la stima PHQ-8

**RQ1. Esiste un segnale predittivo testuale riproducibile?** Confrontare baseline lineari, encoder e decoder su `P-only`, con split per partecipante. Risponde se il problema è sufficientemente identificabile prima di interpretare il modello.

**RQ2. Quali feature e circuiti contribuiscono alla classe?** Leggere i concetti nei layer intermedi con Jacobian Lens e tracciare, con Circuit Tracer, il contrasto `logit(at_or_above_10) - logit(below_10)`. Risponde a *quale calcolo interno* sostiene la decisione.

**RQ3. Le feature trovate sono causalmente rilevanti?** Ablation e patching devono produrre un `delta-logit` selettivo, maggiore di feature di controllo. Risponde se il grafo è una spiegazione fedele oppure una sola attribuzione correlazionale.

Metriche: balanced accuracy, macro-F1, AUROC/AUPRC, MAE/RMSE per lo score continuo, overlap di feature/edge, `delta-logit`, class flip rate e intervalli bootstrap.

## RQ della tesi 2 — Shortcut della struttura d'intervista

**RQ1. Quanto predicono separatamente risposta e domanda?** Confrontare `P-only`, `E-only`, `E+P` e risposte bilanciate per lunghezza/tipo di domanda. Risponde se il dataset introduce leakage dal protocollo.

**RQ2. La decisione resiste a trasformazioni semanticamente neutre?** Rimuovere o permutare la domanda, fare length matching e normalizzare filler/disfluenze. Risponde se il modello è stabile quando il contenuto del partecipante non cambia.

**RQ3. Esistono meccanismi distinti per semantica e protocollo?** Confrontare J-space, circuiti e risposta alle ablation nelle condizioni originali e controfattuali. Risponde se lo strumento individua davvero la scorciatoia.

**RQ4. Il modello generalizza a famiglie di domande non viste?** Usare leave-question-family-out. Risponde se la performance deriva da regolarità generali o da una mappa domanda-label specifica.

Metriche: differenze appaiate di macro-F1/AUROC, agreement, class flip rate, variazione del logit gap, divergenza Jensen-Shannon, overlap di circuiti e test di McNemar/bootstrap.

## RQ della tesi 3 — Rappresentazioni dei domini PHQ-8

**RQ1. Gli item PHQ-8 sono predicibili separatamente?** Formulare una predizione multi-task dello score totale e dei singoli item. Risponde se il target globale può essere decomposto in domini informativi.

**RQ2. Le feature interne sono specifiche o generiche?** Separare feature associate a sonno, energia, umore e altri item da sentiment, negazione, lunghezza e stile. Risponde se il modello codifica concetti distinguibili oppure un unico segnale di distress.

**RQ3. Le feature sono necessarie e sufficienti?** Testare ablation, insertion e patching su ciascun item. Risponde se una feature ha un effetto selettivo sulla predizione del dominio dichiarato.

**RQ4. I metodi meccanicistici sono più fedeli delle spiegazioni post-hoc?** Confrontarli con saliency, attention e rationale testuali sotto lo stesso budget di intervento. Risponde quale spiegazione anticipa meglio l'effetto causale reale.

Metriche: MAE/RMSE e Spearman per score, macro-F1 e quadratic weighted kappa per item, similarità delle direzioni, selettività degli interventi e accordo tra annotatori.

## RQ della tesi 4 — Spiegazioni multimodali testo-voce

**RQ1. L'audio aggiunge valore oltre al testo?** Confrontare transcript, COVAREP/formanti, embedding audio e fusione tardiva. Risponde se una componente multimodale è scientificamente giustificata.

**RQ2. Quale modalità guida la decisione?** Rimuovere testo/audio o sostituirne rappresentazioni fra esempi matched. Risponde se esiste complementarità, dominanza o leakage di una modalità.

**RQ3. L'interazione testo-voce è spiegabile causalmente?** Usare token acustici interpretabili nel decoder, oppure un piccolo modello di fusione, e applicare ablation/patching. Risponde a come il modello combina contenuto linguistico e proxy prosodici.

Metriche: guadagno della fusione rispetto al migliore unimodale, `delta-logit` per modality ablation, calibrazione, conditional permutation importance e intervalli bootstrap per partecipante.

## RQ della tesi 5 — Stabilità sotto shift

**RQ1. Prestazione e calibrazione degradano sotto shift?** Addestrare/calibrare su un sottogruppo o dominio e testare su un altro. Risponde se il comportamento è trasferibile, non solo accurato in-distribution.

**RQ2. I concetti J-space si trasferiscono?** Confrontare DAIC con una versione social-media controllata per keyword/community, oppure con sottogruppi e famiglie di domande DAIC. Risponde se le feature semantiche sopravvivono quando cambia la fonte del testo.

**RQ3. I circuiti causali vengono riusati?** Selezionare feature sul dominio sorgente e misurare l'effetto delle ablation sul dominio target. Risponde se la spiegazione è stabile oppure locale al dataset.

**RQ4. Possiamo separare shortcut di intervista e shortcut di community?** Permutare/rimuovere la domanda su DAIC e mascherare auto-diagnosi o riferimenti di community sul dominio esterno. Risponde quale artefatto produce il cambiamento della decisione.

Metriche: generalization gap, Brier/ECE, risk-coverage curve, overlap di feature/edge, selettività cross-domain e intervalli bootstrap gerarchici. I label social non vanno mai trattati come equivalenti a una diagnosi o al PHQ-8.

## Vincoli metodologici comuni

1. Split, soglie, metriche primarie e controlli devono essere fissati prima dell'analisi interpretativa.
2. L'unità statistica è la sessione/partecipante; il PHQ-8 non assegna un label clinico a ogni turno.
3. Il backbone resta inizialmente congelato: un LoRA può invalidare lens e transcoders adattati al checkpoint base.
4. Ogni feature/circuito deve essere testato con interventi, controlli abbinati e replica su esempi/split; un grafo attraente non dimostra causalità.
5. Non pubblicare trascrizioni, audio, ID o esempi riconoscibili; riportare risultati aggregati e failure table.

## Perimetro consigliato

La scelta più realistica è la **tesi 2**, con la tesi 1 come metodo: audit meccanicistico del contributo di contenuto, domanda e lunghezza in un predittore PHQ-8 basato sulle sole trascrizioni. La tesi 5 diventa il capitolo conclusivo di robustezza, prima tra famiglie di domanda e sottogruppi DAIC; un'estensione Reddit/SWMH è facoltativa e non necessaria per completare la tesi.

La formulazione iniziale può essere:

> *Can mechanistic interventions distinguish symptom-relevant evidence from interview-structure shortcuts in a small language model that estimates PHQ-8 severity from DAIC transcripts?*

## Piano di lavoro

| Fase | Risultato atteso | Stato |
| --- | --- | --- |
| Scelta del perimetro | Una tesi, RQ, target e criteri di successo selezionati | Da definire |
| Dati e protocollo | Asset disponibili, split per partecipante, controlli ed etica | Da definire |
| Baseline | TF-IDF/lineare, encoder e decoder piccolo valutati | Da definire |
| Audit meccanicistico | J-space, circuiti, ablation e patching con controlli | Da definire |
| Robustezza | Controfattuali, stabilità per split/domanda/sottogruppo | Da definire |
| Scrittura e replica | Artefatti riproducibili, tabelle, figure e discussione dei limiti | Da definire |

## Prossimo passo

Scegliere una delle cinque tesi, fissare target e RQ primarie, quindi registrare il primo avanzamento nel [diario delle settimane]({{ '/settimane/' | relative_url }}).
