# Modelli Hugging Face per linguaggio psichiatrico e salute mentale

## Scopo e vincolo

Questa è una bibliografia operativa di checkpoint già addestrati reperiti su Hugging Face. Tutti i modelli elencati sono entro il limite di **3B parametri** e hanno una model card che dichiara almeno task, dati o provenienza dei pesi.

La dicitura “psichiatrico” qui riguarda il **linguaggio della salute mentale**; non implica validazione clinica. I modelli selezionati sono stati pre-addestrati o fine-tuned soprattutto su Reddit/social media, non su colloqui clinici DAIC-WOZ. Non vanno quindi usati per diagnosi e non possono sostituire un modello allenato e valutato sul target PHQ-8 della tesi.

**Verifica effettuata:** 29 settembre 2026.

## Candidati raccomandati

| Modello | Parametri | Tipo e task | Dati/contesto dichiarato | Licenza | Uso consigliato nella tesi |
| --- | ---: | --- | --- | --- | --- |
| [mental/mental-bert-base-uncased](https://huggingface.co/mental/mental-bert-base-uncased) | circa 110M | BERT masked-LM, pretraining di dominio | Post Reddit relativi alla salute mentale | CC-BY-NC-4.0, gated | Baseline encoder da fine-tunare su DAIC; confronto fra pretraining generico e di dominio |
| [mental/mental-roberta-base](https://huggingface.co/mental/mental-roberta-base) | circa 125M | RoBERTa masked-LM, pretraining di dominio | Post Reddit relativi alla salute mentale | CC-BY-NC-4.0, gated | Alternativa a MentalBERT; utile per verificare sensibilità all'architettura |
| [citiusLTL/DisorBERT](https://huggingface.co/citiusLTL/DisorBERT) | circa 110M | BERT masked-LM, doppio domain adaptation | Prima social media, poi salute mentale; mascheramento guidato da lessico dei disturbi | CC-BY-NC-4.0 | Baseline di dominio più controllata, con riferimento ACL e metodologia esplicita |
| [citiusLTL/DisorRoBERTa](https://huggingface.co/citiusLTL/DisorRoBERTa) | circa 125M | RoBERTa masked-LM, doppio domain adaptation | Stesso approccio DisorBERT | CC-BY-4.0 | Baseline encoder preferibile se serve una licenza più permissiva |
| [rafalposwiata/deproberta-large-v1](https://huggingface.co/rafalposwiata/deproberta-large-v1) | 0,4B | RoBERTa-large continuato su dominio depressione | Post Reddit depressivi; shared task LT-EDI 2022 | Non dichiarata nella model card: verificare prima dell'uso | Baseline domain-adapted per esperimenti Reddit/domain shift, non per inferenza clinica |
| [rafalposwiata/deproberta-large-depression](https://huggingface.co/rafalposwiata/deproberta-large-depression) | 0,4B | Classificatore già fine-tuned | Post social in inglese; classi `not depression` / `moderate` / `severe` | Non dichiarata nella model card: verificare prima dell'uso | Baseline esterna congelata; misura il mismatch tra label social e PHQ-8 |
| [SajjadIslam/multiMentalRoBERTA-5-class](https://huggingface.co/SajjadIslam/multiMentalRoBERTA-5-class) | 0,4B | Classificatore RoBERTa fine-tuned a 5 classi | Corpora Reddit curati e testo neutro; ansia, depressione, PTSD, suicidal, none | Apache-2.0 | Baseline multi-classe per domain shift e studi sugli shortcut; non usare per crisis triage |
| [reab5555/mentBERT](https://huggingface.co/reab5555/mentBERT) | 0,1B | Classificatore BERT fine-tuned a 9 classi | Post Reddit bilanciati; le menzioni dirette delle patologie dichiarate rimosse | Apache-2.0 | Baseline leggera per testare quanto l'effetto dipenda da keyword esplicite |

## Candidati specialistici o di confronto

| Modello | Parametri | Perché può essere utile | Limite per il progetto corrente |
| --- | ---: | --- | --- |
| [tahaenesaslanturk/mental-health-classification-v0.2](https://huggingface.co/tahaenesaslanturk/mental-health-classification-v0.2) | 0,3B | Classificatore a 15 community/categorie di salute mentale, basato su una fonte bibliografica dichiarata | Card dichiara uso del 2% del dataset; qualità e riproducibilità più deboli rispetto ai candidati principali |
| [ELiRF/Longformer-es-m-large-ICE-MR24-DD](https://huggingface.co/ELiRF/Longformer-es-m-large-ICE-MR24-DD) | circa 435M | Modello di ricerca long-context per disorder detection e early detection | È spagnolo: utile solo come riferimento metodologico, non per i transcript inglesi DAIC |
| [superb/wav2vec2-base-superb-er](https://huggingface.co/superb/wav2vec2-base-superb-er) | circa 95M | Classificatore vocale di emozioni già fine-tuned, utile per l'estensione audio | Le emozioni IEMOCAP non sono disturbi psichiatrici né PHQ-8; usarlo solo come controllo paralinguistico |

## Come interpretarli correttamente

### Pretrained di dominio: da fine-tunare sul target della tesi

MentalBERT, MentalRoBERTa, DisorBERT, DisorRoBERTa e DepRoBERTa-v1 forniscono rappresentazioni linguistiche già adattate a testi relativi alla salute mentale. Per DAIC-WOZ vanno usati con un classifier head addestrato esclusivamente su train/dev, con split per partecipante. Sono baseline appropriate, ma non sostituiscono Gemma 2 2B o Qwen3 1.7B nell'analisi J-space/Circuit Tracer.

### Classificatori già fine-tuned: da usare congelati come baseline esterna

DepRoBERTa-depression, multiMentalRoBERTa e mentBERT restituiscono label costruite su social media. La loro utilità è stimare il **domain mismatch**: un modello addestrato su community online cambia comportamento su interviste semi-strutturate? Non devono essere riaddestrati e poi presentati come “già validati” per PHQ-8.

### Cosa non si può concludere

- Una classe `depression` su Reddit non equivale a `PHQ8 >= 10`.
- Un'alta confidence del modello non è una valutazione psichiatrica affidabile.
- Gli score dichiarati sulle model card sono spesso self-reported e non sono confrontabili tra dataset diversi.
- I modelli encoder elencati non dispongono della stessa toolchain di transcoders/Jacobian Lens prevista per Gemma/Qwen; sono baseline predittive e di domain shift, non il nucleo dell'audit meccanicistico.

## Selezione minima consigliata

Per evitare un confronto troppo esteso:

1. **Baseline generale:** TF-IDF + Logistic Regression e ModernBERT-large.
2. **Baseline di dominio da fine-tunare:** DisorRoBERTa oppure MentalRoBERTa.
3. **Baseline sociale congelata:** multiMentalRoBERTa-5-class oppure DepRoBERTa-depression.
4. **Modello meccanicistico principale:** Gemma 2 2B; **replica:** Qwen3 1.7B.

Questa configurazione separa quattro domande diverse: beneficio di un encoder neurale, beneficio del pretraining mental-health, fallimento/trasferimento di un classificatore social già pronto e meccanismi interni del decoder tracciabile.

## Riferimenti bibliografici e fonti primarie

- Ji et al. (2022), *MentalBERT: Publicly Available Pretrained Language Models for Mental Healthcare*. La model card di [MentalBERT](https://huggingface.co/mental/mental-bert-base-uncased) e quella di [MentalRoBERTa](https://huggingface.co/mental/mental-roberta-base) descrivono il pretraining su Reddit e avvertono che le predizioni non sono diagnosi.
- Aragon et al. (2023), *DisorBERT: A Double Domain Adaptation Model for Detecting Signs of Mental Disorders in Social Media*, ACL. Le card di [DisorBERT](https://huggingface.co/citiusLTL/DisorBERT) e [DisorRoBERTa](https://huggingface.co/citiusLTL/DisorRoBERTa) documentano la doppia domain adaptation.
- Poświata e Perełkiewicz (2022), *Detecting Signs of Depression from Social Media Text using RoBERTa Pre-trained Language Models*, LT-EDI/ACL. Vedi [DepRoBERTa](https://huggingface.co/rafalposwiata/deproberta-large-v1) e il suo [classificatore fine-tuned](https://huggingface.co/rafalposwiata/deproberta-large-depression).
- Islam, Fields e Madiraju (2025), *multiMentalRoBERTa: A Fine-tuned Multiclass Classifier for Mental Health Disorder*, IEEE BigData. La [model card](https://huggingface.co/SajjadIslam/multiMentalRoBERTA-5-class) specifica classi, dati, metriche e limiti.
- La card di [mentBERT](https://huggingface.co/reab5555/mentBERT) documenta classi, conteggi e prestazioni self-reported, incluse le limitazioni necessarie per un suo uso solo sperimentale.

## Modelli esplicitamente esclusi

- `loeslab/mistral_small_psych`: molto pertinente per estrazione di fenotipi psichiatrici in cartelle cliniche spagnole, ma deriva da Mistral Small 24B e supera il vincolo.
- Chat model mental-health da 7B o 8B: superano il vincolo e, in genere, non hanno evidenza migliore per l'audit meccanicistico proposto.
- Checkpoint senza una card che riporti dati, task o limiti: non sono inclusi nella selezione consigliata, anche se compaiono nelle ricerche del Hub.
