# Track `vision`: spotting disease on bean leaves in field photos

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/snow10100/aie113-projects/blob/main/tracks/vision/notebook.ipynb)
· [بالعربية](README.ar.md)

This track uses a real, published image model and real photos taken in bean fields in Uganda.
Nobody trains anything: the notebook downloads the model, checks how good it is on photos it has
never seen, shows where in each photo it looked, and saves what a service needs.

## 1. What this model does

You give the model a photo of a bean leaf. It answers with one of three labels, **healthy**,
**angular leaf spot** or **bean rust** (two common bean diseases), and with how sure it is, for
example "bean rust, 95%". Think of a farm advisor who has studied about a thousand labelled leaf
photos: show them a new photo and they name what they see, quickly, without any lab test. Like
that advisor, the model can be wrong, and it only knows these three answers.

## 2. The data

- **What:** the *beans* dataset, photos of bean leaves taken with a basic smartphone in fields in
  several districts of Uganda. Experts from Uganda's National Crops Resources Research Institute
  (NaCRRI) labelled each photo in the field. Collected by the Makerere AI Lab, released in 2020.
- **Where:** [AI-Lab-Makerere/beans](https://huggingface.co/datasets/AI-Lab-Makerere/beans) on
  Hugging Face (original: [github.com/AI-Lab-Makerere/ibean](https://github.com/AI-Lab-Makerere/ibean)).
  **Licence: MIT.** It downloads without an account.
- **How big:** 1,295 photos of 500 × 500 pixels: 1,034 for training, 133 for validation and 128
  for testing. The notebook only downloads the **128 test photos** (18 MB): 43 angular leaf spot,
  43 bean rust and 42 healthy.
- **One example** is one photo (a leaf, often with other leaves and soil behind it) and the label
  the expert gave it.

**The model:** [nateraw/vit-base-beans](https://huggingface.co/nateraw/vit-base-beans) on Hugging
Face, published in 2021. **Licence: Apache-2.0.** It also downloads without an account. It is
343 MB: 86 million numbers.

## 3. How the model learns

We do not train it, but this is how its author did. The model is a *Vision Transformer* (ViT):
it cuts a photo into 196 small squares and learns which squares, and which combinations of
squares, point to each answer. It first learned to "see" in general from about 14 million
everyday photos (a Google model called `google/vit-base-patch16-224-in21k`). Then it was shown the
1,034 labelled bean photos a few times, and each time it was wrong it adjusted its numbers a
little (this is called *fine-tuning*). Before it looks at a photo, the photo is always resized to
224 × 224 pixels and its colours are turned into numbers between -1 and 1. A service must do
exactly the same.

**The baseline** is a "model" that ignores the photo and always gives the most common answer. On
our test photos it is right 33.6% of the time. Any useful model must beat it by far.

## 4. How we know if it is any good

**The metric is accuracy:** of all test photos, the share where the model agrees with the expert.
These photos were never used for training, so this is a fair check.

The notebook's real numbers (128 test photos, CPU):

| | Accuracy |
|---|---|
| Baseline (always "angular_leaf_spot") | 33.6% |
| **vit-base-beans** | **96.9%** (124 of 128 right) |

In words: the model got all 43 bean-rust photos right, 41 of 42 healthy leaves (it called one
bean rust) and 40 of 43 angular-leaf-spot leaves (it called three bean rust). The model's page
reports 94.5% on the same test photos, measured automatically by Hugging Face in 2022 with older
tools. The difference is 3 photos; we checked that both photo-preparation methods available today
give 96.9%.

**What this means for a user:** about 1 photo in 32 gets the wrong label. All four mistakes were
"bean rust" answers: the model never missed a rust leaf and never called a sick leaf healthy, so
it errs on the side of caution. Its confidence helps: it averaged 94% on right answers and 81% on
wrong ones, and this model never goes above 96%, so about 95% is its normal "sure". A "not sure"
rule below 80% would turn two of the four mistakes into "not sure" without losing a single right
answer. But the other two mistakes were 94% sure, as sure as a typical right answer.

### Why did the model say that?

The notebook uses an **occlusion map**. It covers one small grey square of the photo, asks the
model again, and repeats this for 169 positions across the photo. Where hiding a part of the photo
makes the model much less sure, that part mattered for the answer; where hiding it changes nothing,
the model did not rely on it. The notebook paints the important parts blue.

For most photos the model is about 95% sure, and covering any one square changes almost nothing,
because it finds what it needs in many places at once. So the notebook explains two harder cases.
For a leaf with angular leaf spot that the model got right but was only 81% sure about, just 13 of
the 169 squares matter: a small area in the upper part of the leaf, where there are a few brown
spots. Covering it drops the confidence to 30%. For its first mistake (angular leaf spot called
bean rust, 75% sure), 70 of the 169 squares matter: the model relied on a wide area, strongest on
the upper leaf at the top of the photo, not on the clear brown spots of the leaf in the middle.
This shows *what changes the answer*, not *how the model thinks*, and it is slow (169 extra model
runs per photo), so a service would offer it as a separate, optional request.

## 5. Limits and risks

- **Only three answers.** A leaf with another disease, another crop, or a photo of a hand still
  gets one of the three labels. The model has no "I don't know" unless the service adds one (for
  example "not sure" below a confidence level).
- **One place, one kind of photo.** All photos come from fields in Uganda, taken with smartphones
  in daylight. Other bean varieties, other countries, other cameras or unusual light can make it
  less accurate, and we have not measured by how much.
- **A small test.** 128 photos is few: one more or one fewer mistake moves the accuracy by almost
  one point.
- **Confidence is not proof.** Two of the four mistakes were 94% sure, as sure as a typical right answer.
- **Never use it alone** to decide to spray pesticides, destroy a crop, refuse an insurance claim
  or certify seed for sale or export. It is a first screening tool; an agronomist confirms.

## 6. What the notebook does, step by step

1. Installs the `datasets` library if it is missing, and sets the constants and random seeds.
2. Downloads the 128 test photos and counts the labels.
3. Downloads the published model and its photo preparation (224 × 224 pixels, colours scaled).
4. Shows 6 photos with the expert's label and the model's answer.
5. Scores all 128 test photos: accuracy, the baseline, and confidence when right and when wrong.
6. Counts the mistakes by label (a confusion table) and says it in words.
7. Builds an occlusion map ("where did it look") for the right answer it was least sure about and for its first mistake.
8. Saves the model and a small JSON with the numbers into `artifacts/`.
9. Reloads the saved model and scores one photo, the way a service would.

## 7. What the notebook leaves in `artifacts/`

| File | What it is |
|---|---|
| `artifacts/model/model.safetensors` | The model's 86 million numbers (about 343 MB). |
| `artifacts/model/config.json` | The model's settings, including the three labels in order. |
| `artifacts/model/preprocessor_config.json` | The photo preparation: size 224 × 224 and the colour scaling. |
| `artifacts/model_info.json` | Model version, the original model and its exact revision, licences, labels, input size, the test numbers and the confusion table. |

`artifacts/` is ignored by git: the model is far too large for the repository. Deciding where
your service gets the model from (copied into the container, or downloaded when it starts) is
part of Stage 3.

## 8. Project ideas for this track

**The API.** `POST /predict` receives one photo file (JPEG or PNG). It answers with the label,
the confidence, the scores of all three labels, a "not sure" flag when the confidence is below an
agreed level (the notebook tries 80%), and the model version. Files that are not images, or are
too large, are rejected with a clear message. A second request, `POST /explain`, could return the
occlusion map as a picture, since it takes much longer.

**What makes the tests interesting.** The answers come from a large model, so tests check
*behaviour*, not exact numbers. Ideas:
- The service prepares a photo exactly like the notebook (same size, same colour numbers), so the
  same photo gets the same answer as in the notebook.
- A slightly brighter, slightly rotated or mirrored photo keeps its label. Test a share, not every
  photo: we tried it on the 128 test photos and 1 to 3 of them changed their answer.
- A plain grey picture or random noise gets "not sure". The model calls such pictures "healthy",
  but only about 35 to 40% sure, so the "not sure" rule is what protects the user here.
- On a small set of reference photos, accuracy stays above an agreed level (for example 90%).

**A secret.** A Hugging Face access token (`HF_TOKEN`), needed if your group keeps its own private
copy of the model on Hugging Face. It must never appear in the code, the image or the logs.

**The container.** Full PyTorch makes the image very large. A service could convert the model to
ONNX and run it with the much smaller ONNX Runtime; compare the image sizes and check that the
answers stay the same.

**The optional React front end.** A phone-friendly page: take or upload a photo of a leaf, see the
label with a confidence bar, and a button that shows the "where it looked" map over the photo.

## 9. How to open it

- **Colab:** [open the notebook in Colab](https://colab.research.google.com/github/snow10100/aie113-projects/blob/main/tracks/vision/notebook.ipynb)
  and choose *Runtime → Run all*. A CPU is enough; it takes about 3 to 5 minutes, most of it downloading
  the model (343 MB) and making the occlusion maps.
- **Local Jupyter** (Python 3.10 to 3.12), from this folder:

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt jupyter
jupyter notebook notebook.ipynb
```

Without a GPU, first run `pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu`
to get a much smaller PyTorch. A warning about `HF_TOKEN` when the notebook downloads is harmless.
