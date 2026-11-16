# LLM Name-Bias Audit: Red-Teaming a Resume Screener

Course paper project for **DS 203 Responsible AI** (Prof. Dr. Philipp Kellmeyer), University of Mannheim.    
Task: red-teaming an LLM.

## Research question

Do gender- and migration-associated names influence an LLM's suitability scores and interview recommendations for otherwise identical candidates applying for a junior accounting position in Germany?

## Methodological basis

The design adapts the name-swap audit methodology of Armstrong et al. (2024) to the German accounting sector.

- Armstrong, L., Liu, A., MacNeil, S., & Metaxa, D. (2024). The Silicon Ceiling: Auditing GPT's Race and Gender Biases in Hiring. arXiv:2405.04412. https://arxiv.org/abs/2405.04412

Adaptations relative to the original study (Study 1, resume assessment):

| Aspect | Armstrong et al. (2024) | This project |
|---|---|---|
| Labour market | United States | Germany |
| Occupation(s) | 10 occupations, job descriptions from the U.S. Bureau of Labor Statistics | 1 occupation: junior accountant (synthetic job description) |
| Resumes | 10 resumes (320 resume-name combinations in total) | 1 synthetic CV |
| Model | GPT-3.5 (`gpt-3.5-turbo-1106`) | Gemma 4 31B (`gemma-4-31b-it`) |
| Demographic dimensions | Race/ethnicity (4 groups) x gender (2) | Origin (German / migration-associated) x gender (2) |
| Names | 32 names (4 per cell); chosen not to suggest immigrant status | 8 names (2 per cell); origin groups include migration-associated names |
| Repetitions | 50 per resume-name pair (chosen after a pilot) | 5 per name (10 per group), 40 evaluations in total |
| Scale | 1-100 | 1-10 |
| Outcomes | Three separate prompts: overall rating, willingness to interview, hireability | One prompt: suitability score (primary), interview recommendation and explanation (secondary) |
| Explanations | Numeric scores only in Study 1 | Coded for unsupported assumptions (qualifications, language skills, cultural fit) |
| Analysis | Linear mixed-effects models (name as random effect) | _(to be decided; descriptive comparison with dispersion planned)_ |
| Study 2 (resume generation) | Included | Not replicated |


## Design

- **Method:** counterfactual name-swap audit. One synthetic job description and one synthetic CV are used for every evaluation. Only the candidate's name changes.
- **Conditions:** 2 x 2 (gender: male/female x name origin: German/migration-associated).
- **Names:** 2 names per condition (8 names in total) to reduce name-specific effects. See `data/names.csv`.
- **Repetitions:** 5 independent runs per name, i.e. 4 groups x 2 names x 5 runs = **40 evaluations**.
- **Primary outcome:** suitability score (1-10).
- **Secondary outcomes:** interview recommendation (yes/no) and a short free-text explanation. Explanations are coded for unsupported assumptions about qualifications, language skills, and cultural fit.
- **Model:** Gemma 4 31B (`gemma-4-31b-it`) via the Google AI Studio API.
- **Independence:** every evaluation is a separate API call with no shared conversation history.
- **Order:** the 40 calls are run in randomised order (seed recorded in the logs).

## Interpretation caution

This study reports **observed differences in model outputs** under controlled name changes. It does not by itself establish systematic discrimination. Group sizes are small (10 evaluations per group), only one model and one scenario are tested, and the causes of any difference (e.g. training data) cannot be identified here.

## Repository structure

```
.
├── README.md
├── .gitignore               # must include .env
├── .env.example             # GOOGLE_API_KEY=your_key_here (do not commit the real .env)
├── requirements.txt
├── data/
│   ├── job_description.md   # synthetic, identical for all runs
│   ├── cv_template.md       # synthetic, contains the {NAME} placeholder
│   └── names.csv            # name, gender, origin_group, group_id
├── prompts/
│   └── evaluation_prompt.txt
├── run_experiment.py        # builds prompts, calls the API, writes logs
├── logs/
│   └── run_<timestamp>.jsonl  # one line per evaluation (raw prompt + raw response)
├── analysis/
│   ├── results.csv          # parsed scores and recommendations
│   └── coding_scheme.md     # codebook for explanations (fixed before inspecting results)
├── appendix/                # transcripts referenced in the paper
└── notes/                   # course notes (own words), not part of the experiment
```

## Setup

```bash
git clone <this-repository-url>
cd <repository-folder>
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env             # then add your own API key to .env
```

## Running the experiment

```bash
# 1. Pilot: 1-2 calls to check that the JSON output parses correctly
python run_experiment.py --pilot

# 2. Full run: 40 evaluations in randomised order
python run_experiment.py
```

Expected model output per call (JSON only):

```json
{"suitability_score": 7, "interview": "yes", "explanation": "..."}
```

## What is logged

For every evaluation, one JSONL line with:

- timestamp, run ID, random seed, call index
- model ID and all generation parameters (temperature etc. as actually used)
- name, gender, origin group
- full prompt as sent
- raw model response
- parsed score, interview recommendation, explanation, parse status

## Documentation of settings

| Item | Value |
|---|---|
| Model | `gemma-4-31b-it` (Google AI Studio API) |
| Temperature / other parameters | _to be filled in after the pilot (default settings planned)_ |
| Scoring rubric (1 / 5 / 10) | _to be filled in, see `prompts/evaluation_prompt.txt`_ |
| Run date(s) | _to be filled in_ |
| Commit hash of the final run | _to be filled in_ |

## Ethics and data protection

- Only **synthetic** data are used: the CV, the job description, and the names do not refer to real applicants. No personal or sensitive data are entered into the model.
- The experiment follows the course's ethics guidelines for red-teaming (transparent documentation, no illegal or highly sensitive topics, focus on learning rather than provocation).
- The API key is stored only in a local `.env` file and is never committed.

## Reproducibility

The job description, CV, and prompt are identical across conditions; only the name differs. Model outputs are stochastic, so re-running will not reproduce exact scores. The committed logs are the record of the results reported in the paper.

## Author

Kyoungmi Kwon, M.Sc. Business Informatics, University of Mannheim.
