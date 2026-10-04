# Model Card: Mood Machine

## 1. Model Overview

**Model type:**  
I implemented and compared both a rule-based classifier and a machine-learning classifier.

**Intended purpose:**  
The Mood Machine clasifies short text posts as `positive`, `negative`, `neutral`, or `mixed`.

**How it works (brief):**  
The rule-based model lowercases and splits text into tokens. It adds one point for positive words, subtracts one point for negative words, and reverses a word’s contribution when it follows `not`, `never`, or `no`. A positive score becomes `positive`, a negative score becomes `negative`, and zero becomes `neutral`.

The ML model uses `CountVectorizer` to represent words in the labeled posts, then trains a logistic-regression classifier to predict the mood label.

## 2. Data

**Dataset description:**  
`SAMPLE_POSTS` contains 14 short posts, with one correspondng label per post in `TRUE_LABELS`. I added eight posts containing informal language, emojis, sarcasm, mixed emotions, and ambiguous tone.

**Labeling process:**  
I assigned a label at the same time as each new post so that the two lists stayed aligned. Some posts were difficult to label because they contained more than one emotion. For example, `"Lowkey stressed but proud I finished the project"` was labeled `mixed` because it expresses both stress and pride.

**Important characteristics of the dataset:**  

- It contains slang, such as `"No cap"` and `"cooked"`.
- It includes sarcasm, such as `"Love spending an hour stuck in traffic 🙃"`.

**Possible issues with the dataset:**  
The dataset is very small and may not represent many writing styles, regional expressions, languages, or longer posts. Labels for sarcasm and mixed emotions can also be subjective.

## 3. How the Rule Based Model Works

**Your scoring rules:**  

- Positive words add 1 to the score.
- Negative words subtract 1 from the score.
- `not`, `never`, and `no` negate the next recognized positive or negative word.
- I added `"fire"` to the positive-word list as a targeted slang improvement.
- Scores greater than zero are `positive`, below zero are `negative`, and zero is `neutral`.

**Strengths of this approach:**  
The rules are transparent and easy to explain. They correctly classified straightforward examples such as `"I love this class so much"` as `positive`, `"Today was a terrible day"` as `negative`, and `"I am not happy about this"` as `negative`.

**Weaknesses of this approach:**  
The model relies on a limited word list and does not understand context well. It cannot reliably recgonize sarcasm, mixed moods, unknown slang, or emoji-only communication.

## 4. How the ML Model Works

**Features used:**  
The ML model uses a bag-of-words represenation created with `CountVectorizer`.

**Training data:**  
The model trained on the same 14 posts in `SAMPLE_POSTS` and labels in `TRUE_LABELS`.

**Training behavior:**  
After adding more labeled posts, the ML model reported 1.00 accuracy on the dataset it trained on. This result is training accuracy, not evidence that the model will perform as well on unseen posts.

**Strengths and weaknesses:**  
The ML model can learn word-label patterns without manually writing a rule for every word. However, with only 14 examples, it can overfit and may learn accidental associations instead of understanding meaning or sarcasm.

## 5. Evaluation

**How I evaluated the model:**  
I ran both classifiers against the labeled posts in `dataset.py`.

- Rule-based accuracy: **0.57**
- ML accuracy on the training dataset: **1.00**

**Examples of correct predictions:**  

- `"I love this class so much"` was correctly labeled `positive` by the rule-based model because it includes `love`.
- `"I am not happy about this"` was correctly labeled `negative` because the negation rule reverses the positive word `happy`.
- `"No cap, that concert was amazing 😂"` was correctly labeled `positive` because `amazing` is in the positive-word list.

**Examples of incorrect predictions:**  

- `"Love spending an hour stuck in traffic 🙃"` was predicted as `positive` by the rule-based model but labeled `negative`. The rule counted `love` and did not understand the sarcasm.
- `"My phone died again :("` was predicted as `neutral` by the rule-based model but labeled `negative`. None of its words appeared in the negative-word list.
- `"Feeling tired but kind of hopeful"` was predicted as `negative` by the rule-based model but labeled `mixed`. The simple scoring approach could not represent both feelings.

The ML model labeled the traffic example as `negative`, unlike the rule-based model. However, that post was included in its training data, so this does not prove it understands sarcasm.

## 6. Limitations

The rule-based model depends on a small manually selected vocabulary. For example, adding `"fire"` helps classify `"That concert was fire"` as positive, but it can create a new error by labeling `"The building is on fire"` as positive.

Neither model reliably understands sarcasm, mixed emotions, unfamiliar slang, or context. The ML model was evaluated on the same small dataset used for training, so its 1.00 accuracy is overly optimistic.

## 7. Ethical Considerations

A mood classifier could cause harm if it misclassifies a message expressing distress, frustration, or sarcasm. The model may under-serve people who use informal phrasing, regional expressions, non-standard spelling, emojis, or slang that is missing from the dataset.

During review, the AI assistant helped identify gaps in sarcasm, slang, emoji, and mixed-emotion coverage. It should also push back on overly broad conclusions from the ML model’s training accuracy and on narrow word-list fixes that may create new context errors.

Personal text can also be sensitive, so a real mood-detection system should protect privicy and should not be used alone for high-stakes decisions.

## 8. Ideas for Improvement

- Collect a larger and more diverse labeled dataset.
- Add a separate test set that is never used for training.
- Add more examples of sarcasm, emojis, slang, mixed emotions, and literal uses of ambiguous words.
- Improve preprocessing so it recognizes emojis and punctuation.
- Add more careful handling of phrases and context instead of individual words alone.
