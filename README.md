[README.md](https://github.com/user-attachments/files/33276346/README.md)
# Product Demand and Inventory Intelligence

Unsupervised segmentation of the BigBasket product catalogue with K-Means, hierarchical clustering and PCA, with segment-specific inventory and promotion strategies.

Built as a project-based learning assignment for *Fundamentals of Machine Learning (MLE2503)*, B.Tech (Artificial Intelligence and Machine Learning), Amity University Gurugram.

## Overview

Retailers often plan stock and promotions by product category, although products within one category behave very differently. This project asks whether naturally occurring product groups can be discovered from catalogue data, and how they could guide stocking and promotion decisions.

**Dataset.** The *BigBasket Products* dataset from Kaggle: 27,555 listings of an Indian online grocery retailer with price, market price, brand, category, sub-category, product type, rating and description. After removing 354 duplicates and one unnamed product, 27,200 products remain.

> **Data limitation.** The file is a catalogue snapshot with no units sold, stock levels or returns. Demand and inventory behaviour are therefore represented by proxies (discount depth, review traction, brand breadth, relative price), and the strategies below are hypotheses to validate against sales data.

## Approach

| Stage | What was done |
|---|---|
| Data quality | Duplicates removed, brand text standardised, 31.3% missing ratings imputed with the sub-category median plus a missing-rating flag |
| Features | Log price, discount percentage, log brand size, imputed rating, rating-missing indicator (price position and type density were tested and dropped) |
| Outliers | Detected with the 1.5 x IQR rule, retained as genuine premium products and log-transformed |
| Encoding | One-hot for category; frequency encoding for brand and type; price position within sub-category instead of one-hot sub-category |
| Split | Not used (unsupervised); validated by an 80/20 hold-out assignment and bootstrap resampling |
| Scaling | Standardised; unscaled discount percentage carried 98.5% of the variance |
| Models | K-Means (K = 2 to 10), Ward / average / complete hierarchical clustering, Gaussian mixture, PCA for visualisation |
| Evaluation | Elbow plot, silhouette score, Davies-Bouldin index, Calinski-Harabasz index, Adjusted Rand Index |

## Results

K-Means with **K = 5** gave the best overall result.

| Measure | Value |
|---|---|
| Silhouette score | 0.286 |
| Davies-Bouldin index | 1.27 |
| Calinski-Harabasz index | 7,339 |
| Hold-out agreement (ARI) | 0.983 |
| Bootstrap stability (ARI, mean of 10 runs) | 0.987 |

A silhouette near 0.29 indicates moderate, overlapping structure, so the segments should be read as planning groups and not as sharp classes.

### Model comparison (6,000-product sample)

| Solution | Silhouette | Davies-Bouldin |
|---|---|---|
| **K-Means, K = 5** | **0.291** | **1.266** |
| K-Means, K = 4 | 0.286 | 1.272 |
| Ward, K = 5 | 0.266 | 1.296 |
| Gaussian mixture, K = 4 | 0.163 | 1.858 |

Average linkage reaches a low Davies-Bouldin value only by creating a one-product cluster, so it is not a useful solution.

### Segments

| Segment | Products | Median price (INR) | Mean discount | Reading | Suggested action |
|---|---|---|---|---|---|
| Core everyday value | 11,365 (41.8%) | 127.5 | 3.8% | Probable fast-moving staples, all reviewed | High safety stock, frequent replenishment |
| Premium full-price, low review traction | 5,764 (21.2%) | 300.0 | 4.9% | Premium, probable slow-moving, unrated | Minimal stock, small-batch replenishment |
| Promotion-driven | 5,134 (18.9%) | 209.0 | 30.4% | Demand tied to promotions | Stock to the promotion calendar, measure uplift |
| Large-brand discounted, low review traction | 2,649 (9.7%) | 159.0 | 26.7% | Broad, low-priced, unrated | Lean stock, brand-level promotions |
| Low-rated, quality-risk | 2,288 (8.4%) | 297.3 | 10.6% | Average rating 2.44 | Reduce stock, check quality before reordering |

<p align="center">
  <img src="plots/07_elbow_silhouette_davies_bouldin.png" width="95%" alt="Elbow, silhouette and Davies-Bouldin plots"><br>
  <em>Choosing K: elbow, silhouette and Davies-Bouldin index for K = 2 to 10</em>
</p>

<p align="center">
  <img src="plots/12_pca_cluster_plot.png" width="60%" alt="Clusters in PCA space"><br>
  <em>The five clusters projected onto the first two principal components</em>
</p>

<p align="center">
  <img src="plots/14_cluster_profile_heatmap.png" width="60%" alt="Cluster profile heat map"><br>
  <em>Mean standardised feature values per cluster</em>
</p>

<p align="center">
  <img src="plots/09_dendrogram_ward.png" width="75%" alt="Ward dendrogram"><br>
  <em>Ward dendrogram on a 6,000-product sample</em>
</p>

## Repository structure

```
.
|-- Product_Demand_Inventory_Intelligence.ipynb   # executed notebook, commented step by step
|-- plots/                                        # 16 figures (PNG)
|-- report/
|   |-- Project_Report.pdf                        # detailed project report
|   |-- Project_Report.docx
|   `-- Notebook_Export.pdf                       # notebook exported to PDF
|-- requirements.txt
|-- LICENSE
`-- README.md
```

## Running the notebook

1. Search for the *BigBasket Products* dataset on Kaggle, download `BigBasket Products.csv` and place it in the repository root, next to the notebook. The file is not committed because the dataset belongs to its original publisher.
2. Install the dependencies and start Jupyter:

   ```bash
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   jupyter notebook Product_Demand_Inventory_Intelligence.ipynb
   ```
3. Run all cells in order. Random seed 42 is used throughout, so the results reproduce exactly. The figures are written to `plots/`.

## Limitations

- No sales, stock or return data: "fast-moving" and "slow-moving" are inferred from indirect indicators.
- About 31% of ratings are missing and two categories have none, so imputed ratings carry little information.
- The segments overlap (silhouette 0.29).
- Hierarchical clustering ran on a random sample of 6,000 products because the full distance matrix would need about 6 GB.

## Future work

Combine the catalogue with transaction data to build real demand features (units sold, sales velocity, stock turnover, return rate), add a time dimension for seasonality, and test the strategies in controlled pilots.

## References

1. MacQueen, J. (1967). Some methods for classification and analysis of multivariate observations. *Proc. Fifth Berkeley Symposium*, 1, 281-297.
2. Arthur, D. and Vassilvitskii, S. (2007). k-means++: the advantages of careful seeding. *SODA '07*, 1027-1035.
3. Rousseeuw, P. J. (1987). Silhouettes: a graphical aid to the interpretation and validation of cluster analysis. *J. Computational and Applied Mathematics*, 20, 53-65.
4. Davies, D. L. and Bouldin, D. W. (1979). A cluster separation measure. *IEEE TPAMI*, PAMI-1(2), 224-227.
5. Ward, J. H. (1963). Hierarchical grouping to optimize an objective function. *JASA*, 58(301), 236-244.
6. Hubert, L. and Arabie, P. (1985). Comparing partitions. *Journal of Classification*, 2, 193-218.
7. Pedregosa, F. et al. (2011). Scikit-learn: machine learning in Python. *JMLR*, 12, 2825-2830.

## Licence

Code and documentation are released under the MIT Licence (see `LICENSE`). The BigBasket dataset is not included and remains subject to its original terms on Kaggle.
