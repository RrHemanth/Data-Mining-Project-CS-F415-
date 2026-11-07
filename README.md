# Unveiling Customer Segments in Credit Card Data

Clustering 8,950 credit card customers by how they spend, borrow and pay, using K-Means, hierarchical clustering and DBSCAN, then reading the clusters back as customer segments.

Course project for CS F415 Data Mining at BITS Pilani, Hyderabad Campus, supervised by Dr. Aruna Malapati. Done as a team of three: R Hemanth Reddy, Kulkarni Sreyas and Mittapalli Sreetheerdha. The write-up is in `Project_Report.pdf`.

## The data

One row per customer, 18 columns: balance, purchases (total, one-off, installment), cash advances, how often each happens, credit limit, payments, minimum payments, share of full payments and tenure. `CUST_ID` is only an identifier.

Only `CREDIT_LIMIT` and `MINIMUM_PAYMENTS` have missing values. I filled them with the column mean rather than dropping rows, which would have cost 314 customers.

## What the notebook does

1. Cleans the data and compares dropping against mean-filling missing values.
2. Summarizes spending by card tenure.
3. Builds interaction features from balance-update frequency and purchase frequencies.
4. Tries simple random and stratified sampling, checking that group proportions hold.
5. Bins the frequency columns, splits tenure into two groups, then scales with z-score and min-max.
6. Reduces the data to 3 principal components with PCA.
7. Selects features with a variance threshold, a correlation filter (pairs above 0.8) and coefficient of variation.
8. Clusters with K-Means, agglomerative (Ward) and DBSCAN, and plots each in 3D.

## Choosing the number of clusters

The methods did not agree, so this part took some judgment:

- Elbow method suggested 3.
- Silhouette score alone suggested 2.
- The largest gap in the Ward dendrogram gave 4.

Two clusters was too coarse to say anything about customers, so I used 4 for K-Means and hierarchical clustering. For DBSCAN, I searched `eps` (1 to 10) and `min_samples` (5 to 19), guided by a k-distance plot, and kept the best silhouette score among settings that produced exactly 4 clusters.

## Results

| Method | Clusters | Silhouette | Davies-Bouldin | Calinski-Harabasz |
|---|---|---|---|---|
| K-Means (k = 4, k-means++ start) | 4 | 0.595 | 0.589 | 12,601 |
| Agglomerative (Ward) | 4 | 0.618 | 0.572 | 11,647 |
| DBSCAN (eps 4.82, min_samples 5) | 4 + 142 noise points | 0.783 | 1.953 | 879 |

Cluster sizes: K-Means 6,849 / 1,664 / 390 / 47. Agglomerative 6,998 / 1,684 / 240 / 28. DBSCAN 8,785 / 9 / 8 / 6, with 142 outliers.

K-Means and hierarchical clustering produce a similar picture: one very large group of ordinary customers, two mid-sized groups and a small group of extreme accounts. DBSCAN's higher silhouette is misleading, since three of its four clusters have only 6 to 9 customers. Its other two metrics say the same: a Davies-Bouldin of 1.95 (lower is better) and a Calinski-Harabasz of 879 are far worse than the other two methods. What it does well is isolating the 142 outliers.

## What the segments look like

Reading each cluster's average balance, purchase mix, cash advances, credit limit and payments gives these K-Means segments (full profiles are in the report):

- **Cash convenience consumers:** moderate balances, heavy on cash advances, rarely buy in installments.
- **Installment users:** large, infrequent purchases, mostly paid in installments, very little cash advance.
- **Full-payment pioneers:** the most active cardholders, high purchase and installment frequency, rarely take cash advances.
- **Starters:** low balances and limits, little activity of any kind. Likely newer or occasional users.

These names come from reading cluster averages. They are interpretations, not validated labels.

## Running it

Needs Python 3 with `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn` and `scipy`.

1. Put the credit card CSV (`cc.csv`) where the notebook can read it.
2. Change the file path in the second cell to yours.
3. Run the cells top to bottom.

```
pip install numpy pandas matplotlib seaborn scikit-learn scipy
jupyter notebook
```
