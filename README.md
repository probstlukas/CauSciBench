<h1 align="center">
<br>
CauSciBench: A Comprehensive Benchmark for End-to-End Causal Inference in Scientific Research
</h1>

**Note**: This is a work in progress. We will update the repository frequently in the subsequent days.

## Overview

**CauSciBench** is the first comprehensive benchmark designed to evaluate end-to-end causal inference capabilities (from natural language questions to effect estimates) for scientific research. Closely following the scientific causal analysis workflow, our benchmark assesses the ability of AI models to:

- Parse and understand dataset descriptions and queries
- Identify treatment and outcome variables
- Choose appropriate inference models and method-specific variables (e.g., instruments, running variables)
- Implement the selected methods
- Provide statistical interpretations of results in the context of the original query

The benchmark spans both the potential-outcomes and structural causal model (SCM) frameworks.

## Benchmark Data

### Data Sources

We provide a thoroughly filtered dataset with corresponding natural language queries from three distinct sources:

1. **Real-world Studies** 
   - Published papers on empirical causal inference from diverse disciplines including economics, political science, healthcare, and criminology
   - Information on selected studies can be found in `data/source_info.pdf`

2. **Synthetic Scenarios** 
   - Synthetically generated data with known causal effects
   - Hypothetical contexts and variables generated to resemble real-world causal analysis

3. **Textbook Examples** 
   - Examples focused on causal inference from [QRData](https://github.com/xxxiaol/QRData) (Liu et al., 2024)

### Annotation Details

Our expert-curated annotations consist of:

1. Description of the dataset
2. Causal query (in plain language) that does not state what method or variables to pick
3. Reference causal method
4. Causal effect estimate
5. Standard error
6. Statistical significance
7. Treatment variable
8. Outcome variable
9. Control variables / observed confounders
10. Model-specific variables including instrument (IV), running variable (RDD), time variable (DiD), state variable (DiD)

### Data Files

**Metadata files** containing query information:
- `data/real_info.csv` - Real-world studies metadata
- `data/synthetic_info.csv` - Synthetic scenarios metadata
- `data/qr_info.csv` - Textbook examples metadata

Each entry contains the following core information:
- `paper_name`
- `data_description`
- `natural_language_query`
- `answer`
- `std_error`
- `is_significant`
- `method`
- `treatment`
- `outcome`
- `covariates`
- `running_var`
- `temporal_var`
- `instrument_var`
- `state_var`
- `interacting_variable`
- `multirct_treatment`
- `data_files`
- `mediator` (Synthetic Data exclusive)
- `domain` (Real-world Studies exclusive)

**Dataset files** are organized in the following structure:

```
data/
├── synthetic_data/     # Synthetic datasets
├── qrdata/            # Textbook examples data
├── real_data/         # Real-world study datasets
└── json/              # Query files
    ├── qrdata.json
    ├── real_data.json
    └── synthetic_data.json
```

The annotated data is also provided in JSON format in `data/json/`, with corresponding CSV files in `data/real_data`, `data/synthetic_data`, and `data/qrdata`.

## Replication

### Prerequisites
- Python 3.10+ (local host)
- Docker (required for safe code execution)
- Provider credentials (optional, depending on `--api`)
- Required libraries can be found in requirements.txt

### 1. Build Docker Image

```bash
docker build -t python-baseline-http -f baselines/Dockerfile.http baselines
```

### 2. Dataset Organization

If you need to create your own JSON query files from CSV metadata, use:

```bash
bash scripts/create_json.sh
```

### 3. Run Baseline Experiments

You can run experiments using either the provided script or directly with Python:

**Using the script:**
```bash
bash scripts/run_baseline.sh
```

**Using Python directly:**
```bash
python baselines/run_baselines.py \
  --queries data/json/qrdata.json \
  --output output/qrdata/qrdata_react_gpt-4o.json \
  --api openai \
  --model gpt-4o \
  --persistent \
  --react \
  --data-type qrdata
```

#### How baselines/run_baselines.py Works

The `run_baselines.py` script orchestrates the entire causal analysis workflow:

1. **Load Queries**: Reads JSON files containing causal questions and meta information
2. **Initialize LLM**: Connects to your chosen API provider (OpenAI, Azure, Vertex, Together)
3. **Docker Setup**: Starts Python containers for safe code execution
4. **Analysis Loop**: For each query:
   - Sends the causal question to the LLM with dataset context
   - LLM generates Python code for causal estimation
   - Executes code in Docker container with libraries like DoWhy, EconML
   - Iterates based on results and errors
   - Extracts structured final results
5. **Save Results**: Outputs comprehensive JSON with chat history, code, and analysis

**Key Parameters:**
- `--queries`: Path to JSON file with causal questions
- `--output`: File path where results are saved
- `--api`: LLM provider (e.g., openai, azure, vertex, together)
- `--model`: LLM model (e.g., gpt-4o, claude-3-sonnet)
- `--persistent`: Use stateful Python environment
- `--potm/--react/--chain`: Different prompting strategies; default is direct prompting
- `--data-type`: Dataset category (real, synthetic, qrdata)

#### Output Structure

Each experiment produces a JSON file:
```
output/{data_source}/{data_source}_{prompt}_{model}.json
```

Example: `output/qrdata/qrdata_react_gpt-4o.json`

### 4. Compile Results

After running experiments, compile the JSON outputs into CSV format:

```bash
python scripts/compile_results.py -if output/{data_source} -of errors/{data_source} -sd {data_source}
```

For example, for qrdata:
```bash
python scripts/compile_results.py -if output/qrdata -of errors/qrdata -sd qrdata
```

The `compile_results.py` script processes all JSON files in the input folder and creates CSV files for each prompting strategy (basic, cot, pot, react). It standardizes method names, extracts causal effects, and combines results across different models into a structured format for analysis.

#### CSV Results
Compiled results are saved as:
```
errors/{data_source}/{data_source}_{prompt}.csv
```

Each CSV contains the original queries, ground truth methods/effects, and predictions from each model tested.

## Script Configuration

### scripts/run_baseline.sh
Modify these variables to change experimental settings:
- `QUERIES_JSON_PATH`: Which dataset to analyze
- `API`: LLM provider
- `MODEL`: Specific model to use
- `OUTPUT_FOLDER`: Folder where results are saved 

### scripts/create_json.sh
Configure column mappings for CSV-to-JSON conversion:
- `FILENAME_COL`: Dataset filename column
- `DESC_NAME_COL`: Dataset description column
- `QUERY_NAME_COL`: Causal query column
- `PAPER_NAME_COL`: Study identifier column
- `OUTPUT_FOLDER`: Folder where output is saved
- `OUTPUT_NAME`: Name of the output JSON file

## License

We use data from published papers, and the usage terms vary from dataset to dataset. Details about the licenses are provided in the readme.md file in each dataset folder. They can be found in the folders: `data/real_data`, `data/synthetic_data`, and `data/qrdata`. 

**Important**: Users must comply with the license terms of each individual dataset they use. Always review the license terms at the original data sources and ensure compliance.
