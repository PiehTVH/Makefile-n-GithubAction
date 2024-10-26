# Introducing MakeFile

A Makefile is a blueprint for building and managing software project. It is a file containing instructions for automating tasks, streamlining complex build processes, and ensuring consistency.

## Core components

- Target: the desired outcome, like: "build", "test or "clearn". Image them as goals we want to achieve through the Makefile
- Dependencies: the files needed to build the target.
- Actions: The instructions or scripts that tell the system how to build the target.
- Variables: global variables that act like arguments passed to the actions (E.g. $variable_1, $variable_2)

The template looks like:

```bash
Variable = some_value

Target: Dependencies
    Actions: (commands to build the target) $Variable
```

Example:

```bash
# Define variables
variable_1 = 5
variable_2 = 10

# Target to run a test win variables
test: test_data.txt # Dependency on test data file
    python test.py $variable_1 $variable_2
```

# Using MakeFile for a Data Science Project

In this section, we focus on how to use Makefile and Make CLI tools in a data science project.

Dataset: [World Happiness Report 2023](https://www.kaggle.com/datasets/ajaypalsinghlo/world-happiness-report-2023)

We will cover data processing, analysis, and saving data summaries and visualizations.

## Setting up

To start, we create a folder called `Makefile-Action`, and create the `data_processing.py`.

```bash
mkdir Makefile-Action
cd Makefile-Action/
touch(or code) data_processing.py
```

## Data processing

We probably take the following steps (add more if necessary):

1. Load the raw data
2. Check and rename columns for human-readable purpose.
3. Process the missing values.
4. Save the cleaned dataset.

**File**: _data_processing.py_

```bash
import sys
import pandas as pd

# Check if the data location argument is provided
if len(sys.argv) != 2:
    print("Usage: python data_processing.py <data_location>")
    sys.exit(1)

# Load the raw dataset (step 1)
df = pd.read_csv(sys.argv[1])

# Rename columns to more descriptive names (step 2)
df.columns = [
    "Country",
    "Happiness Score",
    "Happiness Score Error",
    "Upper Whisker",
    "Lower Whisker",
    "GDP per Capita",
    "Social Support",
    "Healthy Life Expectancy",
    "Freedom to Make Life Choices",
    "Generosity",
    "Perceptions of Corruption",
    "Dystopia Happiness Score",
    "GDP per Capita",
    "Social Support",
    "Healthy Life Expectancy",
    "Freedom to Make Life Choices",
    "Generosity",
    "Perceptions of Corruption",
    "Dystopia Residual",
]

# Handle missing values by replacing them with the mean (step 3)
df.fillna(df.mean(numeric_only=True), inplace=True)

# Check for missing values after cleaning
print("Missing values after cleaning:")
print(df.isnull().sum())

print(df.head())

# Save the cleaned and normalized dataset to a new CSV file (step 4)
df.to_csv("processed_data\WHR2023_cleaned.csv", index=False)
```

## Data analysis

In `Makefile-action` folder, create _data_analysis.py_

Next, We will follow these stesp below:

1. Load the clean dataset.
2. Generate the data summary, the data information and save it in the file .txt.
3. Plot visualization:
   - The distribution of happiness scores
   - Top 20 countries by happiness score
   - Happiness score versus GDP per capita
   - Happiness Score versus social support
   - A correlation heatmap

**File**: _data_analysis.py_

```bash
import io
import sys

import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns

# Check if the data location argument is provided
if len(sys.argv) != 2:
    print("Usage: python data_analysis.py <data_location>")
    sys.exit(1)

# Load the clean dataset (step 1)
df = pd.read_csv(sys.argv[1])

# Data summary (step 2)
print("Data Summary:")
summary = df.describe()
data_head = df.head()
print(summary)
print(data_head)

# Collecting data information
buffer = io.StringIO()
df.info(buf=buffer)
info = buffer.getvalue()

## Write metrics to file
with open("processed_data/summary.txt", "w") as outfile:
    f"\n## Data Summary\n\n{summary}\n\n## Data Info\n\n{info}\n\n## Dataframe\n\n{data_head}"

print("Data summary saved in processed_data folder!")

# Distribution of Happiness Score (step 3)
plt.figure(figsize=(10, 6))
sns.displot(df["Happiness Score"])
plt.title("Distribution of Happiness Score")
plt.xlabel("Happiness Score")
plt.ylabel("Frequency")
plt.savefig("figures/happiness_score_distribution.png")

# Top 20 countries by Happiness Score
top_20_countries = df.nlargest(20, "Happiness Score")
plt.figure(figsize=(10, 6))
sns.barplot(x="Country", y="Happiness Score", data=top_20_countries)
plt.title("Top 20 Countries by Happiness Score")
plt.xlabel("Country")
plt.ylabel("Happiness Score")
plt.xticks(rotation=90)
plt.savefig("figures/top_20_countries_by_happiness_score.png")

# Scatter plot of Happiness Score vs GDP per Capita
plt.figure(figsize=(10, 6))
sns.scatterplot(x="GDP per Capita", y="Happiness Score", data=df)
plt.title("Happiness Score vs GDP per Capita")
plt.xlabel("GDP per Capita")
plt.ylabel("Happiness Score")
plt.savefig("figures/happiness_score_vs_gdp_per_capita.png")

# Visualize the relationship between Happiness Score and Social Support
plt.figure(figsize=(10, 6))
plt.scatter(x="Social Support", y="Happiness Score", data=df)
plt.xlabel("Social Support")
plt.ylabel("Happiness Score")
plt.title("Relationship between Social Support and Happiness Score")
plt.savefig("figures/social_support_happiness_relationship.png")

# Heatmap of correlations between variables
corr_matrix = df.drop("Country", axis=1).corr()
plt.figure(figsize=(12, 10))
sns.heatmap(corr_matrix, annot=True, cmap="coolwarm", square=True)
plt.title("Correlation Heatmap")
plt.savefig("figures/correlation_heatmap.png")
print("Visualizations saved to figures folder!")
```

## Set up requirements

We need to set up a _requirements.txt_ to install all required Python packages. It looks like:

```bash
pandas
numpy
seaborn
matplotlib
black
```

## Create Makefile

We create a file names _Makefile_ and start adding actions to the targets:

**File**: _Makefile_

```bash
RAW_DATA_PATH = "raw_data/WHR2023.csv"
PROCESSED_DATA_PATH = "processed_data/WHR2023_cleaned.csv"

install:
    pip install --upgrade pip && \
        pip install -r requirements.txt

format:
    black *.py --line-length 88

process: ./raw_data/WHR2023.csv
    python data_processing.py ${RAW_DATA_PATH}

analyze: ./processed_data/WHR2023_cleaned.csv
        python data_analysis.py $(PROCESSED_DATA)

clean:
    rm -f processed_data/* **/*.png

all: install format process analyze
```

To execute the target, we will use `make` tool and the `target name` in the terminal:

```bash
make format
```

Run the Python script for data processing.

```bash
make process
```

If we want to override the existing variable, we can provide an additional argument to the `make` command. In our case:

```bash
make process RAW_DATA_PATH="WHR2022.csv"
```

To automate the entire workflow, we’ll use all as a target, which will learn, install, format, process, and analyze targets one by one.

```bash
make all
```

# To be continued...
# Makefile-n-GithubAction
