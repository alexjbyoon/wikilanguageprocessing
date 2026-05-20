# 📖 Wikipedia Biography NLP — Timeline & Location Extraction

> Extracting structured life event timelines and geographic locations from Wikipedia biographies using spaCy dependency parsing and named entity recognition.

---

## 🧩 Problem Statement

Wikipedia biographies are written in natural language — rich with dates, events, and places — but entirely unstructured. Turning a paragraph like *"Bowie was born on 8 January 1947 in Brixton"* into a structured data point (date: 1947, subject: Bowie, event: born, location: Brixton) requires understanding grammar, not just keywords.

This project asks: **can we automatically extract structured timelines of key life events from Wikipedia biography text using NLP dependency parsing and NER?**

Specifically:
- Who is the subject of each sentence?
- What did they do (verb/root)?
- What are they acting on (object)?
- When did it happen (date)?
- Where did it happen (location)?

---

## 🗂️ Dataset

| File | Contents |
|---|---|
| `bios.tsv.gz` | Compressed TSV of Wikipedia biography articles, indexed by person title with a `gender` column |
| `bios.spacy` | Pre-parsed spaCy `DocBin` file — serialized spaCy `Doc` objects for each biography, keyed by index |

The dataset covers multiple public figures; example analysis is demonstrated on **David Bowie**.

---

## 🔧 Approach

### 🧠 spaCy Dependency Parsing

Each sentence is parsed with spaCy's `en_core_web_sm` model to extract its grammatical structure. Four custom functions build on spaCy's dependency tree:

| Function | What it extracts |
|---|---|
| `get_root(sent)` | The main verb of the sentence |
| `get_subj(sent)` | The subject (`nsubj`, `nsubjpass`, `csubj`) |
| `get_obj(sent)` | The direct/indirect object (`obj`, `dobj`, `iobj`, `attr`) |
| `get_pobjs(sent)` | All prepositional objects as `(prep, pobj)` tuples |

Subjects and objects are then expanded to their full **noun chunk** using `get_noun_chunk()`, which finds the containing noun phrase in spaCy's chunker (e.g. `"his Ziggy Stardust stage show"` instead of just `"show"`).

### 📅 Date Extraction

`get_date(sent)` scans spaCy's named entities for `DATE` labels that contain exactly one 4-digit year using regex. This filters out vague date references (e.g. *"the late 1960s"*) and returns only specific, year-anchored dates — the ones most useful for timeline construction.

### 🙋 Subject Resolution

To filter sentences to only those actually about the biography subject (not a passing mention of someone else), `get_name_tokens()` builds a set of valid references for each person — their first name, last name, and gendered pronoun (`he`/`she`). `is_mention()` checks whether the parsed subject token matches any of these.

### 📅 Timeline Construction

`get_timeline_sents()` filters each biography's sentences to only those that:
1. Have a subject that refers to the biography subject
2. Contain a year-anchored date

These filtered sentences form the **timeline** — structured as (date, subject, verb, object, prepositional objects) tuples, ready for display or downstream use.

### 📍 Location Extraction

`get_locations()` scans timeline sentences for spaCy `GPE` (geopolitical entity) named entities — cities, countries, regions — and returns the unique set of places mentioned in that person's life events.

---

## 📊 Key Findings (David Bowie Example)

- The pipeline successfully extracts year-anchored timeline sentences from Bowie's biography, including events like his 1972 Ziggy Stardust launch in Kingston and his 1974 move to New York City
- Noun chunk expansion significantly improves readability — subjects and objects are returned as full meaningful phrases rather than single head tokens
- `GPE` NER reliably surfaces key locations: **Brixton, Kingston, New York City** appear correctly extracted from timeline sentences
- Sentences where Bowie is referenced only by pronoun (`he`) are correctly captured via the name token set — pronoun resolution without a full coreference model

---

## 🗃️ Repository Structure

```
wikilanguageprocessing/
├── processing.ipynb     # Full NLP pipeline notebook
├── bios.tsv.gz          # Compressed biography dataset
└── bios.spacy           # Pre-parsed spaCy DocBin file
```

---

## 🚀 How to Run

```bash
# 1. Clone the repo
git clone https://github.com/alexjbyoon/wikilanguageprocessing.git
cd wikilanguageprocessing

# 2. Install dependencies
pip install pandas numpy spacy jupyter

# 3. Download the spaCy English model
python -m spacy download en_core_web_sm

# 4. Launch the notebook
jupyter notebook processing.ipynb
```

---

## 📦 Requirements

| Package | Purpose |
|---|---|
| `pandas` | Loading and indexing the biography TSV |
| `numpy` | Index alignment for DocBin loading |
| `spacy` | Dependency parsing, NER, noun chunking |
| `re` | Year pattern matching in date extraction |
| `jupyter` | Notebook environment |

Python 3.8+ recommended. spaCy model `en_core_web_sm` required.
