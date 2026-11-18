# Smart Resume Screening and Candidate Ranking Tool

An NLP-based CLI tool that parses resumes (`.pdf` / `.docx` / `.txt`), extracts
structured candidate profiles, and ranks candidates against a job description
using a weighted blend of text similarity, skill overlap, experience, and
education fit.

## How it works

```
resumes (.pdf/.docx/.txt)          job_description.txt
        |                                  |
        v                                  v
   parser.py                    main.py parses optional
  (extract raw text)             REQUIRED_SKILLS / MIN_YEARS /
        |                        MIN_EDUCATION header
        v                                  |
   extractor.py                            |
  (name, email, phone, years               |
   experience, education, skills)          |
        |                                  |
        +------------------+---------------+
                           v
                      ranker.py
          TF-IDF + cosine similarity (text match)
                + skill overlap %
                + experience fit %
                + education fit %
                = weighted final_score (0-100)
                           |
                           v
                 ranked candidate list
             (console, CSV, and/or JSON)
```

### Scoring breakdown (default weights)

| Signal          | Weight | What it measures                                      |
|-----------------|--------|--------------------------------------------------------|
| Text similarity | 40%    | TF-IDF cosine similarity between resume and job text    |
| Skill match     | 35%    | % of required skills found in the resume                |
| Experience fit  | 15%    | Candidate years vs. minimum years required               |
| Education fit   | 10%    | Candidate degree level vs. minimum required level        |

Weights are configurable in `ranker.py` (`DEFAULT_WEIGHTS`), or can be passed
programmatically to `rank_candidates(..., weights={...})`.

## Setup

```bash
pip install -r requirements.txt
```

## Usage

```bash
python main.py --job job_description.txt --resumes resumes/ --top 5
```

Optional flags:

- `--out-csv results.csv` — write full ranked results to CSV
- `--out-json results.json` — write full ranked results to JSON
- `--top N` — number of top candidates to print (default 10)

### Job description format

The job description file can optionally start with a metadata header:

```
REQUIRED_SKILLS: python, sql, machine learning
MIN_YEARS_EXPERIENCE: 3
MIN_EDUCATION: bachelor
---
<free-text job description goes here>
```

If the header is omitted, the tool ranks purely on text similarity with no
hard requirements.

## Project structure

```
resume_screener/
├── main.py               # CLI entry point
├── parser.py              # PDF/DOCX/TXT text extraction
├── extractor.py            # name, email, phone, skills, experience, education
├── ranker.py                # TF-IDF similarity + weighted scoring
├── requirements.txt
├── job_description.txt     # sample job description
└── resumes/                 # sample resumes (mixed file types, varied match strength)
```

## Extending it

- **Skill vocabulary**: edit `DEFAULT_SKILLS` in `extractor.py`, or pass a
  custom list into `extract_profile(text, skill_vocab=[...])`.
- **Scoring weights**: edit `DEFAULT_WEIGHTS` in `ranker.py`.
- **Better semantic matching**: swap the TF-IDF vectorizer in
  `compute_text_similarity` for sentence embeddings (e.g.
  `sentence-transformers`) if you need to catch synonyms/paraphrasing that
  keyword-based TF-IDF misses.

## Notes / limitations

- Text extraction quality depends on how the source PDF/DOCX is formatted
  (scanned/image-only PDFs won't extract text without OCR).
- Skill and education detection are keyword/pattern-based, not a full
  semantic parser — for production use, expand the vocab and review edge
  cases.
- This is a decision-support tool, not an autonomous hiring decision-maker;
  a human should always review shortlists, particularly to check for bias in
  the underlying resume data and job requirements.
##Output
<img width="1072" height="652" alt="Output" src="https://github.com/user-attachments/assets/7ba3fe35-2517-4fa9-81ec-f79bcfabd816" />

