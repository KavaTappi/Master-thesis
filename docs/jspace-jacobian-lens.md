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

## Derivazione matematica e interpretazione geometrica

### Dal residual stream ai logits

In un decoder Transformer, sia `h_l` lo stato del residual stream a una data posizione e layer `l`, e sia `h_L` lo stato al layer finale. Il modello applica poi normalizzazione finale e unembedding per produrre i logits:

```text
h_L = F_l_to_L(h_l, contesto)
z   = W_U LN_final(h_L) + b_U
```

Dove `z_v` e' il logit del token `v` e `W_U` e' la matrice di unembedding. Un logit lens convenzionale approssima questo readout applicando direttamente `LN_final` e `W_U` a `h_l`:

```text
z_logit_lens ~= W_U LN_final(h_l) + b_U
```

L'approssimazione puo' essere povera nei layer intermedi perche' ignora la trasformazione `F_l_to_L`: attention, MLP, residual additions e cambi di coordinate alterano quale direzione interna contribuira' al logit finale.

### Il Jacobian

Il Jacobian locale di tale trasformazione e' la matrice di derivate:

```text
J_l(x) = d h_L / d h_l
```

Per una piccola perturbazione `delta h_l`, la linearizzazione del primo ordine dice:

```text
h_L(h_l + delta h_l) ~= h_L(h_l) + J_l(x) delta h_l
```

e, assorbendo normalizzazione e unembedding nel readout, per il token `v`:

```text
delta z_v ~= w_v^T J_l(x) delta h_l
```

Il vettore

```text
d_l,v(x) = J_l(x)^T w_v
```

e' quindi la direzione, nello spazio del layer `l`, in cui una perturbazione aumenta localmente il logit del token `v`. Il Jacobian Lens stima una versione media/stabile `J_bar_l` su un corpus e usa queste direzioni per leggere un'attivazione:

```text
J_bar_l = media_x [ J_l(x) ]
score_l,v(h_l) = (J_bar_l^T w_v)^T h_l
```

L'implementazione precisa puo' includere centratura, coordinate di LayerNorm, posizioni token e altre correzioni; la formula mostra l'idea essenziale: non si proietta `h_l` direttamente nel vocabolario, ma lo si trasporta prima nella base di lettura finale usando la sensibilita' del modello.

### Che cos'e' geometricamente il J-space

Le direzioni `d_l,v = J_bar_l^T w_v` associate ai token verbalizzabili costituiscono il frame usato dal lens. Il **J-space** e' il sottospazio o la famiglia di direzioni dello stato residuo che ha un effetto leggibile sul vocabolario finale tramite questo trasporto Jacobiano.

Un'attivazione intermedia non deve essere interamente nel J-space. Il lavoro originale osserva che solo una frazione dell'attivazione e' ben descritta da queste direzioni. Perciò un readout J-space non e' una decodifica completa dello stato del modello: e' una finestra sui concetti che il modello e' localmente disposto a rendere esprimibili.

## Perche' si parla di concetti "verbalizzabili"

Un alto score per `sleep` in un layer intermedio non implica che il modello produrra' subito la parola `sleep`. Significa che esiste una direzione interna che, se il modello fosse portato al punto appropriato di output o gli venisse richiesto di riferire quel contenuto, puo' influenzare in modo causale la produzione di quel concetto.

La distinzione evita due errori:

- non leggere il J-space come una chain-of-thought letterale o come "pensieri nascosti";
- non scartare un concetto solo perche' non appare nella risposta finale del modello.

Nella tesi, il J-space va descritto come **readout di contenuti interni verbalizzabili del modello**, non come misura diretta di stati mentali o sintomi del partecipante.

## Esempio operativo su DAIC

Dato un input `P-only`, il modello deve emettere un token di classe `A` o `B`. Durante il processing della risposta del partecipante, per ciascun layer e token si possono ottenere score J-space per concetti come `sleep`, `tired`, `work`, `family`, `question` o per token collegati a disfluenze.

Una traiettoria ipotetica potrebbe essere:

```text
token nella risposta        early layers        middle layers          late layers
"I cannot sleep..."        less readable       sleep, tired           A/B class token
```

L'ipotesi non e' che la parola `sleep` sia sufficiente a spiegare il risultato. L'ipotesi testabile e': i concetti `sleep` e `tired` compaiono nel band di layer intermedi, sono piu' stabili nei casi target rispetto a controlli matched e predicono la direzione di un intervento sul modello.

## Come valutare un readout J-space

| Proprietà | Domanda pratica | Verifica |
|---|---|---|
| Localita' | Il concetto compare in una parte plausibile del transcript? | Confronto tra token/turni e input permutati. |
| Persistenza | Resta attivo in piu' layer intermedi? | Area sotto la traiettoria o numero di layer sopra soglia. |
| Specificita' | Distingue target e controlli a parita' di domanda/lunghezza? | Modello statistico o test a permutazione. |
| Stabilita' | Si replica su seed, split, parafrasi e casi diversi? | Overlap di top-k concetti e intervalli bootstrap. |
| Causalita' | La direzione letta anticipa un cambiamento del target? | Activation patching/steering e Circuit Tracer. |

Un ranking di token visualizzato nella UI non e' una metrica sufficiente. Il valore della tesi sta nel legare la lettura a una previsione verificabile su interventi.

## Relazione con Circuit Tracer

I due strumenti lavorano a risoluzioni diverse e sono complementari:

| Strumento | Oggetto principale | Domanda | Risultato |
|---|---|---|---|
| Jacobian Lens | Residual stream e direzioni verbalizzabili | "Quale contenuto sembra essere disponibile in questo punto del calcolo?" | Traiettoria di concetti per layer/token. |
| Circuit Tracer | Feature sparse, input e logits | "Quali componenti e percorsi contribuiscono al contrasto di classe?" | Grafo con nodi, edge, segno ed effetti diretti. |

Un protocollo forte e' quindi:

1. il J-space propone un contenuto candidato in un punto/layer;
2. Circuit Tracer identifica feature e input che contribuiscono al logit gap di classe;
3. un'ablation o patching stabilisce se il contenuto/circuito era causalmente rilevante;
4. i controlli su domanda, lunghezza e seed stabiliscono se il risultato e' robusto.

Se J-space e Circuit Tracer convergono su una stessa famiglia di feature e l'intervento ha l'effetto previsto, l'evidenza e' piu' convincente. Se divergono, il risultato e' comunque utile: il readout potrebbe essere non causale per quel target, oppure il circuito potrebbe operare tramite informazione non facilmente verbalizzabile.

## Limiti matematici importanti

- `J_bar_l` e' una media sul corpus: puo' non rappresentare bene un routing dipendente da uno specifico esempio, domanda o posizione.
- La linearizzazione e' affidabile per perturbazioni piccole; steering grandi possono uscire dal regime locale e produrre effetti non previsti.
- Le direzioni dipendono da tokenizer e vocabolario: concetti multi-token, frasi e fenomeni non verbalizzabili richiedono estensioni o analisi complementari.
- Il lens e' specifico del checkpoint. Un LoRA o un fine-tuning modifica le derivate e puo' invalidare un lens pre-fitted.
- Un alto score e' un readout del modello, non una prova di verita' semantica, causalita' o rilevanza clinica.

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
