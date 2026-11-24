# Fraud track: spotting card fraud, with reasons

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/snow10100/aie113-projects/blob/main/tracks/fraud/notebook.ipynb)
· [بالعربية](README.ar.md)

## 1. What this model does

The model looks at one card payment and gives it a **fraud score** between 0 and 1. When the
score reaches the **alert threshold** (0.4 in the notebook), the payment goes to the fraud team
for a check, together with the **top reasons** behind the score. Think of a security guard at a
shop door who has watched thousands of customers: almost everyone walks past, but when someone
behaves like past shoplifters, the guard asks them to step aside and says what caught their
eye. The threshold is the guard's sensitivity: stricter means fewer thieves get through, but
more honest customers are stopped.

## 2. The data

- **What it is:** 284,807 real card payments made by European cardholders over two days in
  September 2013. 492 of them were fraud: about 1 in 600.
- **Where it comes from:** collected by Worldline and the Machine Learning Group of ULB
  (Université Libre de Bruxelles). It is published on
  [Kaggle](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) and on
  [OpenML (dataset 1597)](https://www.openml.org/d/1597). The notebook downloads the original
  CSV file (150 MB) from the
  [public copy](https://storage.googleapis.com/download.tensorflow.org/data/creditcard.csv) that
  Google's TensorFlow team keeps for its tutorials: no account, and fast, also from Colab. If
  that address ever fails, OpenML has the same data. Nothing is stored in this repository.
- **Licence:** the original release is under the Open Database License (ODbL 1.0) for the
  database and the Database Contents License (DbCL 1.0) for its contents; OpenML lists it as
  public. Please cite: A. Dal Pozzolo, O. Caelen, R. A. Johnson and G. Bontempi, *Calibrating
  Probability with Undersampling for Unbalanced Classification*, IEEE CIDM 2015.
- **One row is one payment:**
  - `Time`: seconds since the first payment in the file.
  - `Amount`: the amount paid (the dataset does not name the currency).
  - `V1` to `V28`: 28 anonymous numbers. To protect customers, the bank mixed the real details
    of each payment into 28 new numbers, so outsiders cannot recover the originals. Nobody
    outside the bank knows what any of them means.
  - `Class`: 1 for fraud, 0 for a normal payment. This is the answer the model learns to give.

## 3. How the model learns

The model is **gradient boosting**. It builds 300 small decision trees; each tree asks a few
yes-or-no questions such as "is V14 below -3?" and gives a small vote. Each new tree
concentrates on the payments the earlier trees got wrong, and the fraud score combines all the
votes. Its inputs are V1 to V28, the amount, and the hour of the day (made from `Time`, assuming
the file starts at midnight: the quiet hours fall at night).

The payments are split **by time**, never shuffled, like a bank that learns from the past and is
judged on the future:

| Part | Hours of the two days | Payments | Frauds | Used for |
|---|---|---|---|---|
| Training | 0 to 33.4 | 170,884 | 360 | learning |
| Validation | 33.4 to 40.3 | 56,961 | 57 | choosing the threshold |
| Test | 40.3 to 48.0 | 56,962 | 75 | the final, honest score |

Training takes a few seconds on a laptop, so it uses every payment; there is no need to throw
away normal payments (down-sampling). The **threshold** is chosen on the validation part: the
fraud team's rule is "at least 8 of every 10 alerts must be real fraud", and the notebook keeps
the lowest threshold that meets it.

**Baselines** (simple methods to beat): always answering "not fraud", and a hand-written rule
"alert on every payment above 1,000".

**How the reasons are made.** For one flagged payment, the notebook asks, input by input: *what
if this value were typical?* It replaces that one value with its usual value among normal
payments (the median) and scores the payment again. If the score falls a lot, that input was a
strong reason for the alert. This check works with any model, because it only needs the model's
scores. For the overall picture, the notebook shuffles one input at a time, so that it carries
no information, and measures how much worse the model gets (permutation importance).

## 4. How we know if it is any good

**Accuracy is misleading here.** Because fraud is so rare, a "model" that always answers "not
fraud" is right 99.83% of the time and catches nothing. So we use two plain questions instead:

- **Of the alerts we raise, how many are really fraud?** (precision) This is about not wasting
  the fraud team's time or bothering honest customers.
- **Of all frauds, how many do we catch?** (recall) This is about protecting customers' money.

Moving the threshold trades one for the other. On the validation payments:

| Threshold | Alerts | Real frauds among them | Alerts that are fraud | Frauds caught (of 57) |
|---|---|---|---|---|
| 0.01 | 96 | 43 | 45% | 75% |
| 0.10 | 61 | 42 | 69% | 74% |
| **0.40** (chosen) | 51 | 41 | 80% | 72% |
| 0.90 | 39 | 38 | 97% | 67% |
| 0.99 | 23 | 23 | 100% | 40% |

**The real numbers, on the 56,962 test payments the model never saw (75 frauds):**

| Method | Accuracy | Alerts | Frauds caught | Alerts that are fraud | Share of frauds caught |
|---|---|---|---|---|---|
| Always "not fraud" | 99.87% | 0 | 0 of 75 | no alerts | 0% |
| Rule: amount above 1,000 | 98.99% | 503 | 2 of 75 | 0.4% | 2.7% |
| **Model, threshold 0.4** | 99.95% | 65 | 56 of 75 | **86%** | **75%** |

What this means for a user: when the fraud team receives an alert, it is real fraud about 6
times out of 7. Three out of four frauds are caught; one in four still gets through. Only 9 of
the 56,887 honest payments were wrongly flagged. The frauds it caught add up to 5,086 of the
7,729 paid in all the frauds of that period (66%). The "big amount" rule fails because most frauds here are small
amounts. The notebook also reports the **average precision** (0.79), one number that judges the
whole ranking of scores (1.0 is perfect).

Take these numbers as rough: the test part has only 75 frauds, so one fraud more or less moves
"frauds caught" by more than one point. A different scikit-learn version can give slightly
different numbers (with scikit-learn 1.6 we saw 67 alerts, also catching 56 frauds).

## 5. Limits and risks

- **Old and narrow data:** two days, one issuer, 2013. Fraudsters change their tricks, so a real
  model is retrained and watched over time.
- **It misses about one fraud in four**, and every false alarm is a real customer who gets a
  call or a blocked card.
- **The inputs only exist in this dataset.** V1 to V28 come from the bank's private
  transformation, so the service can only score payments already written in this format. A real
  system would compute its inputs from raw payment details.
- **The reasons are honest but not useful to a customer.** "V14 is unusually low" means nothing
  to anyone outside the bank. A real bank would show named reasons, such as "amount far above
  this customer's usual", "first payment in a new country" or "five payments in two minutes".
  The what-if check also changes one input at a time, so it can miss reasons that only matter
  in combination.
- **The score is a ranking, not a promise.** Treat 0.9 as "more suspicious than 0.5", not as
  "exactly 90% sure".
- **Never use it** to block cards or accuse anyone without a human check, to judge a person
  (credit, employment, anything beyond this dataset), or on real payments: it is a teaching model
  trained on 2013 data.

## 6. What the notebook does, step by step

1. Sets the constants: data address, file paths, split shares, the fraud team's rule.
2. Downloads the data once into `data/` and loads it.
3. Takes a quick look: how rare fraud is, and why accuracy misleads.
4. Turns `Time` into the hour of the day and splits the payments by time.
5. Trains the gradient boosting model on the training part.
6. Tries several thresholds on the validation part, shows the trade-off, and picks one.
7. Scores the test part and compares the model with the two baselines.
8. Explains one real alert and one false alarm with the "what if typical?" check.
9. Shows which inputs matter most overall.
10. Saves the model and its facts into `artifacts/`.
11. Reloads them and scores one new payment, as the service will.

## 7. What the notebook leaves in `artifacts/`

- `fraud_model.joblib` (about 600 KB): the trained model.
- `model_info.json`: the model version, the scikit-learn version, the input names in order, the
  threshold and the rule that chose it, the typical value of each input (needed for the
  reasons), the data size and training hours, and the test numbers with the baselines.

The download stays in `data/creditcard.csv` (150 MB). Both folders are ignored by git.

## 8. Project ideas for this track

**The API.** A request carries one payment: `Time` (or, better, a real timestamp), `Amount`
and `V1` to `V28`. The response gives the fraud score, a decision, the threshold used, the
model version, and the top three reasons, each with the input's name, its value, its typical
value and the score if it were typical. The guide asks for three decisions (allow, review,
block): the saved threshold can mean "review", and your group can add a higher "block"
threshold, which is a business rule to agree on and justify.

**Tests that matter here.** Making a flagged payment's top reason typical must lower its score.
The decision must flip exactly at the threshold. On a saved set of reference payments, the
model must still catch at least an agreed share of frauds (for example 70%) with at least an
agreed share of alerts being fraud. The payment from the notebook's last cell must get the same
score and the same top reason in the service (a golden check). A payment with a missing V
number or a negative amount must be rejected.

**A secret.** A token for an alerting webhook that notifies the fraud team when a payment is
blocked. It must never appear in the code, the image or the logs.

**Front end idea.** A small "fraud desk" page: pick a sample payment, see its score as a gauge,
the decision, and the top reasons as bars; a slider moves the threshold and shows how many
alerts and caught frauds it would give on the test payments.

## 9. How to open it

**Colab:** click the badge at the top, then *Runtime → Run all* (about 2 minutes, most of it the
download).

**Local Jupyter** (Python 3.10 to 3.12), from this folder:

    python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
    pip install -r requirements.txt jupyter
    jupyter notebook notebook.ipynb

If training seems to hang on a machine with many cores (common under WSL), run
`export OMP_NUM_THREADS=1` first (PowerShell: `$env:OMP_NUM_THREADS=1`).
