# Seurat plugin for TBtools

## Install seurat plugin

Open TBtools and install the Rserver plugin in the plugin store first.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/67789d57-03cb-4925-a5bb-3e934940e367)

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/a9c8f804-a89f-4670-a778-7b339abb8918)

Install the seurat plugin.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/7558943a-286e-4562-9425-be1bcc022c80)

After successful installation, open seurat plugin and click "start" to start analyzing.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/8b991636-5e9a-4aa2-b8c5-ef44877908cb)


## Uploading user data

Input data, currently supports two formats of expression matrix:
- The sparse matrix output through Cell Ranger, including three files: barcodes.tsv, genes.tsv, and matrix.mtx.
- Count matrix. Generally download single cell public data in GEO or other public database storage platform, most of them are Count matrix. Note that the matrices require comma delimiters.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/1099557c-4e35-4954-8e61-36d50399c4fe)


## Analyzing examples (demo data)

### Data preprocessing and creat Seurat objects

Once the data is uploaded, the user can access the count matrix through the "Data Preview" section located on the right side of the interface. By default, only the first 30 cells are displayed, but the user has the option to browse or retrieve the entire count matrix.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/25fe0b78-0efa-45ac-a3c4-966059b3a7db)

Data preprocessing, which involves filtering out non-expressed genes and cells with minimal gene expression, significantly speeds up downstream computation. Filtering by two parameters：
1. min.cells, keep genes whose expression is detected in at least a few cells, default is 3 cells.
2. min.features, keep cells with at least how many genes are detected to be expressed, default is 200 genes. If the cell expresses too few genes, then it is likely to be an abnormal cell and filter it out.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/2814f166-77cb-43b9-ab60-1b0dccb19152)


## Quality control

In this step, we visualize the QC metrics for the data, allowing users to freely explore and assess the normality of their data. we provide an interface that allows the user to calculate the proportion of some genes expressed in all cells. For instance, users can easily determine the proportion of mitochondrial gene expression, a widely used metrics for identifying and filtering out abnormal cells. Typically, low-quality/dying cells often exhibit extensive mitochondrial contamination and needs to be filtered out. Additionally, users have the option to calculate the expression levels of ribosomal genes and similar markers.

### QC step1: Calculat Gene Expression Ratio (e.g. MT gene)

Here, we use mitochondrial genes as an example. In the Arabidopsis genome (TAIR10), mitochondrial gene name begin with "ATMG", allowing us to identify all mitochondrial genes in Arabidopsis using the regular expression "^ATMG". For humans, mitochondrial genes begin with MT-, while for mice, they start with mt-.

We provide `two modes` of gene search:
1. regular matching, matching sets of genes with specific patterns such as mitochondrial genes. Example: ^ATMG.
2. User-defined gene set. The user input a customized gene set for calculation, one gene per line.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/5b3bed34-1ada-4c96-a60e-3b978c35d8e4)

> **Tips: If the species you are studying is not a model species such as Arabidopsis, how do we determine its mitochondrial genes?**
> - Download the mitochondrial gene annotation file for your species in the Ensembl database and check if there is a consistent pattern of mitochondrial gene IDs for that species, if a pattern exists then you can use a regular expression to match it, e.g., ^ATMG. If there is not then TBtools extracts all of the mitochondrial gene IDs, and then, using mode2, inputs a collection of mitochondrial gene IDs and performs the calculation. If not, the user can use TBtools to extract all mitochondrial gene IDs. then using mode2, input the mitochondrial gene ID set to calculate.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/dc00846d-3fbd-4378-9b2f-c88c77b14a67)

### QC Step2: Visualization & Filter Low Quality Cells

In Step2, we visualized several QC metrics, including user-specified calculations of gene expression (e.g. mitochondria) in Step1. The visualized metrics are listed below:
- Proportion of mitochondrial genes per cell (if user has calculated in Step1).
- Number of genes detected per cell (nFeature_RNA). Low-quality cells or empty droplets will often have very few genes and cell doublets or multiplets may exhibit an aberrantly high gene count.
- Total number of UMIs measured per cell (nCount_RNA).

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/254a3d76-c4cd-43d4-94de-b5689a5ee35d)

**Users can filter according to their data situation.**

### Exploring the quality of single-cell data

We also performed correlation analysis and visualized the feature values. Users can assess the normality of the data by examining the correlation between nFeature_RNA and nCount_RNA. In theory, cells that capture more UMIs should also get more genes, resulting in a strong positive correlation between these two metrics.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/4f558ae5-f076-4859-a4be-4457bd0700af)


## Data normalizing and scaling

After QC, the next step is to Normalize, Find Variable Genes, and Scale the data.
- Normalize the data. A global-scaling normalization method "LogNormalize" that normalizes the feature expression measurements for each cell by the total expression, multiplies this by a scale factor (10,000 by default), and log-transforms the result (The normalized formula is: A = log( 1 + ( UMIA ÷ UMITotal ) × 10000). According to seurat official recommendations, in general, the default parameters are suitable for most data.
- Find Variable Genes. That is calculating a subset of features that exhibit high cell-to-cell variation in the dataset (i.e, they are highly expressed in some cells, and lowly expressed in others). Focusing on these genes in downstream analysis helps to highlight biological signal in single-cell datasets. By default, seurat return 2,000 features per dataset.
- Scale Data. A linear transformation ("scaling") that is a standard pre-processing step prior to dimensional reduction techniques like PCA. The purpose of Scaling the data is (1)Shifts the expression of each gene, so that the mean expression across cells is 0; (2)Scales the expression of each gene, so that the variance across cells is 1. After such treatment, the influence of highly expressed genes can be eliminated.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/6924cf45-d9c2-4e30-813a-10d94aa76688)

## Perform PCA linear dimensionality reduction.

By default, the PCA dimensional reduction calculates the first 50 principal components.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/a0c7f914-d5d2-41c4-8507-3547498e6ae5)
![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/d986fce9-155a-4f3a-b3f8-0c15cf12a2b0)
![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/b13df7db-5004-41c7-834b-8c1f4156d885)

## Cell clustering

**Before performing cell clustering, we first have to determine two parameters: `the number of PCs` and `Resolution`**
1. When determining the number of principal components (PCs) to use, it is crucial to strike the right balance. Using too many PCs can introduce interference in the clustering process, while using too few PCs may not adequately explain the dataset. Therefore, we recommend using the Elbow Plot method to determine the optimal number of PCs to use. In the Elbow Plot, the number of PCs that correspond to the "elbow" should be selected for clustering. These PCs explain a significant portion of the dataset.
2. Resolution. The Resolution parameter determines the number of clusters obtained from downstream clustering. Here, we recommend using clustree to determine the optimal resolution for your data. clustree allows you to perform clustering at various resolutions and visualize the results, enabling you to assess the number of different cell clusters obtained at each resolution.

> Tips: If Resolution is chosen to be small, the clusters will be divided into fewer clusters, which will merge some similar cells into one big cluster, thus lead to challenges in subsequent cell type identification and the loss of valuable cell type information. If the Resolution is too large, the cluster will be divided into too many clusters, which will increase the difficulty of subsequent cell type identification. Therefore, we usually choose a medium value for Resolution, so that clusters with the same marker gene and similar distance in umap/tSNE can be categorized as the same cell type when identifying cell types.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/1b632d3a-f309-4fa9-860e-2297704254d5)

After determining the number of PCs and Resolution, then you can click the button for cell clustering.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/a8f68ef4-1f65-4514-b309-9972d808a4e7)


## Perform non-linear dimensional reduction (UMAP/tSNE)

Once the cell clustering is completed, the user can click the button to perform UMAP/tSNE non-linear dimensional reduction and visualize the results.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/97b7f548-56c9-48ba-bf9b-14bfecc34156)

## Marker gene identification/differential expression analysis

Marker gene identification, in essence, is actually to perform differential analysis to identify the differentially expressed genes. Marker genes of a cluster are typically specifically expressed only in that cluster and not in other clusters. Therefore, when identifying markers, we usually set the parameter "only.pos=TRUE," which means that only differentially up-regulated genes are retained.

In this module, three modes for identifying marker genes are provided: 
1. One-click identification of marker genes for all clusters (slower, runtime proportional to number of cells and clusters)；
2. Identifies user-specified marker genes for a particular cluster;
3. The user can specify the identification of marker genes between two specific clusters, conducting a differential expression analysis for these clusters.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/2cfb71e2-356f-4c6a-8eb0-1a2a3cb7eaba)

## Marker gene visualization

We provide several different visualizations to show marker genes.

- `Heatmap` display of marker genes in each cluster, user can select the number of marker genes to be displayed.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/806f166b-3cb3-469d-8dc5-65b7d9baa51b)

- `Dot plot`

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/2a9bc4a9-dc72-4f38-967a-fd2867d28e52)

- User inputs genes or gene set for Violin plot visualization.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/fbb82cab-5c7e-4a7e-951d-80f89eef1413)

- User inputs marker gene or gene set and perform feature Plot visualization, mapping the expression of marker genes to UMAP or tSNE.

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/de56cef9-76a5-488c-bd28-42821b0efa31)

- Ridge plot

![image](https://github.com/athanasy-ljw/Seurat_Plugin_for_TBtools/assets/53424383/829b9c83-b522-4430-bf1f-f55c0f425601)



