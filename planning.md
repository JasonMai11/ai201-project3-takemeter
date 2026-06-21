# planning.md

## Project Overview

This project will build a text classifier for discourse in **r/soccer**, a large Reddit community where users discuss football news, transfers, match results, tactics, statistics, club drama, and fan reactions. The goal is to classify posts or comments by the kind of discourse they represent: informational updates, evidence-based analysis, reactionary opinions, or humor/banter. This distinction matters because r/soccer contains both useful football discussion and low-effort fan reactions, so a classifier could help separate substantive discussion from noise.

## 1. Community

The community I chose is **r/soccer**. I chose this community because it is active, text-heavy, and varied in quality. A single thread can include breaking news, serious tactical discussion, player comparisons, transfer rumors, emotional hot takes, jokes, and rivalry banter.

This community is a good fit for a classification task because posts and comments often have different purposes even when they discuss the same topic. For example, a comment about Manchester United could be a news update, a tactical explanation, a frustrated reaction, or a joke. Regular users in r/soccer would recognize the difference between a thoughtful football argument and a low-effort comment like “this club is finished.”

The discourse is varied enough to be interesting because football fans often combine facts, opinions, emotion, sarcasm, and analysis in the same discussion space. This makes the classification task more challenging than simply sorting by topic.

## 2. Labels

I will use four labels.

### Label 1: News / Information Update

A **News / Information Update** is a post or comment whose main purpose is to share football-related information, such as a transfer report, official announcement, injury update, match result, fixture update, or source-based claim.

Example 1:
“Official: Chelsea have announced the signing of a new midfielder on a seven-year contract.”

Example 2:
“Brazil’s squad for the next international break has been released, with Vinícius Jr., Rodrygo, and Endrick included.”

### Label 2: Evidence-Based Analysis

An **Evidence-Based Analysis** post or comment makes a football argument using reasoning, statistics, tactical explanation, financial context, historical comparison, or specific examples from matches.

Example 1:
“Arsenal struggled because their left side was overloaded. Their fullback kept stepping into midfield, but the winger stayed too wide, which left space behind them every time they lost possession.”

Example 2:
“Mbappé’s defensive numbers are low, but that may be intentional. If the manager wants him ready for counterattacks, keeping him higher up the pitch can create more value than asking him to press constantly.”

### Label 3: Reaction / Hot Take

A **Reaction / Hot Take** is a post or comment whose main purpose is to express a strong opinion, emotional judgment, or quick conclusion without giving much evidence or explanation.

Example 1:
“This manager is completely finished. No serious club should hire him again.”

Example 2:
“Scoring goals does not matter if you disappear every time your team plays a good defense.”

### Label 4: Humor / Banter

A **Humor / Banter** post or comment is mainly intended to joke, meme, mock a player or club, exaggerate for comedic effect, or participate in rivalry-based banter rather than make a serious argument.

Example 1:
“Chelsea are going to sign every teenager in Europe before remembering they need to play only eleven players.”

Example 2:
“He has the first touch of a trampoline and the confidence of prime Ronaldo.”

## 3. Hard Edge Cases

The hardest edge case will be between **Evidence-Based Analysis** and **Reaction / Hot Take**. Many r/soccer comments sound analytical because they use football language, but they do not actually provide evidence or reasoning.

For example:
“He only works in transition-heavy systems and gets exposed against low blocks.”

This could be analysis if the user explains why, gives match examples, or references tactical patterns. However, if it is written as a single unsupported claim, I will label it as **Reaction / Hot Take**.

My rule will be: a comment only counts as **Evidence-Based Analysis** if it includes at least one concrete reason, example, statistic, tactical explanation, financial explanation, historical comparison, or source-based claim. If the comment only states a judgment, even if it sounds intelligent, I will label it as **Reaction / Hot Take**.

Another edge case will be between **News / Information Update** and **Reaction / Hot Take**. Some users share rumors or reports but add emotional language.

For example:
“Romano says United are interested in another overpriced winger. This club never learns.”

If the main purpose is to report the rumor, I will label it **News / Information Update**. If the main purpose is the user’s complaint or judgment, I will label it **Reaction / Hot Take**. I will decide based on the dominant purpose of the text.

A third edge case will be between **Humor / Banter** and **Reaction / Hot Take**. Some comments insult a player or club in a way that could be either a joke or a serious opinion.

For example:
“Send him to the Saudi league immediately.”

If the surrounding tone is clearly joking or exaggerated, I will label it **Humor / Banter**. If it appears to be a serious criticism without evidence, I will label it **Reaction / Hot Take**.

## 4. Data Collection Plan

I will collect at least **200 public posts or comments** from r/soccer. I plan to collect mostly comments because comments usually contain more varied discourse than post titles alone.

I will collect examples from several types of threads:

* Post-match threads
* Match threads
* Daily discussion threads
* Transfer rumor threads
* Official news threads
* Player statistic threads
* Tactical discussion threads
* Major tournament or Champions League discussion threads

My target distribution is:

* 50 examples of **News / Information Update**
* 50 examples of **Evidence-Based Analysis**
* 50 examples of **Reaction / Hot Take**
* 50 examples of **Humor / Banter**

If one label is underrepresented after 200 examples, I will do targeted collection from thread types where that label is more likely to appear. For example, if I do not have enough **Evidence-Based Analysis**, I will collect more from tactical, statistics, or post-match discussion threads. If I do not have enough **Humor / Banter**, I will collect more from match threads, rivalry threads, or highly active post-match threads.

If a label is still underrepresented after targeted collection, I will review whether the label is too narrow. If it is too narrow, I may slightly revise the definition or merge it with a nearby label. However, I will avoid changing labels after annotation begins unless the label clearly fails to appear often enough or creates too much ambiguity.

I will avoid annotating AutoModerator comments, deleted comments, removed comments, pure URLs with no user-written text, and comments that require too much missing context to understand.

## 5. Evaluation Metrics

Accuracy alone is not enough for this task because the dataset may not be perfectly balanced, and some labels are harder to detect than others. A model could get high accuracy by doing well on common labels while performing poorly on more important labels like **Evidence-Based Analysis**.

I will use the following evaluation metrics:

### Accuracy

Accuracy will show the overall percentage of correct predictions. This is useful as a general performance measure, but it will not be the only metric.

### Macro F1 Score

Macro F1 will be my main metric because it treats each label equally. This matters because I want the model to perform reasonably well across all four discourse types, not just the most common one.

### Per-Label Precision and Recall

I will check precision and recall for each label. Precision matters because I want examples labeled as **Evidence-Based Analysis** to actually be substantive. Recall matters because I do not want the model to miss most of the real analysis examples.

### Confusion Matrix

I will use a confusion matrix to see which labels the model mixes up most often. I expect the most common confusion to be between **Evidence-Based Analysis** and **Reaction / Hot Take**, and between **Humor / Banter** and **Reaction / Hot Take**.

### Error Analysis

After evaluation, I will manually review incorrect predictions and group the errors into patterns. I will look for whether the model struggles with sarcasm, short tactical claims, transfer rumors with opinions, or jokes that look like serious criticism.

## 6. Definition of Success

A genuinely useful classifier for this task should be able to separate substantive football discourse from lower-effort reactions while also recognizing news and banter as their own categories.

My target success criteria are:

* At least **75% overall accuracy**
* At least **0.70 macro F1**
* At least **0.65 F1 for each individual label**
* No single label should have recall below **0.60**
* The confusion matrix should show that most errors happen in understandable edge cases, not randomly across all labels

I would consider the model “good enough” for a real community tool if it could reliably surface **Evidence-Based Analysis** while filtering out most low-effort reactions and jokes. For example, a useful classifier could help users find more thoughtful football discussion after a match or help moderators understand what kinds of discourse dominate certain threads.

I would not consider the model successful if it performs well overall but fails badly on **Evidence-Based Analysis**, because that label is the most important for identifying high-quality discussion.

## 7. AI Tool Plan

This project’s AI tool usage will focus on label design, annotation support, and failure analysis. I will not use AI to generate the actual dataset. The actual examples will come from public r/soccer posts and comments.

### Label Stress-Testing

Before annotating all 200 examples, I will give an AI tool my four label definitions and ask it to generate 5–10 borderline examples that could fit between two labels.

I will specifically ask for examples that sit between:

* **Evidence-Based Analysis** and **Reaction / Hot Take**
* **Humor / Banter** and **Reaction / Hot Take**
* **News / Information Update** and **Reaction / Hot Take**

If the AI produces examples that I cannot classify clearly, I will revise my label definitions before annotating the full dataset. This will help me catch label overlap early instead of discovering the problem after annotation.

### Annotation Assistance

I may use an LLM to pre-label a small batch of examples, but I will manually review every label myself. I will not accept AI labels automatically.

My annotation process will be:

1. Collect 200 public r/soccer examples.
2. Manually label the first 40 examples myself to test the label definitions.
3. Use an LLM to pre-label the remaining examples only as a suggestion.
4. Review every AI-suggested label manually.
5. Mark examples as “AI-assisted” if an LLM suggested the first label.
6. Keep a notes column for difficult examples.

I will track whether each example was:

* Manually labeled from scratch
* Pre-labeled by AI and accepted
* Pre-labeled by AI and changed
* Marked as uncertain

This will make my AI usage transparent and allow me to discuss whether AI helped or introduced bias.

### Failure Analysis

After training and evaluating the classifier, I will give the model’s wrong predictions to an AI tool and ask it to identify patterns in the mistakes. For example, I will ask whether the model is confusing sarcasm with hot takes, short analysis with unsupported opinions, or source-based transfer posts with user reactions.

I will not accept the AI’s failure analysis automatically. I will verify the patterns myself by reviewing the original examples and checking the confusion matrix. If the AI suggests a pattern that is not supported by the actual errors, I will not include it in the final evaluation.

The failure analysis will help me decide whether the model needs better labels, more examples, more balanced data, or improved preprocessing.

## 8. Stretch Feature Plan

I will update this `planning.md` before starting any stretch features. I will not add extra features until the basic dataset, labels, model, and evaluation are complete.

Possible stretch features could include:

* A confidence score for each prediction
* A filter that shows only **Evidence-Based Analysis** comments
* A dashboard showing the distribution of discourse types by thread
* A separate “uncertain” flag for borderline examples

These will only be considered after the core classifier is working and evaluated against the success criteria above.
