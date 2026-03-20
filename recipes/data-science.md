# Claude Code Recipe: Data Science / ML

> Turn Claude Code into your pair-programming data scientist — from messy CSV to production pipeline.

## Quick Start

### CLAUDE.md

Drop this into your project root as `CLAUDE.md`:

```markdown
# Project: [Your Project Name]

## Overview
[One sentence: what data, what problem, what model]

## Project Structure
```
├── notebooks/          # Exploration only — not the source of truth
├── src/                # Production-ready Python modules
│   ├── data/           # Loading, cleaning, feature engineering
│   ├── models/         # Model definitions and training logic
│   ├── evaluation/     # Metrics, visualization, reporting
│   └── utils/          # Shared helpers
├── tests/              # pytest tests for src/
├── data/               # LOCAL ONLY — never committed
│   ├── raw/            # Immutable original data
│   ├── processed/      # Cleaned/transformed data
│   └── external/       # Third-party reference data
├── models/             # Serialized model artifacts (git-ignored)
├── experiments/        # Experiment configs and results (YAML/JSON)
├── outputs/            # Figures, reports, exported tables
├── requirements.txt    # or pyproject.toml
└── CLAUDE.md
```

## Environment
- Python 3.11+
- Package manager: pip with venv (or conda)
- Key libraries: pandas, numpy, scikit-learn, matplotlib, seaborn
- Optional: xgboost, lightgbm, pytorch, tensorflow
- Experiment tracking: [MLflow / Weights & Biases / None]

## Data Rules
- NEVER commit files in data/ or models/ — they are git-ignored
- Reference datasets by path, never embed data in code
- All data loading goes through src/data/loader.py
- Document data sources and schemas in data/README.md

## Code Conventions
- Prefer .py scripts over notebooks for all logic
- Notebooks are for visualization and storytelling only
- Type hints on all function signatures
- Docstrings on all public functions (NumPy style)
- Random seed: 42 (set via src/utils/reproducibility.py)
- Max line length: 88 (Black formatter)

## Notebook Rules
- Clear all outputs before committing (use pre-commit hook)
- Every notebook starts with a markdown cell explaining its purpose
- Notebooks import from src/ — never duplicate logic

## Reproducibility
- Pin all dependency versions in requirements.txt
- Set random seeds for numpy, random, and framework-specific RNGs
- Log experiment parameters and metrics to experiments/ directory
- Every model training run produces a config YAML and a metrics JSON

## Testing
- Test data pipelines with small fixture CSVs in tests/fixtures/
- Test feature engineering functions with known input/output pairs
- Test model training completes without error on tiny datasets
- Coverage target: 80% on src/
```

### settings.json

Save as `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(python *)",
      "Bash(python -m pytest*)",
      "Bash(python -m pip install*)",
      "Bash(pip install*)",
      "Bash(pip freeze*)",
      "Bash(conda install*)",
      "Bash(conda list*)",
      "Bash(jupyter nbconvert*)",
      "Bash(black *)",
      "Bash(ruff *)",
      "Bash(mypy *)",
      "Bash(dvc *)",
      "Bash(mlflow *)"
    ],
    "deny": [
      "Bash(rm -rf data/*)",
      "Bash(rm -rf models/*)"
    ]
  }
}
```

### Recommended MCP Servers

- [modelcontextprotocol/server-sqlite](https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite) - Query and analyze local SQLite databases. **When to use:** You have structured data in SQLite and want Claude to explore schemas, run analytical queries, or prototype SQL transformations before writing Python code.

- [modelcontextprotocol/server-filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) - Secure file operations with access controls. **When to use:** Claude needs to read CSV headers, inspect data samples, or browse data directories outside the project root to understand what you're working with.

- [modelcontextprotocol/server-fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) - Fetch web content with robots.txt compliance. **When to use:** Downloading public datasets, reading API documentation for data sources, or pulling reference material for a specific ML technique.

- [modelcontextprotocol/server-memory](https://github.com/modelcontextprotocol/servers/tree/main/src/memory) - Persistent knowledge graph across sessions. **When to use:** Multi-session ML projects where you need Claude to remember which features worked, which models were tried, hyperparameter decisions, and dead ends to avoid.

## Recommended Workflow

### Step 1: Data Exploration and Profiling

Start by asking Claude to generate a profiling script — not to analyze data directly (Claude cannot see your data files).

```
Write a data profiling script that loads data/raw/transactions.csv and prints:
- Shape, dtypes, memory usage
- Missing value counts per column
- Descriptive statistics for numeric columns
- Value counts for categorical columns (top 10)
- Correlation matrix for numeric features
Save figures to outputs/profiling/
```

Run the script yourself and share key output with Claude. This grounds the conversation in real data characteristics.

### Step 2: Data Cleaning and Feature Engineering

Work iteratively: describe the issues you found in Step 1 and ask Claude to write cleaning functions.

```
Based on the profiling results:
- Column 'amount' has 3% nulls — impute with median
- Column 'category' has 47 unique values — group rare ones (<1%) into 'Other'
- Create time-based features from 'timestamp': hour, day_of_week, is_weekend

Write this as src/data/features.py with a build_features(df) -> df function.
Include tests in tests/test_features.py.
```

Keep each transformation as a pure function: input DataFrame in, new DataFrame out. This makes testing straightforward and Claude can reason about each step in isolation.

### Step 3: Model Development and Experimentation

Ask Claude to scaffold model training with proper cross-validation and experiment logging.

```
Create src/models/train.py that:
- Loads processed data from data/processed/features.csv
- Splits into train/test (80/20, stratified, seed=42)
- Trains a RandomForestClassifier with 5-fold CV on the train set
- Logs parameters and CV metrics to experiments/rf_baseline.yaml
- Saves the model to models/rf_baseline.pkl
```

Run experiments yourself and feed results back to Claude for iteration. Be specific about what metrics matter ("optimize for recall at 90% precision").

### Step 4: Evaluation and Visualization

Use Claude to generate evaluation scripts and visualizations. Be explicit about what plots you want.

```
Create src/evaluation/report.py that loads the trained model and test set, then generates:
- Classification report (precision, recall, F1 per class)
- Confusion matrix heatmap (seaborn, save as PNG)
- ROC curve with AUC score
- Feature importance bar chart (top 20)
Save all figures to outputs/evaluation/ and metrics to experiments/rf_baseline_metrics.json
```

### Step 5: Productionization

Refactor validated notebook logic into clean, tested modules.

```
Refactor notebooks/03_model_experiments.ipynb into proper modules:
- Extract data loading into src/data/loader.py
- Extract feature pipeline into src/data/features.py (already exists — merge)
- Extract model training into src/models/train.py (already exists — update)
- Add a CLI entry point: python -m src.models.train --config experiments/config.yaml
- Make sure all existing tests still pass
```

This is where Claude Code shines — it can read the notebook, understand the intent, and produce clean modules with proper error handling and type hints.

## Tips and Pitfalls

1. **Use .py scripts, not notebooks, as the primary interface.** Claude Code reads and edits `.py` files far more reliably than `.ipynb` files. Use notebooks only for final visualization and storytelling. Write all logic in `src/` and import it into notebooks.

2. **Claude cannot see your data — always describe it.** Instead of saying "analyze this CSV," run a profiling script first and paste the output summary into the conversation. Include dtypes, shape, sample rows, and distribution stats.

3. **Pin your environment versions in CLAUDE.md.** A simple "Python 3.11, pandas 2.1, scikit-learn 1.3" saves you from Claude generating code with incompatible APIs. This is especially important for fast-moving libraries like PyTorch or TensorFlow.

4. **Enforce .gitignore from day one.** Add `data/`, `models/`, `*.pkl`, `*.h5`, `*.parquet`, and `*.csv` to `.gitignore` before your first commit. Claude may try to `git add` data files if they exist — a strict deny rule in `settings.json` adds a safety net.

5. **Break the "one big notebook" habit.** Instead of asking Claude to modify a 500-cell notebook, ask it to extract logic into focused scripts: one for feature engineering, one for training, one for evaluation. Each script is easier to test, review, and version.

6. **Be explicit about visualization preferences.** Tell Claude "use seaborn with the whitegrid style, figsize=(10,6), save as PNG at 150 DPI" rather than letting it guess. Add these defaults to your CLAUDE.md.

7. **Use Claude for data pipeline tests.** Data scientists often skip testing. Ask Claude to write tests that validate: input schema (column names and types), output shape after transformations, edge cases (empty DataFrames, all-null columns), and determinism (same input produces same output).

8. **Track experiments outside of git.** Use a simple YAML/JSON convention in `experiments/` or a tool like MLflow. Tell Claude which tracking system you use so it generates the right logging code.

## Starter Prompt Templates

### Exploratory Data Analysis Script

```
Write a Python script src/data/profile.py that:
1. Loads [DATASET_PATH] using pandas
2. Prints shape, dtypes, and memory usage
3. Shows missing value percentages per column (sorted descending)
4. Generates descriptive statistics for all numeric columns
5. Plots histograms for numeric columns (4 per row) and saves to outputs/profiling/distributions.png
6. Plots a correlation heatmap and saves to outputs/profiling/correlations.png
7. Prints value counts (top 10) for all object/category columns

Use matplotlib and seaborn. Set style to 'whitegrid'. Create output directories if they don't exist.
```

### Feature Engineering Pipeline

```
Create src/data/features.py with a function build_features(raw_df: pd.DataFrame) -> pd.DataFrame that:
1. Drops columns: [LIST_COLUMNS_TO_DROP]
2. Fills missing values: [COLUMN] with [STRATEGY: median/mode/constant]
3. Encodes categoricals: [COLUMN] using [one-hot / label / target encoding]
4. Creates derived features: [DESCRIBE NEW FEATURES]
5. Returns a new DataFrame (do not mutate the input)

Also create tests/test_features.py with:
- A pytest fixture loading tests/fixtures/sample_raw.csv (5-10 rows covering edge cases)
- Test that output has expected columns
- Test that no nulls remain in output
- Test that function is pure (input DataFrame unchanged)
```

### Model Training with Cross-Validation

```
Create src/models/train.py that:
1. Loads features from [FEATURES_PATH]
2. Splits target [TARGET_COLUMN] from features
3. Performs train/test split (test_size=0.2, stratify=target, random_state=42)
4. Defines a pipeline: StandardScaler -> [MODEL: RandomForestClassifier / XGBClassifier / etc.]
5. Runs 5-fold stratified CV on training set, reporting [METRICS: accuracy, f1, roc_auc]
6. Fits final model on full training set
7. Evaluates on test set and prints classification report
8. Saves model to models/[MODEL_NAME].pkl using joblib
9. Saves experiment config and metrics to experiments/[EXPERIMENT_NAME].yaml

Use scikit-learn pipelines. Set all random seeds to 42.
```

### Results Visualization Dashboard

```
Create src/evaluation/visualize.py that loads model from [MODEL_PATH] and test data from [TEST_DATA_PATH], then generates:

1. Confusion matrix heatmap (annotated with counts and percentages)
2. ROC curve with AUC score (if binary) or per-class ROC (if multiclass)
3. Precision-Recall curve
4. Feature importance plot (top 20, horizontal bar chart)
5. Prediction error distribution (histogram of residuals, if regression)

Save all plots to outputs/evaluation/ as PNGs at 150 DPI.
Use seaborn whitegrid style, figsize=(10, 6) for all plots.
Add titles and axis labels to every plot.
Print a summary of key metrics to stdout.
```

### Notebook-to-Module Refactoring

```
Read notebooks/[NOTEBOOK_NAME].ipynb and refactor it into production modules:

1. Identify all data loading logic -> move to src/data/loader.py
2. Identify all transformations -> move to src/data/features.py
3. Identify all model code -> move to src/models/train.py
4. Identify all evaluation/plotting -> move to src/evaluation/visualize.py
5. Create a thin notebook that imports from src/ and only contains:
   - Markdown explanations
   - Function calls
   - Visualization outputs

Preserve all existing tests. Add new tests for any extracted functions that lack coverage.
Do not change any logic — this is a pure refactor.
```
