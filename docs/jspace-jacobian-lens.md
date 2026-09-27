# J-space e Jacobian Lens

## Scopo

Il **Jacobian Lens** e' uno strumento di interpretabilita' meccanicistica per modelli linguistici a pesi aperti. Permette di leggere, a uno specifico layer e a una specifica posizione della sequenza, quali concetti il modello e' localmente predisposto a rendere verbalizzabili in seguito.

Il **J-space** e' l'insieme delle direzioni dello stato residuo che il Jacobian Lens riesce a decodificare come concetti verbalizzabili. Non e' una regione fisica separata della rete e non e' una misura dei pensieri di una persona: e' una proprieta' delle rappresentazioni interne del modello.

Nel contesto DAIC, la domanda interessante non e' "il J-space riconosce la depressione?", ma:

> Durante una predizione di PHQ-8, quali concetti interni del modello emergono e restano causalmente rilevanti dopo aver controllato domanda, lunghezza e struttura dell'intervista?

## Intuizione

Un normale logit lens prende un'attivazione intermedia e la legge con il decoder finale, come se quel layer fosse gia' l'ultimo. Questo puo' essere fuorviante: nel passaggio dai layer intermedi all'output, le rappresentazioni cambiano base e vengono elaborate ulteriormente.

Il Jacobian Lens stima invece, per ogni layer, come piccole modifiche allo stato residuo influenzano i logits finali. In forma semplificata, per un'attivazione `h_l` al layer `l`:

```text
J_l = media_corpus [ d h_final / d h_l ]
readout_l(h_l) = unembed( J_l h_l )
```

Il readout puo' quindi mostrare token o concetti che il modello potrebbe rendere espliciti se interrogato al momento appropriato. Il Jacobian Lens non afferma che quei token saranno necessariamente il prossimo output.

## Cosa misurare in una tesi su DAIC

Per ogni risposta del partecipante, salvare per token e layer:

- top-k concetti/tokens letti dal Jacobian Lens;
- layer di comparsa, persistenza e intensita' del concetto;
- predizione PHQ-8 e margine tra classi;
- domanda di Ellie, lunghezza, turni e altri controlli.

Le categorie di analisi vanno stabilite prima dell'ispezione:

| Categoria | Esempi | Interpretazione corretta |
|---|---|---|
| Domini del PHQ-8 | sonno, energia, concentrazione, anedonia | Ipotesi su contenuto usato dal modello, non sintomo osservato nella persona |
| Contesto dell'intervista | famiglia, lavoro, trauma, domanda corrente | Possibile evidenza semantica o dipendenza dal protocollo |
| Scorciatoie | lunghezza, filler, token di domanda, stile | Possibile confondente da controllare |
| Neutro | temi non legati al target | Controllo negativo |

Un'analisi credibile confronta: risposta sola, domanda sola, domanda+risposta, e versioni controfattuali con domanda permutata o lunghezza normalizzata.

## Protocollo J-space proposto

1. Scegliere un modello decoder-only open-weight per cui esista un Jacobian Lens verificato, oppure fittare il lens localmente sul modello esatto usato.
2. Congelare inizialmente il backbone; se si applica LoRA/fine-tuning, documentare che il Jacobian Lens pre-fitted del modello base potrebbe non essere piu' fedele e va rivalidato o rifittato.
3. Fissare split per partecipante, prompt, target e top-k prima di analizzare le attivazioni.
4. Estrarre traiettorie J-space per un campione bilanciato di predizioni corrette/errate e classi PHQ-8.
5. Usare annotazione cieca di concetti o regole lessicali dichiarate per aggregare i readout in categorie.
6. Trattare le feature/contenuti piu' stabili come candidate da verificare con ablation o patching; il J-space da solo non prova causalita'.

## Valutazione

| Domanda | Misura |
|---|---|
| Il concetto e' stabile? | ripetizione su seed, split, esempi e parafrasi |
| E' associato alla predizione? | associazione con logit gap, con controlli per domanda/lunghezza |
| E' specifico? | confronto con input neutri e classi opposte |
| E' causale? | effetto di ablation/activation patching sul logit, meglio se con Circuit Tracer |

Un risultato negativo e' utile: se i concetti J-space cambiano drasticamente quando si permuta la domanda oppure non predicono l'effetto di un intervento, non sono una spiegazione robusta della predizione.

## Multimodalita': cosa e' possibile

Il Jacobian Lens non interpreta automaticamente onde audio, frame video o landmark facciali. Puo' essere usato in un progetto multimodale in tre modi, con valore scientifico crescente:

1. **Audio/video convertiti in testo o token discreti.** Per esempio, estrarre pause, F0, intensita' e descrittori COVAREP, trasformarli in token come `LONG_PAUSE` o `LOW_PITCH_VARIABILITY`, e darli al modello linguistico insieme alla trascrizione. Il J-space mostra come il modello usa questi token, ma non interpreta direttamente il segnale acustico originale.
2. **Modello multimodale con encoder + LLM.** Se un audio/video encoder proietta le sue rappresentazioni nel residual stream del LLM, in linea di principio il J-lens puo' leggere le posizioni multimodali. Servono pero' un lens fittato per quel modello esatto e controlli che separino encoder, proiettore e LLM. Non assumere che un lens di un LLM testuale funzioni su una sua variante multimodale.
3. **Fusion model personalizzato.** Se testo e audio vengono fusi tramite un proprio cross-attention/fusion layer, il J-space del solo LLM non basta. La tesi deve aggiungere probe e interventi sul ramo audio e sul layer di fusione.

Per una tesi magistrale, il percorso prudente e': baseline testo -> token audio interpretabili -> solo dopo, modello multimodale nativo. In questo modo ogni aumento di complessita' produce una domanda verificabile: l'audio aggiunge informazione causale oltre al testo?

## Limiti da dichiarare

- Il metodo dipende dal modello, dalla tokenizzazione e dal corpus usato per fittare il Jacobian medio.
- I concetti multi-token o non verbalizzabili possono non essere letti bene.
- Un readout semanticamente plausibile non e' una spiegazione causale senza intervento.
- Non dedurre stati mentali dei partecipanti da attivazioni del modello.

## Riferimenti essenziali

- Gurnee et al., *Verbalizable Representations Form a Global Workspace in Language Models* (2026): [paper](https://arxiv.org/abs/2607.15495).
- Neuronpedia, *Welcome to the J-Space*: [documentazione/annuncio](https://www.neuronpedia.org/blog/jacobian-lens).
