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

