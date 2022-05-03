Design Strategy
===============

See EnhDesignMotifBased.ipynb and RankCandidates.ipynb. They can be run linearly but not all cells need to be executed-- exercise caution.

Preparation:
------------
- We have 5 folds of models trained for each cell type
- Clustered TF-MoDISco motifs across cell types (see data) for both positive and negative motifs
- Each clustered motif has a PWM and list of constituent members
- For first model fold, predict counts on 10k peaks for each model to get distribution of predicted counts in peaks. Use this for calibrating across cell types.
- Also filtered GC matched negative regions for each cell type ("filtered negative") to those that don't overlap with any cell type's peaks (not just its own).

Designing candidates:
---------------------
In this round we focused on two cell types- Horizontal and Retinalganglion. Aim is to generate sequences that are specifically accessible in one cell type and not in any other.
- For positive motifs, keep only those found in <=3 cell types
- For each TARGET cell type for which an exclusively accessible sequence needs to be designed, filter to positive motifs that are found in that cell type and negative motifs not found in that cell type
- Objective function: quantile of predicted counts in TARGET - max(quantile of predicted counts in all cell types != TARGET)
- Use first 3 folds of models in design stage 
- To generate a sequence (beam_optimize_motif):
    - Choose a desired design width (DES_WIDTH). All changes are made in the central DES_WIDTH bases of sequence (with some overflow on the right end if motif is inserted right at the end).
    - Start with a filtered negative for the TARGET cell type
    - Keep a beam size of 3 sequences
    - Each step consists 1 round of motif insertion and 2 rounds of single base mutation
    - In motif round, for each sequence in beam, all candidate motifs are inserted at 10 random positions (for each orientation), and scored. Top 3 (beam size) as per objective are retained
    - In single base mutation round, for each sequence in beam, each mutation (ACGT) is randomly inserted at 10 random positions and scored. Top 3 as per objective are retained.
    - In each round a randomly chosen fold (0, 1 or 2) of the model is used to avoid being swayed by any one fold.
    - 5 total steps are performed (5 x (1 x motif + 2 x single base)).
    - Best sequence is retained as a candidate, central DES_WIDTH + 20 bp is cropped (10bp on either side).
    
Performed this 100 times for each target separately.

Ranking candidates:
-------------------
- Use remaining 2 folds of models in ranking stage.
- Use 100 random filtered negative for the TARGET cell type as background sequence and insert each candidate
- Make predictions with all models (with remaining folds) and convert to quantile and take average quantile over folds and backgrounds
- Rank by average TARGET quantile - max(average quantile over all cell type != TARGET)
