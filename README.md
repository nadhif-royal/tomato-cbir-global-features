# Tomato Image Retrieval (CBIR) with Global Features

Content-based image retrieval (CBIR) system for tomato images, built for the course assignment **Global Feature Extraction and CBIR** (CIF61339 — Digital Image Processing and Computer Vision).

**Author:** Nadhif Rif'at Rasendriya (NIM 235150201111074)

---

## Overview

Given a query tomato image, the system ranks 5 database images from most to least similar. The pipeline has four steps:

1. Segment the tomato from the background.
2. Extract global features from the tomato pixels only.
3. Normalize and weight the features.
4. Rank the database by distance.

An optional section demonstrates local features (ORB) under rotation.

## Pipeline

| Step | Method |
|---|---|
| Segmentation | GrabCut (rectangle init) + dominant-hue filter in HSV + morphology |
| Color features | RGB mean/std, hue as (cos, sin), mean S and V |
| Shape features | Aspect ratio, circularity, solidity, extent |
| Texture features | GLCM (Haralick: contrast, homogeneity, energy, correlation) |
| Normalization | Min-max, parameters from the database only |
| Similarity | Weighted Euclidean distance (cosine as a comparison) |
| Ranking | `np.argsort` on the distances |

The final feature vector has **18 dimensions** (10 color, 4 shape, 4 texture). Group weights are **0.50 / 0.25 / 0.25**.

## Repository structure

```
.
├── Tugas_P17_P18_CBIR_Tomat_Nadhif_Rifat_Rasendriya.ipynb   # main notebook
├── dataset/                                                  # 7 tomato images
│   ├── db_1_merah_bulat.jpg
│   ├── db_2_hijau_mentah.jpg
│   ├── db_3_oranye_ceri.jpg
│   ├── db_4_merah_lonjong_plum.jpg
│   ├── db_5_kuning_bergerombol.jpg
│   ├── query_1_merah_ceri.jpg
│   └── query_2_hijau_tangkai.jpg
└── README.md
```

## How to run

### Google Colab
1. Open the notebook in Google Colab.
2. Upload the 7 images from `dataset/` when prompted, or put them in a folder named like the one set in `DATA_DIR`.
3. Run all cells (`Runtime → Run all`).

### Locally
```bash
git clone https://github.com/nadhif-royal/tomato-cbir-global-features.git
cd tomato-cbir-global-features
pip install numpy pandas opencv-python matplotlib scikit-image scipy jupyter
jupyter notebook
```

> The notebook reads images from the folder set in `DATA_DIR` (first line of cell 1.1). Set it to `"dataset"`.

## Results

| Query | Rank 1 | Distance | P@1 | AP |
|---|---|---|---|---|
| `query_1_merah_ceri` | `db_3_oranye_ceri` | 0.643 | 0.0 | 0.50 |
| `query_2_hijau_tangkai` | `db_2_hijau_mentah` | 0.476 | 1.0 | 1.00 |

Relevance was judged visually by color class (subjective).

- **Query 2** finds the matching green tomato at rank 1. Color decides the result (60% of the squared distance).
- **Query 1** returns an orange tomato at rank 1. The red tomato (`db_1`) is at rank 2. The shape group pulls the query toward `db_3`, and the result is sensitive to the weights (the rank-1 image becomes `db_1` when the color weight is ≥ 0.65).

## Key findings

- Color alone is the most informative group for ripeness.
- Min-max normalization is essential. Without it, GLCM contrast dominates and P@1 drops to 0 for both queries.
- Euclidean and cosine give the same rank-1 image for both queries.

## Limitations

- Very small database (5 images) and only 2 queries, so the results are not statistically meaningful.
- Color features are sensitive to lighting (for example the dark plum photo `db_4`).
- Shape is computed on the whole mask, not per fruit. Clustered tomatoes give irregular silhouettes.
- Segmentation is imperfect. Thick sepals on clustered fruit may remain, and the top of `db_2` is clipped by a highlight.
- Relevance labels are visual and subjective.

## Dataset and image sources

All images come from Wikimedia Commons (accessed 15 October 2026). Each image keeps its own license, so check the original page before reuse.

| File | Source |
|---|---|
| `db_1_merah_bulat.jpg` | File:Tomato je.jpg — Softeis (2005) |
| `db_2_hijau_mentah.jpg` | File:Green Tomato.jpg — Cooper, C. / Prosthetic Head (2016) |
| `db_3_oranye_ceri.jpg` | File:Orange tomato (6951843461).jpg — Rulkens, T. (2012) |
| `db_4_merah_lonjong_plum.jpg` | File:Plum-tomato.jpg — Wikimedia Commons (2016) |
| `db_5_kuning_bergerombol.jpg` | File:Lufa Farms Landmarks - Yellow Tomato.jpg — Lufa Farms (2012) |
| `query_1_merah_ceri.jpg` | File:Red tomato (6951842889).jpg — Rulkens, T. (2012) |
| `query_2_hijau_tangkai.jpg` | File:Unripe tomato on its stem.jpg — Toony (2006) |

## References

- Course materials P17 (Global Feature Extraction and CBIR) and P18 (Local Feature Extraction).
- Rother, C., Kolmogorov, V., Blake, A. (2004). *GrabCut: Interactive Foreground Extraction using Iterated Graph Cuts.* ACM TOG 23(3).
- Haralick, R. M., Shanmugam, K., Dinstein, I. (1973). *Textural Features for Image Classification.* IEEE Trans. SMC 3(6).
- Rublee, E. et al. (2011). *ORB: An efficient alternative to SIFT or SURF.* ICCV.

## Academic note

This repository contains coursework and is shared for documentation purposes.
