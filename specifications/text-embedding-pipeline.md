I want to create a Python script to process the given data file to generate a tsv file for visualization.

# Input data

The input file is a TSV file. User will provide a path to the file, e.g., `/home/xxx/xxx.tsv`

The script should read this file to understand its structure. The file contains fields such as CUI, Terms, and SAB. You can read a few lines to get better understanding.


# Processing steps

1. Prepare text for embeddings. For each row, generate the text used for embedding as:

```text = f"{Terms} | {SAB}"```


2. Generate embeddings. Use the following model: 

```google/embeddinggemma-300m```

Load the model with `sentence-transformers`.

Requirements:
- Use GPU if available
- Use a small batch size (e.g., 16) to reduce VRAM usage
- Print progress information while generating embeddings
- Since generating embeddings can be expensive, cache the results locally. If a text has already been embedded before, load it from the cache instead of recomputing.


3. Reduce dimensionality. After generating embeddings, keep only the first 256 dimensions of each embedding vector. Then use openTSNE to reduce the embeddings to 2D. Enable verbose output


4. Generate the output TSV. The final TSV file should contain the following columns:

```
- pid: use CUI
- title: split Terms using | and use the first term
- year: set to 2026
- x: The 2D coordinates from TSNE result
- y: The 2D coordinates from TSNE result
- size: Number of terms in Terms after splitting by |
- keywords: The remaining terms joined by |
```

# Tech stack and requirements

- Python
- Print useful information during execution
- Input data may contain millions of rows, do NOT load everything into memory at once, use streaming loading.
- Python’s default CSV field-size limit could be small. Increase it to a large value.
- Keep the script simple and readable
