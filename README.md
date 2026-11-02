
### Medical Record Semantic Retrieval System

A Python-based semantic information-retrieval prototype that retrieves
relevant existing medical records from free-text queries using sentence
embeddings and vector similarity.
Disclaimer: This is an educational information-retrieval project,
not a clinical decision-support system. It does not diagnose, triage,
recommend treatment, summarize records, or generate clinical answers.

Table of Contents
- Project Overview
- Objectives
- Dataset
- Embedding Models
- System Workflow
- Experiments
- Evaluation Metrics
- Results
- Success and Failure Analysis
- Final Model Selection
- Repository Structure
- Installation and Execution
- Limitations
- Future Improvements
- Safety and Data Privacy
Project Overview
The objective is to build a semantic retrieval system for medical
records. Given a free-text query, the system converts the query and
existing medical record notes into numerical vector representations
called embeddings. It compares the query embedding with record
embeddings and retrieves the most relevant existing records.

Unlike keyword-based search, semantic retrieval compares embedding
representations of meaning rather than relying only on exact word
matches.
Objectives
- Build a semantic retrieval pipeline for medical records.
- Generate embeddings using three embedding models.
- Compare native and PCA-reduced embedding dimensions.
- Implement cosine similarity and dot product.
- Compare normalized and unnormalized vector behaviour.
- Implement Top-K retrieval.
- Evaluate Precision@K, Recall@K, and Mean Reciprocal Rank (MRR).
- Measure query latency and embedding storage.
- Visualize embeddings using PCA and UMAP.
- Analyze successful and unsuccessful retrieval examples.
Dataset
  Dataset component                           Count
  Medical records                               289
Total queries                                  57
Queries with usable relevance labels           54
Queries without usable relevance labels         3

The medical-record dataset contains stable record IDs and note text. The
query dataset contains query IDs, query text, and lists of relevant
medical record IDs.

The three queries without usable relevance labels were excluded from
metric evaluation. Their relevance was not inferred from model
predictions. All three models were evaluated using the same corpus,
labelled queries, and relevance labels.
Embedding Models
  Model          Model identifier                              Native dimension
  MiniLM         sentence-transformers/all-MiniLM-L6-v2                   384
MPNet          sentence-transformers/all-mpnet-base-v2                  768
S-PubMedBERT   pritamdeka/S-PubMedBert-MS-MARCO                         768

Each model generated embeddings for both medical-record notes and search
queries.
System Workflow
1. Load and validate the medical-record and query datasets.
2. Clean the required text and relevance-label fields.
3. Load the three embedding models.
4. Generate embeddings for records and queries.
5. Create native-dimensional and PCA-reduced representations.
6. Calculate similarity scores between query and record vectors.
7. Rank records by similarity.
8. Retrieve the Top-K distinct records.
9. Evaluate predictions against known relevance labels.
10. Compare model performance, storage, and latency.
Medical Records                         Search Queries
(record_id, note_text)                  (query_id, query_text)
       |                                         |
       v                                         v
Text preprocessing                       Text preprocessing
       |                                         |
       v                                         v
Embedding model                          Same embedding model
       |                                         |
       v                                         v
Corpus embeddings                        Query embedding
       |                                         |
       +------------------+----------------------+
                          |
                          v
              Optional PCA transformation
          (fit on corpus; transform queries)
                          |
                          v
                Similarity calculation
              (Cosine / Dot Product)
                          |
                          v
                   Top-K ranking
                          |
                          v
              Retrieved medical record IDs
                          |
                          v
          Evaluation against relevance labels
Experiments
Dimensionality reduction
Three settings were evaluated for each model:
- Native embedding dimension
- PCA 128D
- PCA 64D
PCA was fitted using corpus embeddings only. The same fitted PCA
transformation was applied to query embeddings so that corpus and query
vectors remained in the same reduced vector space.

PCA 2D and UMAP 2D were also used for visualization and exploration.
These two-dimensional representations were not used for retrieval.
Similarity measures
- Cosine similarity: compares the direction of two vectors.
- Dot product: calculates the sum of element-wise vector products
  and can be affected by vector magnitude.
- Normalized dot product: dot product calculated after
  L2-normalizing vectors.
For L2-normalized vectors, cosine similarity and dot product produce
equivalent rankings, subject to numerical precision. Raw dot product can
produce different rankings when vector magnitudes differ.

The inspected MiniLM and MPNet Sentence Transformer pipelines included a
normalization module. S-PubMedBERT did not show the same normalization
module in the inspected model structure. Normalization behaviour was
considered during the comparisons.

Similarity scores are not calibrated probabilities and should not be
interpreted as clinical confidence.
Top-K retrieval
The retrieval implementation ranks records by similarity and returns the
highest-ranked records. The final comparative evaluation reported here
uses K = 5.
Evaluation Metrics
  Metric                              Description
  Precision@5                         Relevant records retrieved in the
Top-5 divided by 5

  Recall@5                            Relevant records retrieved in the
Top-5 divided by the total number
of known relevant records for that
query

  MRR@5                               Mean reciprocal rank of the first
relevant result within the Top-5

  Query latency                       Time measured for query encoding
and retrieval in the local
experiment
Embedding storage                   Approximate storage required for
corpus embedding vectors
Aggregate retrieval metrics were calculated over the same 54 labelled
queries for every model.
Results
Best 128D configuration for each model
The following table shows the selected similarity configuration for each
model at 128 dimensions. Cosine similarity and normalized dot product
had identical reported retrieval metrics in these configurations.
  Model          Similarity     Precision@5     Recall@5        MRR@5  Latency (ms)     Corpus
storage
(MB)
  MiniLM         Cosine              0.7630       0.4028   0.9506   41.8511     0.1411

  MPNet          Cosine          0.7667   0.4088       0.8935      209.2649     0.1411
S-PubMedBERT   Cosine              0.7074       0.3767       0.9228      138.8546     0.1411
Interpretation
- MiniLM achieved the highest MRR@5 and the lowest measured query
  latency among the selected 128D configurations.
- MPNet achieved the highest Precision@5 and Recall@5.
- S-PubMedBERT achieved an MRR@5 between MiniLM and MPNet, but lower
  precision and recall.
- All three models required the same 128D corpus storage.
Native dimension versus PCA dimensions
For each model and dimension, the table shows the highest MRR@5 among
the evaluated similarity methods at that setting.

  Model            Native MRR@5   PCA 128D MRR@5   PCA 64D MRR@5
  MiniLM             0.9630           0.9506          0.9179
MPNet              0.9506           0.8935          0.8549
S-PubMedBERT       0.9506           0.9228          0.9244

Native dimensions generally produced stronger ranking performance in
these experiments. The impact of PCA varied by model. S-PubMedBERT's PCA
64D result was slightly higher than its PCA 128D result for MRR@5.
Embedding storage
  Model              Native storage   PCA 128D storage    PCA 64D storage
(MB)               (MB)               (MB)
  MiniLM                     0.4233             0.1411             0.0706

  MPNet                      0.8467             0.1411             0.0706
S-PubMedBERT               0.8467             0.1411             0.0706
PCA 128D reduced approximate corpus embedding storage by 66.7% for
MiniLM and 83.3% for MPNet and S-PubMedBERT. PCA 64D reduced storage
further, but generally involved a retrieval-quality trade-off.
Query latency
Measured latency for the selected 128D cosine configurations:

  Model            Measured latency (ms)
  MiniLM                         41.8511
MPNet                         209.2649
S-PubMedBERT                  138.8546

Latency changed very little between native, PCA 128D, and PCA 64D
settings in the notebook measurements. For this relatively small
dataset, query encoding contributed substantially to the measured time,
so reducing vector dimensions did not produce a large end-to-end latency
improvement.

These are local experimental measurements, not guaranteed production
response times. Confirm the exact timing methodology and median
calculation before treating these values as production benchmarks.
Success and Failure Analysis
Five examples were inspected using the actual relevance labels and each
model's Top-5 predictions.

For this example-level analysis, a model was counted as successful if at
least one of its Top-5 retrieved record IDs matched an actual relevant
record ID.
Example 1 --- Q001: Wheezing and tight chest
Outcome: All three models succeeded.

  Model          Matched relevant IDs
  MiniLM         MR0049, MR0141, MR0210, MR0235
MPNet          MR0049, MR0086, MR0141, MR0210
S-PubMedBERT   MR0049, MR0086, MR0159, MR0210, MR0235

All three models retrieved multiple records that matched the relevance
labels.
Example 2 --- Q042: Headache with no head injury
Outcome: Two models succeeded.

  Model          Matched relevant IDs
  MiniLM         MR0076, MR0081, MR0162
MPNet          MR0076, MR0081, MR0255
S-PubMedBERT   None

MiniLM and MPNet retrieved relevant records, while S-PubMedBERT did not
retrieve a labelled relevant record in its Top-5 results.
Example 3 --- Q039: N/V/D x 2 days
Outcome: All three models failed.

  Model          Matched relevant IDs
  MiniLM         None
MPNet          None
S-PubMedBERT   None

None of the models retrieved a record matching the known relevance
labels in its Top-5 results. The query contains the abbreviation
"N/V/D". Abbreviation handling is one possible area for further
investigation, but this example alone does not establish the cause of
the failure.
Example 4 --- Q002: Fever with productive cough and lung infection
Outcome: All three models succeeded.

  Model          Matched relevant IDs
  MiniLM         MR0132, MR0137, MR0282
MPNet          MR0091, MR0282
S-PubMedBERT   MR0132, MR0137, MR0271, MR0282

All three models retrieved at least one relevant record.
Example 5 --- Q029: Feeling sad and lost interest in everything
Outcome: All three models succeeded.

  Model          Matched relevant IDs
  MiniLM         MR0032, MR0247
MPNet          MR0032, MR0084, MR0111, MR0247, MR0296
S-PubMedBERT   MR0276, MR0296

All three models retrieved at least one relevant record. MPNet retrieved
the largest number of matching records in this example.
Overall success/failure distribution
The full query-level analysis covered all 54 labelled queries.

  Outcome                    Number of queries   Percentage
  All 3 models succeeded                    52        96.3%
2 models succeeded                         1         1.9%
All 3 models failed                        1         1.9%
Total                             54     100%

This binary success measure is not the same as Precision@5, Recall@5, or
MRR@5. It only indicates whether at least one relevant record appeared
in the Top-5 results.
Final Model Selection
The recommended configuration for this prototype is:

MiniLM + PCA 128D + Cosine Similarity + Top-5 Retrieval

Reasons:
1. It achieved the highest MRR@5 among the three selected 128D
   configurations.
2. It had the lowest measured query latency.
3. Its approximate corpus embedding storage was 0.1411 MB.
4. Cosine similarity and normalized dot product produced equivalent
   reported rankings.
5. It provided a practical balance between ranking quality, storage,
   and speed.
MPNet remains a useful alternative when Precision@5 and Recall@5 are
prioritized over latency and MRR.

This recommendation is specific to the dataset, query set, relevance
labels, and experimental setup used in this project. It is not a
universal ranking of embedding models.
Repository Structure
Suggested GitHub repository layout:
medical-record-semantic-retrieval/
│
├── README.md
├── Semantic_Retrieval_system.ipynb
├── data/
│   ├── medical_records.json
│   └── queries.json
│
├── results/
│   ├── figures/
│   │   ├── pca_visualization.png
│   │   └── umap_visualization.png
│   └── evaluation_results.csv
│
└── requirements.txt
Adjust filenames and folders to match the files actually present in your
repository. Do not upload private or identifiable patient data.
Installation and Execution
1. Clone the repository
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd medical-record-semantic-retrieval
2. Create a virtual environment
Windows:
python -m venv .venv
.venv\Scripts\activate
macOS/Linux:
python -m venv .venv
source .venv/bin/activate
3. Install dependencies
pip install numpy pandas scikit-learn sentence-transformers matplotlib umap-learn jupyter
If the notebook uses additional libraries, add them to the installation
command and to requirements.txt.
4. Run the notebook
jupyter notebook
Open Semantic_Retrieval_system.ipynb, ensure dataset paths point to
the correct local files, and run the notebook cells in order.

The first run may download the pretrained embedding models. Model
loading time should be distinguished from query latency when reporting
benchmarks.
Limitations
- The dataset contains 289 medical records, which is relatively small.
- Only 54 queries had usable relevance labels.
- Three queries were excluded because usable relevance labels were
  unavailable.
- Retrieval performance depends on the quality and completeness of the
  relevance labels.
- Performance may change with different records and query types.
- PCA can remove information useful for retrieval.
- Latency depends on local hardware and software configuration.
- Five qualitative examples do not explain every success or failure in
  the dataset.
Future Improvements
1. Investigate failed queries, including abbreviation-based queries.
2. Review relevance labels with an appropriate domain expert.
3. Test additional approved clinical embedding models.
4. Explore abbreviation normalization or query expansion as separate
   experiments.
5. Evaluate on a larger and more diverse labelled dataset.
6. Repeat latency measurements and report the median consistently.
7. Build a simple interface for entering a query and inspecting
   retrieved record IDs, ranks, and similarity scores.
Future changes should be evaluated against the same corpus and relevance
labels when comparing with this baseline.
Safety and Data Privacy
- This project is an educational semantic information-retrieval
  prototype, not a clinical system.
- Use synthetic or appropriately approved de-identified data only.
- Do not upload patient names, dates of birth, addresses, medical
  record numbers, or other direct identifiers to GitHub.
- Do not send sensitive medical data to external embedding services
  unless explicitly approved.
- Similarity scores are not probabilities or measures of clinical
  correctness.
- Retrieved records must not be used to diagnose, triage, or recommend
  treatment.
Conclusion
This project implemented a semantic medical-record retrieval workflow
using MiniLM, MPNet, and S-PubMedBERT.

The experiments compared native and PCA-reduced dimensions, cosine
similarity, dot product, normalized dot product, Top-K retrieval,
evaluation metrics, storage, and latency.

At 128 dimensions, MiniLM with cosine similarity achieved an MRR@5 of
0.9506 and a measured query latency of approximately 41.85 ms. MPNet
achieved the highest Precision@5 and Recall@5 among the selected 128D
configurations.

The query-level analysis showed that all three models retrieved at least
one relevant record for 52 of the 54 labelled queries under the
project's Top-5 success definition.

Based on the measured results, MiniLM with PCA 128D and cosine
similarity is recommended as the final configuration for this prototype.

The project demonstrates how embedding models, dimensionality reduction,
similarity measures, and evaluation metrics can be combined to build a
semantic information-retrieval system for existing medical records.
