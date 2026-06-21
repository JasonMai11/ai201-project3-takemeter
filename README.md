# r/soccer Discourse Classifier

## Project Overview

This project builds a text classifier for discourse in **r/soccer**, a Reddit community where users discuss football news, transfers, match results, tactics, statistics, club drama, jokes, and fan reactions. The goal is to classify each post or comment into one of four discourse categories:

- `news / information update`
- `evidence-based analysis`
- `reaction / hot take`
- `humor / banter`

The classifier is meant to distinguish substantive football discussion from other common forms of community discourse. This matters because r/soccer contains a wide range of text quality: some comments provide useful analysis or information, while others are emotional reactions, jokes, or rivalry banter.

## Community Choice and Reasoning

I chose **r/soccer** because it is active, public, text-heavy, and varied in quality. A single thread can contain official news, transfer rumors, tactical analysis, emotional reactions, jokes, low-effort banter, and arguments about refereeing or player performance.

This community is a good classification target because the task is not just about topic. Two posts can both mention the same club, player, or match but have different discourse purposes. For example, a comment about a manager could be a factual update, an evidence-based tactical argument, a hot take, or a joke. Regular users in the community would recognize that those are meaningfully different kinds of contributions.

## Label Taxonomy

### Label 1: `news / information update`

**Definition:** The text mainly shares football-related information, such as a transfer report, official announcement, injury update, match result, fixture update, or source-based claim.

**Example 1:**  
"Official: Chelsea have announced the signing of a new midfielder on a seven-year contract."

**Example 2:**  
"Brazil's squad for the next international break has been released, with Vinicius Jr., Rodrygo, and Endrick included."

### Label 2: `evidence-based analysis`

**Definition:** The text makes a football argument using reasoning, statistics, tactical explanation, financial context, historical comparison, or specific examples from matches.

**Example 1:**  
"Arsenal struggled because their left side was overloaded. Their fullback kept stepping into midfield, but the winger stayed too wide, which left space behind them every time they lost possession."

**Example 2:**  
"Mbappe's defensive numbers are low, but that may be intentional. If the manager wants him ready for counterattacks, keeping him higher up the pitch can create more value than asking him to press constantly."

### Label 3: `reaction / hot take`

**Definition:** The text mainly expresses a strong opinion, emotional judgment, or quick conclusion without much evidence or explanation.

**Example 1:**  
"This manager is completely finished. No serious club should hire him again."

**Example 2:**  
"Scoring goals does not matter if you disappear every time your team plays a good defense."

### Label 4: `humor / banter`

**Definition:** The text is mainly intended to joke, meme, mock a player or club, exaggerate for comedic effect, or participate in rivalry banter.

**Example 1:**  
"Chelsea are going to sign every teenager in Europe before remembering they can only play eleven players."

**Example 2:**  
"He has the first touch of a trampoline and the confidence of prime Ronaldo."

## Data Collection Source and Labeling Process

The dataset contains **200 labeled examples** collected from public r/soccer posts and comments. I used public community content only and did not include private messages, deleted comments, removed comments, or AutoModerator comments.

I collected examples from different types of r/soccer threads to avoid making the dataset too narrow:

- Post-match threads
- Match threads
- Daily discussion threads
- Transfer rumor threads
- Official news threads
- Player statistic threads
- Tactical discussion threads
- Major tournament or Champions League discussion threads

The CSV file contains one complete labeled dataset and is not pre-split. The notebook handles the train/validation/test split automatically.

### Dataset Columns

| Column | Description |
|---|---|
| `text` | The post or comment text being classified. |
| `label` | The manually assigned discourse label. |
| `notes` | Optional notes for difficult or ambiguous examples. |
| `source_url` | The public source URL for the example. |

### Label Distribution

| Label | Count |
|---|---:|
| `news / information update` | 50 |
| `evidence-based analysis` | 50 |
| `reaction / hot take` | 50 |
| `humor / banter` | 50 |
| **Total** | **200** |

The dataset is balanced, with each label making up 25% of the full dataset. No label exceeds 70% of the dataset.

## Difficult-to-Label Examples

The hardest cases were examples where r/soccer users mixed football-specific language, sarcasm, and emotion. I labeled each example by its dominant purpose.

| Example | Possible Labels | Final Label | Decision |
|---|---|---|---|
| "I think it was actually encroachment inside the box maybe" | `evidence-based analysis`, `reaction / hot take` | `evidence-based analysis` | This gives a possible rule-based explanation for a refereeing decision, so I treated it as analysis even though it is short and uncertain. |
| "Arsenal heritage" | `humor / banter`, `reaction / hot take` | `humor / banter` | This relies on fan culture, sarcasm, and club reputation rather than making a serious football argument. |
| "Technology now being abused. This isn't what it's meant for. Trash penalty didn't deserve retake." | `reaction / hot take`, `evidence-based analysis` | `reaction / hot take` | The comment discusses VAR/refereeing, but it does not explain the rule or provide evidence. Its dominant purpose is emotional judgment. |
| "Romano says United are interested in another overpriced winger. This club never learns." | `news / information update`, `reaction / hot take` | `reaction / hot take` | Even though it references a source, the main purpose is the user's complaint about the club. |

## Train / Validation / Test Split

The notebook automatically split the dataset into train, validation, and test sets.

| Split | Examples |
|---|---:|
| Train | 140 |
| Validation | 30 |
| Test | 30 |

### Train Label Distribution

| Label | Count |
|---|---:|
| `news / information update` | 35 |
| `humor / banter` | 35 |
| `reaction / hot take` | 35 |
| `evidence-based analysis` | 35 |

### Test Label Distribution

| Label | Count |
|---|---:|
| `news / information update` | 8 |
| `reaction / hot take` | 8 |
| `humor / banter` | 7 |
| `evidence-based analysis` | 7 |

## Model Approaches

This project compared two approaches:

1. **Zero-shot Groq baseline** using `llama-3.3-70b-versatile`
2. **Fine-tuned DistilBERT** using `distilbert-base-uncased`

## Baseline Description

The baseline used the Groq API with `llama-3.3-70b-versatile`. The model was not trained on my dataset. Instead, it was prompted with the label definitions and asked to classify each test example directly.

The baseline was evaluated on the same 30-example test set as the fine-tuned model. Only parseable responses were counted, and the prompt instructed the model to output exactly one valid label.

### Baseline Prompt Used

```text
You are a strict text classifier for Reddit posts and comments from r/soccer.

Classify each text into exactly ONE of these lowercase labels:

news / information update
evidence-based analysis
reaction / hot take
humor / banter

Label definitions:

news / information update:
The text mainly shares football-related information, such as a transfer report, official announcement, injury update, match result, fixture update, or source-based claim.

evidence-based analysis:
The text makes a football argument using reasoning, statistics, tactical explanation, financial context, historical comparison, or specific examples from matches.

reaction / hot take:
The text mainly expresses a strong opinion, emotional judgment, or quick conclusion without much evidence or explanation.

humor / banter:
The text is mainly intended to joke, meme, mock a player or club, exaggerate for comedic effect, or participate in rivalry banter.

Output rules:
Return only one lowercase label.
Do not explain.
Do not include "Label:".
Do not use quotes.
Do not include punctuation after the label.
Your entire response must be exactly one of these labels.
```

## Fine-Tuning Approach

The fine-tuned model used **DistilBERT**, specifically `distilbert-base-uncased`, as the base model. The model was trained as a four-class text classifier using the labels from the CSV.

### Training Setup

| Setting | Value |
|---|---|
| Base model | `distilbert-base-uncased` |
| Task | Four-class sequence classification |
| Training examples | 140 |
| Validation examples | 30 |
| Test examples | 30 |
| Final training epochs | 4 |
| Best validation accuracy shown | 0.600 |
| Final validation loss shown | 0.984531 |

### Hyperparameter Decision

The main hyperparameter decision was increasing training from **3 epochs to 4 epochs**. At 3 epochs, the model was weaker and did not classify the test set well. After training for 4 epochs, validation accuracy improved to 0.600 and final test accuracy improved to 0.767.

Training log summary:

| Epoch | Training Loss | Validation Loss | Validation Accuracy |
|---:|---:|---:|---:|
| 1 | No log | 1.124959 | 0.500000 |
| 2 | 1.059308 | 1.097051 | 0.500000 |
| 3 | 0.970258 | 1.048790 | 0.566667 |
| 4 | 0.948842 | 0.984531 | 0.600000 |

## Evaluation Metrics

Accuracy alone is not enough for this task because some labels are harder to classify than others. The most important metric is **macro F1**, because it treats all four labels equally and shows whether the classifier performs reasonably well across the full label set.

The evaluation uses:

- Accuracy
- Per-label precision, recall, and F1-score
- Macro average F1-score
- Weighted average F1-score
- Confusion matrix
- Manual error analysis

## Baseline Results: Zero-Shot Groq

| Metric | Score |
|---|---:|
| Accuracy | 0.767 |
| Macro Precision | 0.88 |
| Macro Recall | 0.76 |
| Macro F1 | 0.77 |
| Weighted Precision | 0.88 |
| Weighted Recall | 0.77 |
| Weighted F1 | 0.77 |

### Baseline Per-Class Metrics

| Label | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| `news / information update` | 1.00 | 0.75 | 0.86 | 8 |
| `evidence-based analysis` | 1.00 | 0.43 | 0.60 | 7 |
| `reaction / hot take` | 0.53 | 1.00 | 0.70 | 8 |
| `humor / banter` | 1.00 | 0.86 | 0.92 | 7 |

The baseline met the original target accuracy threshold of 0.75 and achieved a strong macro F1 score of 0.77. Its weakest label was `evidence-based analysis`, with recall of 0.43, meaning it missed several true analysis examples.

## Fine-Tuned Model Results: DistilBERT

| Metric | Score |
|---|---:|
| Accuracy | 0.767 |
| Macro Precision | 0.77 |
| Macro Recall | 0.76 |
| Macro F1 | 0.76 |
| Weighted Precision | 0.77 |
| Weighted Recall | 0.77 |
| Weighted F1 | 0.76 |

### Fine-Tuned Per-Class Metrics

| Label | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| `news / information update` | 0.89 | 1.00 | 0.94 | 8 |
| `evidence-based analysis` | 0.62 | 0.71 | 0.67 | 7 |
| `reaction / hot take` | 0.71 | 0.62 | 0.67 | 8 |
| `humor / banter` | 0.83 | 0.71 | 0.77 | 7 |

## Results Comparison

| Model | Accuracy | Macro F1 |
|---|---:|---:|
| Zero-shot Groq baseline | 0.767 | 0.77 |
| Fine-tuned DistilBERT | 0.767 | 0.76 |

The fine-tuned model matched the zero-shot baseline on accuracy. The baseline had a slightly higher macro F1, but the fine-tuned model had more balanced recall across labels. The fine-tuned model also improved `evidence-based analysis` recall compared with the baseline, increasing it from 0.43 to 0.71.

## Confusion Matrix

The confusion matrix for the fine-tuned model is included below as a markdown table and also saved as `confusion_matrix.png`.

| True Label \\ Predicted Label | `news / information update` | `evidence-based analysis` | `reaction / hot take` | `humor / banter` |
|---|---:|---:|---:|---:|
| `news / information update` | 8 | 0 | 0 | 0 |
| `evidence-based analysis` | 1 | 5 | 0 | 1 |
| `reaction / hot take` | 0 | 3 | 5 | 0 |
| `humor / banter` | 0 | 0 | 2 | 5 |

![Confusion Matrix](confusion_matrix.png)

## Wrong Predictions With Analysis

The fine-tuned model made **7 wrong predictions out of 30 test examples**.

| Text | True Label | Predicted Label | Confidence | Analysis |
|---|---|---|---:|---|
| "I think it was actually encroachment inside the box maybe" | `evidence-based analysis` | `humor / banter` | 0.31 | This is short and uncertain, so the model may have missed that it was trying to explain a refereeing decision. |
| "Mbappe dead last is crazy. What's crazier is wtf is Ferran doing down there tf, I swear he presses a lot actually" | `reaction / hot take` | `evidence-based analysis` | 0.43 | The model likely focused on the discussion of pressing/stat rankings, but the tone is mostly emotional and reactionary. |
| "Refs decided to go back to the opening game mentality." | `reaction / hot take` | `evidence-based analysis` | 0.34 | Refereeing comments can sound analytical, but this one is mainly a quick judgment without evidence. |
| "It's coming home, England doesn't have to do anything" | `humor / banter` | `reaction / hot take` | 0.39 | This is based on a football meme phrase, but the model treated it as a serious opinion. |
| "BS penalty given and even bigger BS given again. This has nothing to do with football anymore gents." | `reaction / hot take` | `evidence-based analysis` | 0.44 | The comment mentions refereeing decisions, but the wording is emotional and unsupported, so it belongs in hot take. |

The remaining mistakes show that the hardest boundary is between **analysis** and **reaction/hot take**. Some comments include a reason or football-specific observation, but the tone is emotional or informal. The model also confused some banter with hot takes because r/soccer humor often works by exaggerating a serious opinion.

## Sample Classification Results

The fine-tuned model produced **23 correct predictions out of 30 test examples**. The table below shows several correct predictions with the model's predicted label and confidence score.

| Text | True Label | Predicted Label | Confidence | Correct? |
|---|---|---|---:|---|
| "The Athletic on Alexi Lalas: He is one of the most insufferable analysts in American TV sports history..." | `news / information update` | `news / information update` | 0.34 | Yes |
| "Schmeichel blasts Martinez: 'He's wasting Portugal just like he wasted Belgium.'" | `news / information update` | `news / information update` | 0.43 | Yes |
| "Norway's football team official photo for World Cup 2026" | `news / information update` | `news / information update` | 0.71 | Yes |
| "Post Match Thread: Portugal 1 - 1 Congo DR \| FIFA World Cup 2026" | `news / information update` | `news / information update` | 0.75 | Yes |
| "Nah, Raphinha ranking high in these type of stats" | `reaction / hot take` | `reaction / hot take` | 0.36 | Yes |
| "Harry Kane and missing PKs" | `humor / banter` | `humor / banter` | 0.32 | Yes |
| "So as I understand it the goalkeeper has to stay on the goal line until the shot is taken. Apparently the goalkeeper was minimally off the goal line" | `evidence-based analysis` | `evidence-based analysis` | 0.51 | Yes |
| "Ferran works hard at pressing and runs all the time, every Barca fan knows he has issues, but pressing isnt one of them" | `evidence-based analysis` | `evidence-based analysis` | 0.40 | Yes |

### Correct Example Explanation

One useful correct prediction was:

> "So as I understand it the goalkeeper has to stay on the goal line until the shot is taken. Apparently the goalkeeper was minimally off the goal line"

The true label was `evidence-based analysis`, and the model also predicted `evidence-based analysis` with confidence 0.51. This is a good example because the comment is not just reacting emotionally to a referee decision. It tries to explain the rule and connect the decision to the goalkeeper's positioning, which matches the definition of evidence-based analysis.

Another strong correct prediction was:

> "Post Match Thread: Portugal 1 - 1 Congo DR | FIFA World Cup 2026"

The true label was `news / information update`, and the model predicted `news / information update` with confidence 0.75. This shows that the model learned a clear signal for informational posts, especially thread titles or headlines that mainly report an event or result.


## Did the Model Meet the Success Criteria?

The original success criteria were:

- At least 75% overall accuracy
- At least 0.70 macro F1
- At least 0.65 F1 for each individual label
- No single label should have recall below 0.60

### Fine-Tuned Model

The fine-tuned model **met the success criteria**.

| Criterion | Target | Fine-Tuned Result | Met? |
|---|---:|---:|---|
| Accuracy | >= 0.75 | 0.767 | Yes |
| Macro F1 | >= 0.70 | 0.76 | Yes |
| Each label F1 | >= 0.65 | Lowest: 0.67 | Yes |
| Each label recall | >= 0.60 | Lowest: 0.62 | Yes |

### Zero-Shot Baseline

The zero-shot baseline met the overall accuracy and macro F1 goals, but it did not meet every per-label goal.

| Criterion | Target | Baseline Result | Met? |
|---|---:|---:|---|
| Accuracy | >= 0.75 | 0.767 | Yes |
| Macro F1 | >= 0.70 | 0.77 | Yes |
| Each label F1 | >= 0.65 | Lowest: 0.60 | No |
| Each label recall | >= 0.60 | Lowest: 0.43 | No |

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the difference between four types of r/soccer discourse: factual updates, evidence-based analysis, unsupported reactions, and humor/banter. The fine-tuned model appears to have learned the strongest boundary for `news / information update`, since it correctly classified all 8 news examples in the test set.

The model learned some of the intended distinction between analysis, hot takes, and banter, but those boundaries remain weaker. The most common errors involved emotional comments that mentioned statistics, refereeing, or football concepts. This suggests the model sometimes used football-specific words as a signal for `evidence-based analysis`, even when the dominant purpose was actually reaction or banter.

Overall, the model learned a useful version of the label taxonomy, but it still needs more hard examples around the boundary between analytical language and emotional fan reactions.

## Spec Reflection

One way the project specification helped was by forcing the labels to be defined before collecting all 200 examples. That made the annotation process more consistent because I had clear rules for deciding between labels.

One way implementation diverged from the original plan was the training setup. I initially expected the fine-tuned model to work with the original number of epochs, but the first result was much weaker than expected. I increased training to 4 epochs, which significantly improved the fine-tuned model and allowed it to meet the success criteria.

## Data Improvements to Try Next

The biggest remaining issue is the boundary between `evidence-based analysis`, `reaction / hot take`, and `humor / banter`.

Recommended data improvements:

1. **Add more hard negatives for `evidence-based analysis`.**  
   Add more examples that contain football-specific words, player names, stats, or tactical language but should still be labeled `reaction / hot take`, `humor / banter`, or `news / information update`.

2. **Add more referee and VAR examples.**  
   Several wrong predictions involved officiating comments. These comments often sound analytical because they discuss decisions, but many are really emotional reactions.

3. **Replace very short low-context examples or add context.**  
   Comments like "It's coming home" require cultural context. If possible, include the thread title with the comment text.

4. **Add thread context to the `text` column.**  
   For ambiguous comments, the text could include both the thread title and the comment, such as:
   `Thread: [post title] Comment: [comment text]`.

5. **Audit examples where the baseline and fine-tuned model disagree.**  
   Disagreements between the two models can help identify unclear label boundaries or inconsistent annotations.

6. **Increase the dataset size.**  
   Moving from 200 examples to 400 or more would give the fine-tuned model more examples of the subtle label boundaries.

Synthetic examples should not be added to the training data unless the assignment explicitly allows it. The dataset should remain based on public r/soccer content.

## AI Usage

AI tools were used during planning, label stress-testing, annotation support, debugging, and evaluation writing.

### Instance 1: Label Design and Planning

I used AI to help turn the r/soccer idea into a concrete label taxonomy. I directed the AI to define 2-4 mutually exclusive labels, write one-sentence definitions, and identify hard edge cases. I revised the output by tightening the boundary between `evidence-based analysis` and `reaction / hot take`, especially by requiring analysis examples to include a concrete reason, statistic, tactical explanation, comparison, or source-based claim.

### Instance 2: Annotation Assistance

I used AI assistance to help produce and organize a labeled CSV, but I treated the labels as AI-assisted rather than fully automatic. I reviewed the label definitions and adjusted the labels to use lowercase strings so that the notebook's `LABEL_MAP` and parsing logic would work correctly. I also documented difficult examples in the `notes` column and in the planning document.

### Instance 3: Debugging and Evaluation Writing

I used AI to debug notebook issues, including label parsing problems caused by lowercase model outputs and long confusion-matrix labels. I also used AI to help interpret the baseline and fine-tuned model results, compare the two approaches, and write the README in a way that honestly describes what the model learned and where it still struggles.

The dataset examples were collected from public r/soccer content. AI-generated examples were not used as training data. Any AI-assisted labels were reviewed manually.

## How to Run

1. Place the labeled dataset CSV in the repo.
2. Open the project notebook in Google Colab.
3. Make sure the CSV path is correct.
4. Confirm the `LABEL_MAP` matches the lowercase labels:

```python
LABEL_MAP = {
    "news / information update": 0,
    "evidence-based analysis": 1,
    "reaction / hot take": 2,
    "humor / banter": 3,
}
```

5. Run the notebook cells in order.
6. Save the generated outputs, including:
   - `evaluation_results.json`
   - `confusion_matrix.png`

## Conclusion

The fine-tuned DistilBERT model met the project's success criteria after training for 4 epochs. It achieved 0.767 accuracy and 0.76 macro F1 on the test set, matching the zero-shot Groq baseline's accuracy while producing more balanced recall across labels.

For a real community tool, the fine-tuned model is a reasonable first version because it is fast, local, and does not require an LLM API call for every classification. However, the model should still be tested on a larger dataset before being treated as production-ready. The next improvements should focus on adding more hard examples around the boundary between analysis, hot takes, and banter.
