# Custom Mapper and Inverter Guide

## Purpose

This guide explains how to adapt DB-converter to two new tabular cohorts. It is written for an agent working in this repository. The goal is to create a coherent mapper and inverter pair, prepare matching data splits, and update the YAML configuration without breaking the existing code path.

Start this workflow when the user types `!build_custom_NN` in the agent chat.

If preprocessed data already exists, inspect it and infer the input and output dimensions. If preprocessed data are missing, ask the user where the data are saved and whether they are already selected, numeric, normalized, and split. If the data are saved outside the default folder, update `config/data_path.yaml` after confirming the intended location.

## What the Paper and Code Do

DB-converter trains two neural networks:

- `mapper`, also called the forward DB-converter or `m`, maps cohort A into cohort B's schema.
- `inverter`, also called the backward DB-converter or `i`, maps cohort B into cohort A's schema.

The practical assumption is that both cohorts measure comparable underlying constructs. They do not need paired participants, but they should describe related realities. In the paper, this is framed as finding functors between two database categories while preserving local order or neighborhood structure.

In the bi-directional version used by this repo, training uses round trips:

- A sample from cohort A goes through `mapper(A)` and then `inverter(mapper(A))`; the round-trip output should resemble the original A sample.
- A sample from cohort B goes through `inverter(B)` and then `mapper(inverter(B))`; the round-trip output should resemble the original B sample.

The current loss combines reconstruction losses and optional distribution or topology terms:

- L1 loss
- L2 or MSE loss
- Havrda-Charvat entropy term
- pairwise cosine similarity term
- k-neighborhood topology or twin Jaccard term

The important architectural rule is simple: the first layer of each network must match that network's input feature count, and the final layer must match that network's output feature count.

## Current Repository Example

The current repo is configured for DEAS to SHARE:

- Cohort A or DB-A has 56 variables.
- Cohort B or DB-B has 31 variables.
- `mapper`: `56 -> 128 -> 256 -> 512 -> 256 -> 128 -> 31`
- `inverter`: `31 -> 128 -> 256 -> 512 -> 256 -> 128 -> 56`

Files involved:

- `src/model.py` defines `Mapper` and `Inverter`.
- `src/db_converter.py` instantiates both networks and defines the training losses.
- `src/dataloder.py` loads A and B train, validation, and test arrays.
- `test/generate_ouput.py` loads a checkpoint and writes converted and reconverted outputs.
- `config/architecture.yaml` declares mapper and inverter dimensions.
- `config/data_path.yaml` points to preprocessed train, validation, and test files.
- `config/config.yaml` contains run names, output paths, checkpoint loading names, and common loss or training settings.
- `config/advanced_config.yaml` contains optimizer settings and k-neighborhood settings.

Important repo detail: `config/architecture.yaml` declares the dimensions, but `src/model.py` currently hard-codes the layer dimensions inside `nn.Linear(...)` and `nn.BatchNorm1d(...)`. For a new cohort pair, updating YAML alone is not enough. An agent must also update `src/model.py`, or replace the hard-coded model with a dimension-driven implementation.

## Step 1 Find the User Data Dimensions

First check whether preprocessed data already exists:

- Read `config/data_path.yaml`.
- Inspect the folder named by `data_path`.
- Load these six files if present: `x_train`, `x_val`, `x_test`, `y_train`, `y_val`, `y_test`.
- Record row counts and feature counts.

Expected shapes:

- `x_train`, `x_val`, and `x_test` must all have the same number of columns. This is `Mapper_input_size` and `Inverter_output_size`.
- `y_train`, `y_val`, and `y_test` must all have the same number of columns. This is `Mapper_output_size` and `Inverter_input_size`.
- `x_train` and `y_train` must have the same number of rows because the loader returns a `TensorDataset(x_train, y_train)`.
- `x_val` and `y_val` must have the same number of rows.
- `x_test` and `y_test` must have the same number of rows.

If data does not exist yet, ask the user for:

- the cohort A data file
- the cohort B data file
- the exact columns to use in each cohort
- the missing-data rule
- whether the rows are paired, unpaired but comparable, or intentionally subsampled
- where to write the preprocessed arrays

## Step 2 Prepare the Data

The converter expects numeric arrays or CSV files that can be loaded as numeric arrays. The safest standard for new work is `.npy`.

Recommended preprocessing:

1. Select only reviewed variables for cohort A and cohort B.
2. Convert booleans and ordered categories to numeric values.
3. Drop or impute missing values using a user-approved rule.
4. Ensure both cohorts have equal row counts per split. If one cohort is larger, subsample it to the smaller count after documenting the choice.
5. Min-max normalize every selected feature into a comparable numeric range, usually `0` to `1`.
6. Split into train, validation, and test arrays. Use a fresh random seed and record it.
7. Save the files under a project-specific subfolder such as `data/preprocessed/<project_name>/`.

For rigorous validation, fit each scaler on the training partition and apply it to validation and test. If the user wants to reproduce the older notebook examples exactly, note that those examples often normalize before splitting.

## Step 3 Update data_path.yaml

Point `config/data_path.yaml` to the chosen preprocessed folder and filenames. Example:

```yaml
data_path: "data/preprocessed/my_project"

x_train: "cohort_a_train.npy"
x_val: "cohort_a_val.npy"
x_test: "cohort_a_test.npy"

y_train: "cohort_b_train.npy"
y_val: "cohort_b_val.npy"
y_test: "cohort_b_test.npy"
```

After editing this file, run shape checks in a fresh Python process. `src/dataloder.py` loads files at import time, so a long-running Python session may keep old values.

## Step 4 Update architecture.yaml

Set dimensions from the preprocessed arrays:

```yaml
Mapper_input_size: <number of cohort A variables>
Mapper_output_size: <number of cohort B variables>
Mapper_learning_rate: null
Inverter_input_size: <number of cohort B variables>
Inverter_output_size: <number of cohort A variables>
Inverter_learning_rate: null
```

YAML `null` is safer than the literal string `None`.

The two consistency equations are:

- `Mapper_input_size == Inverter_output_size == cohort A feature count`
- `Mapper_output_size == Inverter_input_size == cohort B feature count`

## Step 5 Create the Custom Mapper and Inverter

Use fully connected layers unless the user's data has a strong reason for another architecture. The paper states that the architecture can be changed as long as input and output dimensions match the schemas.

For ordinary tabular cohorts, use a mirrored hidden stack:

- Let `a_dim` be the number of cohort A variables.
- Let `b_dim` be the number of cohort B variables.
- Let `d = max(a_dim, b_dim)`.
- For small data, use `[max(32, 4*d), max(64, 8*d), max(32, 4*d)]`.
- For medium tabular data, use `[round_to_16(2*d), round_to_16(4*d), round_to_16(8*d), round_to_16(4*d), round_to_16(2*d)]`, capped around 512 or 1024 if sample size is limited.
- For very small sample sizes, reduce width or increase regularization before adding depth.

The mapper should be:

```text
a_dim -> hidden widths -> b_dim
```

The inverter should be:

```text
b_dim -> same hidden widths -> a_dim
```

Each hidden layer in the current code uses:

- `nn.Linear`
- `nn.PReLU`
- optional `nn.Dropout`
- `nn.BatchNorm1d`

The current code also applies `PReLU` on the final layer. Keep this for compatibility unless there is a task-specific reason to change it. Do not add `sigmoid` only because the data are min-max normalized; it may make optimization harder. Clip or inverse-transform outputs after inference if the downstream task requires strict bounds.

## Dimension Driven Model Template

An agent can either edit the hard-coded widths in `src/model.py` or replace the two classes with a reusable builder. If replacing, preserve the public class names `Mapper` and `Inverter` because other files import them.

```python
import torch
import torch.nn as nn
import pytorch_lightning as pl
from src.read_config import read_config

config = read_config("config/config.yaml")
dropout_rate = config.get("dropout_rate", 0.3)

def make_mlp(input_size, output_size, hidden_sizes, dropout_rate):
    layers = []
    prev = input_size
    for width in hidden_sizes:
        layers.append(nn.Linear(prev, width))
        layers.append(nn.PReLU(num_parameters=1, init=0.25))
        if dropout_rate > 0:
            layers.append(nn.Dropout(p=dropout_rate))
        layers.append(nn.BatchNorm1d(num_features=width, affine=False))
        prev = width
    layers.append(nn.Linear(prev, output_size))
    layers.append(nn.PReLU(num_parameters=1, init=0.25))
    return nn.Sequential(*layers)

class Mapper(pl.LightningModule):
    def __init__(self, Mapper_input_size, Mapper_output_size, Mapper_learning_rate):
        super().__init__()
        self.Mapper_input_size = Mapper_input_size
        self.Mapper_output_size = Mapper_output_size
        self.Mapper_learning_rate = Mapper_learning_rate
        self.net = make_mlp(Mapper_input_size, Mapper_output_size, [128, 256, 512, 256, 128], dropout_rate)

    def forward(self, x):
        x = torch.flatten(x, start_dim=1)
        return self.net(x)

class Inverter(pl.LightningModule):
    def __init__(self, Inverter_input_size, Inverter_output_size, Inverter_learning_rate):
        super().__init__()
        self.Inverter_input_size = Inverter_input_size
        self.Inverter_output_size = Inverter_output_size
        self.Inverter_learning_rate = Inverter_learning_rate
        self.net = make_mlp(Inverter_input_size, Inverter_output_size, [128, 256, 512, 256, 128], dropout_rate)

    def forward(self, x):
        x = torch.flatten(x, start_dim=1)
        return self.net(x)
```

If using this template with checkpoint loading code that expects layer names such as `fc1`, either retrain from scratch or also update the checkpoint loading logic. For fresh projects, retraining is normal.

## Step 6 Check YAML and Model Coherence

Before training, verify:

- `config/data_path.yaml` points to existing files.
- all arrays are two-dimensional numeric matrices.
- A splits share one column count.
- B splits share one column count.
- row counts match within each split.
- `architecture.yaml` matches the array column counts.
- `src/model.py` first and last layer sizes match `architecture.yaml`.
- batch size is greater than `1` and not larger than the smallest split if batch normalization is active.
- `project_name` and `model_name` in `config/config.yaml` are unique for this run.
- `load_project_name`, `load_model_name`, and `load_epoch_name` point to the checkpoint intended for inference.

## Step 7 Train and Inspect

Run training only after preprocessing and dimension checks pass. During training, compare:

- `train/mapping_loss`
- `train/inverting_loss`
- `val/mapping_l1_loss`
- `val/inverting_l1_loss`
- `val/mapping_l2_loss`
- `val/inverting_l2_loss`

If one direction learns much faster than the other:

- adjust `mapper_repeat` or `inverter_repeat` in `config/config.yaml`
- reduce the stronger model's width
- increase dropout or regularization
- increase the weaker model's repeat count
- revisit feature selection for constructs that are not comparable

## Step 8 Run Inference

Inference writes four output families:

- `DB-A_converted`: `mapper(A)`, so A expressed in B's schema.
- `DB-A_reconverted`: `inverter(mapper(A))`, so A converted to B and then back to A.
- `DB-B_converted`: `inverter(B)`, so B expressed in A's schema.
- `DB-B_reconverted`: `mapper(inverter(B))`, so B converted to A and then back to B.

Keep a variable manifest next to each output so users know which column each output value represents. The repo currently saves numeric arrays without column names.

## Do Not Break Existing Work

For a new user task:

- Prefer adding a project-specific data subfolder instead of overwriting `data/preprocessed`.
- Preserve old checkpoints unless the user asks to delete them.
- Use new `project_name` and `model_name` values for new experiments.
- Commit or copy the original `src/model.py` before major architecture changes if this becomes a standalone repo later.
- Keep feature selection optional. The converter can still run if the user directly supplies already selected, normalized, split data.
