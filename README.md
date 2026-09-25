# From Knowledge Access to Source Learning: Developing Source-Specific Competence

Code for the paper *From Knowledge Access to Source Learning: Developing Source-Specific Competence*. **SourceLearn** maintains an explicit, revisable source model M over a persistent
authoritative source D and develops it in three steps, every persistent update being reconstructed from
the source itself:

```
M_0 = Construct(D; Pi_0)          source model construction                  draft/
M_S = SelfLearn(D, M_0)           Self-Directed Source Learning              self_learning/
M_T = TaskLearn(D, M_S; G)        Task-Guided Source Learning                task_learning/
```

## Layout

```
src/sourcelearn/
  pipeline.py        the only entry point: construction -> self-directed -> task-guided learning
  draft/             M_0: reading brief, partition of large sources into regions, entity-centred reading
  self_learning/     M_S: Inspect -> Study (Deepen / Connect) -> Consolidate
  task_learning/     M_T: diagnose E*_t and C_t, failure-guided local refinement, lessons -> Pi_1, recalibration
  reconstruction/    grounded reconstruction shared by every stage (Inspect / Reconstruct / GroundApply); defaults.py
  answering/         the task solver F(q, R(D, q), A_B(M, q)) and the judge
  source_model.py    knowledge units and source-model activation A_B(M, q)
  retrieval.py       hybrid retrieval R(D, q): BM25 + dense with reciprocal-rank fusion
  core/              LLM client (OpenAI-compatible), source elements, tracing
```

## Setup

```bash
pip install -e .            # Python >= 3.11
cp .env.example .env        # OPENAI_API_KEY (and/or OPENROUTER_API_KEY for <vendor>/<model> ids)
```

## Run

```bash
python -m sourcelearn.pipeline --workspace <source> --questions <questions.json> --adapter docs \
    --model <backbone> --judge-model <judge> --out <run> --seeds 0,1,2
```

- `--workspace`: a directory of documents (`.md`, `.rst`, `.txt`; `--adapter docs`) or a Python
  repository (`--adapter code`, optionally `--region <package>`).
- `--questions`: a JSON list of `{"qid", "question", "answer"}`; `"options"` + `"answer": <index>` for
  multiple choice.
- `--seeds`: one task-guided learning run per seed, each on a random 30 % / 70 % guidance/test split
  (`--split-dir <dir>` with `seed<N>.json` files `{"train": [...], "test": [...]}` to use given splits).
- `--model` is the backbone of every stage; `--judge-model` judges answers (default: the backbone).

The pipeline always runs the whole method in order; a stage whose output exists is skipped, so an
interrupted run resumes with the same command. All other quantities are fixed constants of the paper's
protocol (`reconstruction/defaults.py`).

## Output

```
<run>/task_s<seed>/m_task.json      M_T, the learned source model: a JSON list of knowledge units
                                    (`SourceModelUnit` in `source_model.py`: statement, role, scope, conditions, support anchors)
<run>/task_s<seed>/summary.md       accuracy of M_T on the test tasks (answered with hybrid retrieval + the activated model)
```
