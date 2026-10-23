# Local LLM Benchmark

Experiments with running and evaluating Llama 3.2 3B locally using Ollama, Python, and Jupyter notebooks.

## Project objective

Explore how a local language model performs in terms of generation speed and answer accuracy, and investigate how quantization affects its results.

Quantization reduces the precision used to store model weights, making models smaller. This project compares two versions of the same model: **Q4_K_M** and **Q8_0**.

## Project progress

### 1. First interaction with a local LLM

[Notebook 01 — First LLM](notebooks/01_first_llm.ipynb)

- Send a prompt to Llama 3.2 3B through Ollama's local HTTP API.
- Read the generated response and token counts.
- Calculate generation speed in tokens per second.
- Create a reusable generation function.

### 2. Generation speed benchmark

[Notebook 02 — Benchmark](notebooks/02_benchmark.ipynb)

- Calculate generation speed using the number of generated tokens and generation duration.
- Repeat each prompt five times and calculate the average speed.
- Test four prompts covering explanation, mathematical reasoning, text generation, and code generation.

The saved final results range from **32.93 to 35.06 tokens per second** across these four prompts.

These measurements describe token generation speed. They do not include the full request duration or assess whether the generated answers are correct. They are preliminary observations from a small set of prompts.

### 3. Quantization comparison on GSM8K

[Notebook 03 — Quantization comparison](notebooks/03_quantization_comparison.ipynb)

Evaluate **Q4_K_M** and **Q8_0** on the same first 50 questions from the GSM8K test split.

Evaluation settings:

- Model: Llama 3.2 3B
- Temperature: 0
- Maximum generated tokens: 512
- Prompt instruction: provide reasoning and finish with `FINAL_ANSWER: <number>`
- Scoring: extract the final answer, remove dollar signs and commas, and compare its numeric value with the reference answer

Results:

| Quantization | Correct answers | Accuracy |
|---|---:|---:|
| Q4_K_M | 38 / 50 | 76% |
| Q8_0 | 38 / 50 | 76% |

The overall scores are identical, but the successful answers differ:

| Outcome | Questions |
|---|---:|
| Both versions correct | 35 |
| Only Q4_K_M correct | 3 |
| Only Q8_0 correct | 3 |
| Both versions incorrect | 9 |

On this sample, Q8_0 does not improve the overall score. However, identical scores do not mean identical model behavior.

## Error analysis and limitations

Reviewing the full model responses revealed two types of failure:

- **Reasoning errors:** incorrect interpretation of percentages or missing costs in a profit calculation.
- **Output format failures:** a correct answer may be counted as incorrect if the model omits the required `FINAL_ANSWER:` marker.

For example, Q4_K_M correctly calculated **60%** for a dance-class problem but omitted the marker. The extraction procedure therefore recorded no final answer.

The reported accuracy measures both answer correctness and compliance with the requested format.

Additional limitations:

- Only 50 of the 1,319 GSM8K test questions were evaluated.
- The sample uses the first 50 questions rather than a random selection.
- A single evaluation run per quantization does not establish consistency across runs.
- Exact dependency versions and model identifiers need to be recorded more systematically.
- Generation speed has not yet been compared between Q4_K_M and Q8_0.

These results are exploratory and do not establish that the two quantizations are equivalent on the full dataset.

## Saved results

- [Q4_K_M results](results/gsm8k_q4.csv)
- [Q8_0 results](results/gsm8k_q8.csv)

The CSV files contain the questions, reference answers, extracted predictions, correctness labels, and raw model responses.

## Tools used

- Python
- Jupyter Notebook
- Ollama
- Requests
- pandas
- NumPy
- Hugging Face Datasets

## Running the notebooks

The notebooks require an active local Ollama server at `http://localhost:11434` and the model used by each notebook.

Models used:

- `llama3.2:3b` — verified as Q4_K_M during this experiment
- `llama3.2:3b-instruct-q8_0` — Q8_0

Run the notebooks from the `notebooks` directory so that relative export paths resolve correctly. Check the model name before running each evaluation and use separate output files for Q4 and Q8.

Model files are stored locally and are not included in this repository.

## Next steps

- Distinguish output format failures from reasoning errors in the evaluation report.
- Record model identifiers, dependency versions, and evaluation settings.
- Extend evaluation to a larger GSM8K sample.
- Compare generation speed and memory usage across quantization levels.
