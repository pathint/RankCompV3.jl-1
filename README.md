## About RankCompV3 

RankCompV3 is a differential expression analysis algorithm based on relative expression ordering (REO) of gene pairs. It can be applied to bulk or single-cell RNA-sequencing (scRNA-seq) data, microarray gene expression profiles and proteomics profiles, etc. When applied in scRNA-seq data, it can run in single-cell mode or pseudo-bulk mode. The pseudo-bulk mode is expected to improve the accuracy while decreasing cost. 

### Installation

The algorithm is implemented in Julia. Version 1.7 or later is recommended.

```julia
using Pkg
Pkg.add("RankCompV3")
```

### Examples

#### case 1: use test data for analysis. 

```julia
julia> using RankCompV3
# Use the default values for the following other parameters. If you need to modify the parameters, add them directly.
julia> result = reoa(use_testdata="yes")
# The results are saved in the current directory. Including result file, ctrl and treat group expression profile file, as well as pval, padj, 3 x 3 associative table parameters, Δ1, Δ2, se, z1 distribution, etc
julia> result
(19999×16 DataFrame
   Row │ Name     pval         padj        n11      n12      n13      n21      n22      n23      n31      n32      n33      Δ1           Δ2          se         z1
       │ String   Float64      Float64     Float64  Float64  Float64  Float64  Float64  Float64  Float64  Float64  Float64  Float64      Float64     Float64    Float64
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
     1 │ DE1      0.23566      0.716036     1532.0     75.0      0.0   1220.0  11010.0    602.0     16.0   2933.0   1674.0   1.91208      1.81954    0.0393608    48.5783
     2 │ DE2      0.280118     0.761567     2277.0    288.0      0.0    965.0  11514.0    372.0     20.0   2615.0   1011.0   1.74141      1.70287    0.0405313    42.9647
     3 │ DE3      0.0578546    0.359121     1576.0     37.0      0.0   1376.0   8716.0    293.0    113.0   5211.0   1740.0   3.05828      3.02011    0.0437133    69.9623
   ⋮   │    ⋮          ⋮           ⋮          ⋮        ⋮        ⋮        ⋮        ⋮        ⋮        ⋮        ⋮        ⋮          ⋮           ⋮           ⋮           ⋮
 19998 │ EE19999  0.344847     0.823375     1475.0    307.0      0.0   1756.0  15356.0    128.0      0.0     38.0      2.0   1.52307      1.41599    0.0525798    28.9668
 19999 │ EE20000  0.694484     0.980397     1571.0    315.0      0.0    979.0  15555.0    362.0      1.0    225.0     54.0   0.63329      0.577392   0.0481845    13.143
                                                                                                                                             19965 rows omitted, 19999×16
```

#### case 2: use local files for analysis. 

##### Input: 

- **metadata file (required).**

  The metadata file needs to contain two columns of information. The first column is the sample name that matches the expression profile, and the second column is the grouping information.

- **expression profile file (required).**

  The expression profile should contain gene, sample and expression value information. The row represents the gene, and the column represents the sample, among which the first behavior sample name and the first column gene name.

```julia
julia> using RankCompV3
# Use the default values for the following other parameters. If you want to modify the parameters, add them directly.
julia> reoa("/public/yanj/data/fn_expr.txt",
		"/public/yanj/data/fn_metadata.txt")
# Specific parameter example
julia> reoa("/public/yanj/data/fn_expr.txt",
    	"/public/yanj/data/fn_metadata.txt";
    	expr_threshold = 0,
    	min_profiles = 0,
    	min_features = 0,
    	pval_reo = 0.01,
     	pval_deg = 1.00,
     	padj_deg = 0.05,
    	use_pseudobulk = 0,
    	use_hk_genes = "yes"
    	hk_file = "HK_genes_info.tsv",
    	gene_name_type = "ENSEMBL",
    	ref_gene_max = 3000,
    	ref_gene_min = 100
    	n_iter = 128,
    	n_conv = 5,
    	cell_drop_rate = 0,
    	gene_drop_rate = 0,
    	work_dir = "./",
    	use_testdata = "no")
```

### Parameters for details

| Parameter      | Parameter types | Default value       | Parameters to describe                                       |
| -------------- | --------------- | ------------------- | ------------------------------------------------------------ |
| fn_expr        | AbstractString  | "fn_expr.txt"       | Gene expression profile file path. (required)                |
| fn_metadata    | AbstractString  | "fn_metadata.txt"   | Grouping information file path. (required)                   |
| expr_threshold | Number          | 0                   | Gene expression threshold.                                   |
| min_profiles   | Int             | 0                   | Include features (genes) detected in at least this many cells. |
| min_features   | Int             | 0                   | Include profiles (cells) where at least this many features are detected. |
| pval_reo       | AbstractFloat   | 0.01                | Stable threshold for p-value.                                |
| pval_deg       | AbstractFloat   | 0.05                | Significant  reversal threshold for p-value.                 |
| padj_deg       | AbstractFloat   | 0.05                | Significant reversal threshold for FDR  value.               |
| use_pseudobulk | Int             | 0                   | 0 for not using pseudobulk mode, 1 for automatic, 2~5 not used, 6~100 for number of pseudobulk profiles in each group. |
| use_hk_genes   | AbstractString  | "yes"               | Whether to use the housekeeping gene, yes or no.             |
| hk_file        | AbstractString  | "HK_genes_info.tsv" | House-keeping genes  file path.                              |
| gene_name_type | AbstractString  | "ENSEMBL"           | Available choices: Name, REFSEQ, SYMBOL, ENTREZID, ENSEMBL, UNIGENE and GENENAME. |
| ref_gene_max   | Int             | 3000                | If the number of available features is higher than this, take a random sample of this size. |
| refinement     | Int             | 100                 | If the number is lower than this, ignore the house-keeping genes. |
| n_iter         | Int             | 128                 | Max iterations.                                              |
| n_conv         | Int             | 5                   | Convergence condition: max. difference in the number of DEGs between two consective iterations. |
| work_dir       | AbstractString  | "./"                | Working Directory.                                           |
| use_testdata   | AbstractString  | "no"                | Whether to use the default provided test data for analysis, yes or no. |
