# Contract 2.0

Contract 2.0 is an experimental Jupyter-based workflow for analyzing an
employment contract from a job applicant's perspective. It uses the OpenAI
Agents SDK to:

1. convert applicant requirements into structured data;
2. extract structured facts from a contract and a CV;
3. compare the contract with the CV;
4. compare the contract with the applicant's requirements; and
5. generate Markdown reports containing mismatches, risks, and questions for a
   contract discussion.

The project is currently implemented as a notebook rather than a command-line
application or reusable Python package. 

costed me 8 cents for gh copilot and 20 cents OPENAI API calls, compared to 20$ and 28 cents for contract1.0 version:)

## Project structure

```text
contract2.0/
|-- .env                         # Local API key and model configuration
|-- pyproject.toml               # Project metadata and Python dependencies
|-- uv.lock                      # Reproducible dependency lock file
|-- output/
|   -- constraints.json     # Structured applicant requirements
|   -- contract.json        # Structured facts extracted from the contract
|   -- cv.json              # Structured CV profile
|   -- constraints.md       # Contract-versus-requirements report
|   -- matchmaker.md        # Contract-versus-CV report
|-- src/
|   -- loader.ipynb             # Complete extraction and analysis workflow
|-- constraints.txt          # Applicant's contract requirements
|-- contract.pdf             # Employment contract used as input
|-- cv.pdf                   # CV used as input
`-- .venv/                       # Local Python virtual environment
```

### Main files

| File | Purpose |
| --- | --- |
| `src/loader.ipynb` | Defines the Pydantic data models, file tools, agents, prompts, and report-generation steps. |
| `constraints.txt` | Human-readable requirements such as salary, contract duration, working hours, travel, vacation, and non-compete conditions. |
| `contract.pdf` | Source contract read by the contract extraction agent. |
| `cv.pdf` | Source CV read by the CV extraction agent. |
| `/output/constraints.json` | Requirements normalized into nested `Constraint_Element` records. |
| `/output/contract.json` | Contract sections, parties, summaries, facts, conditions, and supporting quotations. |
| `/output/cv.json` | CV summary, experience, education, projects, competencies, languages, and evidence. |
| `/output/matchmaker.md` | Applicant-oriented assessment of how well the contract matches the CV. |
| `/output/constraints.md` | Applicant-oriented assessment of how well the contract satisfies the stated requirements. |

## How the workflow works

The notebook runs four logical stages:

1. **Requirements extraction**  
   A lightweight model reads `constraints.txt`, maps its contents to
   `Constraint_List`, and saves `output/constraints.json`.

2. **PDF extraction**  
   `pypdf` extracts text from `contract.pdf` and `cv.pdf`. Agents convert that
   text into the typed `Contract` and `CVProfile` Pydantic models, then save
   `output/contract.json` and `output/cv.json`.

3. **Contract-to-CV comparison**  
   A more capable model receives both JSON documents and generates
   `output/matchmaker.md`, including aligned and misaligned aspects, expected
   contract terms, a prioritized applicant risk analysis, and interview
   questions.

4. **Contract-to-requirements comparison**  
   The same analysis model compares the structured contract with the structured
   requirements and writes `output/constraints.md`.

The OpenAI Agents SDK tracing context is enabled for each agent run.

## Requirements

- Python 3.13 or newer
- [`uv`](https://docs.astral.sh/uv/) for the locked setup shown below
- An OpenAI API key
- Jupyter support in an editor or browser

The declared dependencies are:

- `openai-agents` for agents, tool calls, structured output, and tracing
- `pypdf` for PDF text extraction
- `pydantic` through the Agents SDK for typed output models
- `python-dotenv` for local environment configuration
- `ipykernel` for notebook execution
- `pywin32` for the current Windows-oriented environment

## Setup

From the project root:

```powershell
uv sync
```

Create or update `.env` without committing credentials:

```dotenv
OPENAI_API_KEY=your-api-key
NANO=your-lightweight-model-id
MINI=your-extraction-model-id
GPT=your-analysis-model-id
```

The model IDs are intentionally configurable:

- `NANO` handles requirements extraction.
- `MINI` handles contract and CV extraction.
- `GPT` handles the two comparison reports.

## Running the analysis

Open `src/loader.ipynb`, select the environment created by `uv`, and run the
cells in order.

The notebook uses paths relative to `src`, so its working directory must be:

```powershell
Set-Location .\src
```

For example, start Jupyter from that directory:

```powershell
uv run jupyter notebook loader.ipynb
```

Replace `constraints.txt`, `contract.pdf`, and `cv.pdf` with the inputs to
analyze, keeping those filenames unless the notebook prompts are updated.
Running the notebook overwrites the corresponding files under `src/output`.

## Data models

The notebook uses typed Pydantic models to constrain agent output:

- `Constraint_List` and `Constraint_Element` represent nested mandatory or
  optional applicant requirements.
- `CVProfile` contains work experience, education, projects, competencies,
  programming and spoken languages, certifications, and source evidence.
- `Contract` contains parties and numbered sections, with normalized facts and
  source quotations.

These schemas make later comparisons more predictable than comparing the raw
PDF text directly.

## Important notes

- The input PDFs and generated JSON files can contain personal, contractual,
  and otherwise sensitive information. Do not publish them or commit real API
  credentials.
- PDF extraction works best with text-based PDFs. Scanned documents require OCR,
  which this project does not currently provide.
- Agent output is probabilistic. Verify extracted facts and quotations against
  the original documents before relying on a report.
- The generated reports are decision-support material, not legal advice.
- The `fulfilled` values in extracted requirements are initialized during
  parsing; the final Markdown comparison is the current source of the actual
  alignment assessment.
