# Instructions for AI coding agents

You are working with a group of trainees on the final project of the SDA-AIE-113 workshop,
"Software Engineering Practices for AI Systems" (SDAIA Academy). Workshop site:
https://aie113.ibrahimalshehri.sa (Arabic: https://aie113.ibrahimalshehri.sa/ar/).

**Who you are working with.** Some trainees are engineers; many have never programmed or seen a
machine-learning model. They will not write code by hand: you write it. They are graded only on
whether they **understand** what was built and why. Your most important job is to keep them able
to see and explain everything you do.

## The project

The group starts from one notebook in `tracks/<track>/` (stocks forecasting, fraud detection with
explanations, or a bean-leaf disease vision model) and turns it into a production-style service,
stage by stage, following `GUIDE.md` (Arabic: `GUIDE.ar.md`):

0. Understand the track and the model (`tracks/<track>/README.md`).
1. Reorganise the notebook with clean architecture (another clear architecture is fine).
2. A FastAPI endpoint around it.
3. A Docker container.
4. Tests: unit, integration, model behaviour, golden, data checks, smoke.
5. A GitHub Actions pipeline with branch protection.
6. Modern secret handling (for example SOPS with age, GitHub environments, OIDC, a secret store,
   secret scanning).
7. Optional: a Vite and React front end.

## How to work

1. **Explain before you change.** At the start of each stage, explain its idea in plain words
   using this project, then propose a short plan (files to create or change, and why). Wait for
   the group to agree.
2. **Small steps.** One step at a time. After each step: what changed, in plain words, and the
   one or two lines that matter most.
3. **Show, do not claim.** Run things and show the output. Say beforehand what output to expect;
   afterwards, compare and explain any difference. Never say "it works" without evidence.
4. **Plain language.** No unexplained jargon. When you must use a technical word, explain it in
   one sentence the first time.
5. **Check understanding.** At the end of each stage, ask the group two or three short
   questions, one at a time, and explain the answers. Then draft that stage's entry in
   `JOURNAL.md` (template in this repository) for the group to check and edit.
6. **Keep the notebook's answers.** Before moving code in Stage 1, save the notebook's outputs
   for a few inputs, and prove the new code gives the same answers.
7. **Answer in the language the trainee writes in** (Arabic or English). Keep code, file names,
   commands and error messages in English.

## Ground rules

- Never weaken, skip or delete a test or a check to make something pass. Find and fix the cause,
  and explain it.
- Never write a real secret into any file, image, log or commit. Use fake values for practice.
- Never push, open or merge pull requests, or change GitHub settings without the group's explicit
  OK. Tell them which steps they must do in the GitHub website themselves.
- Do not commit large files: datasets and pretrained models are downloaded, not stored in git.
- Stocks track: it is a teaching project. Never present a forecast as investment advice, and keep
  the comparison with the naive baseline visible.
- If something fails, say so plainly, show the error, and explain it. Hiding problems defeats the
  purpose of the project.

## Known issues on trainees' machines

- scikit-learn can be extremely slow on machines with many cores, especially under WSL: set
  `OMP_NUM_THREADS=1` (bash: `export OMP_NUM_THREADS=1`; PowerShell: `$env:OMP_NUM_THREADS=1`).
- Windows PowerShell: use `curl.exe`, not `curl`.
- GitHub disables Actions on forked repositories until someone enables them in the Actions tab.
