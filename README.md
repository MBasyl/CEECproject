## Data Preprocessing (corpus clean up)

>Files downloaded from: https://zenodo.org/records/5887101

### 1. Build dataset 
1. **Overview**
	-  tag_overview.py`: make inventory of all tags & attributes
	- `meta_plot.py` : make plots for high-level data exploration 
2. Standardize spelling: run **VARD** `cd [VARD_directory] && java -Xms256M -Xmx2G -jar gui.jar`
3. **Build dataset**: `preprocess.py` makes _"datasetRAW.csv"

### 2. Clean & prep
1. `data_clean.py` ➜ _"datasetCLEAN.csv"_ ➜ **manual check**
	- **Uncertain dates**: dropped letters with "?" in the title (saved separately as `dubbia_dataset.csv`).
	- **Language exclusion**: removed `GOWER_008` (French).
	- **Spelling normalisation**: recurring variants in function wrds missed by VARD (*tho'*, *wel*, *shoud*...) to modern forms.
	- **Opening/closing salutations**: regex removal of dateline + salutation + signature
	- **Function-word list (one-off)**: built from POS-tagged corpus vocabulary (freq > 5, > 3 docs); punctuation kept except commas (unreliable OCR).
2. `data_plot.py`: **Exploratory 3 & 5-year windows**; plotted word count by author × time × relationship, plus a social-metadata author summary.
3. `data_prep.py` ➜ _"dataset_ready_win3.csv"_
	- **Window size decision**: 3-year windows retain 28 authors versus 24 at 5 years
	- **Author floor**: kept authors with over 8,000 words (25 authors).
	- **Window filter**: windows of 800+ words (chunk size minus margin); authors with 3+ surviving windows.
	- **Generations**: 30-year bins from each author's earliest letter year.
	- **Window indexing**: per-author window rank, tagged first / middle / last.
	- **Aggregation**: all letters in an author-window concatenated; relationship code collapsed to the most frequent one.
	- **Chunking**: sentence-boundary chunks of ~900 tokens (±100); chunks under 800 tokens dropped, then 3+ windows per author re-checked.
	- **Generation merge**: the single gen-4 author folded into gen 3; windows re-indexed.
	- **Family filter**: relationship code (left off).
4. `prepR_lambdag.py` ➜ Final cleaning (==manual check on incipit/excipit of letters== for identical phrases within author) + TextDistortion for lambdaG

### 3. Verification pipeline (LambdaG)

- run **per generation** on distorted text, sentence-tokenised as 20-wrd sequence.
- **K** = ==either== loop through all texts ==OR== aggregate all texts from same gap (but some Ks & gaps over-reppresented)
- **Q** = loop through all texts different from K within that generation
- **Length cap**: each Q downsampled to the smallest K (~40-43 sentences) so all comparisons are equal-sized.
- **Reference set**: all other texts in the generation, excluding both the K and Q authors, downsampled so all authors are equally reppresented
- **Scoring**: LambdaG for every Q × K pair; metadata joined (k/q window, time gap, word count); length-corrected score (score / √word count).
- **Evaluation**: per-author performance via `idiolect::performance`; 

### 4. Modelling results (mixed effects)