# SPRING_py3

## Conda Environment and Installing Python libraries

To run SPRING Viewer locally, you'll need to set up a conda environment for it and install its dependencies:

`numpy`
`scipy`
`matplotlib`
`h5py`
`flask`
`networkx`
`scikit-learn` (imported as `sklearn`)
`python-louvain` (imported as `community`)
`fa2`

We recommend Anaconda (specifically Miniconda) to manage your Python libraries. You can download it here: https://conda.io/miniconda.html. 

Create a Python 3 conda environment for SPRING viewer to run in. For example, to create a conda environment called `spring_viewer` with Python 3.12, you can open Terminal (Mac) or Anaconda Prompt (Windows) and enter:

`conda create -n spring_viewer python=3.12`

Activate your new conda environment by running:

`conda activate spring_viewer`

Most of the libraries SPRING needs can then be installed by running:

`conda install numpy scipy matplotlib h5py flask networkx scikit-learn`

Also run:

`pip install annoy python-louvain`

The `fa2` library has to be installed from modified source code. To do so, you'll create a folder with the source code by running these commands:

```bash
git clone https://github.com/bhargavchippada/forceatlas2
cd forceatlas2
```

You then need to modify the `setup.py` file in that folder as described in [this pull request](https://github.com/bhargavchippada/forceatlas2/pull/46). To do so, copy the contents of `fa2_patched_setup.py` from this folder to `setup.py` in the `forceatlas2` folder. Do not change the name of `setup.py`. Once you've copied over the changes, run the following command while in the `forceatlas2` folder:

```bash
pip install . --user
```

## Setting up a SPRING data directory
See the example notebooks:  
[Hematopoietic progenitor FACS subpopulations](./data_prep/spring_example_HPCs.ipynb)  
[Mature blood cells (10X Genomics 4k PBMCs)](./data_prep/spring_example_pbmc4k.ipynb)  
[CITE-seq data from 10X Genomics](./data_prep/spring_notebook_10X_CITEseq.ipynb)  
[PBMCs from 10X Genomics](./data_prep/spring_notebook_10X.ipynb)

A SPRING data set consist of a main directory and any number of subdirectories, with each subdirectory corresponding to one SPRING plot (i.e. subplot) that draws on a data matrix stored in the main directory. The main directory should have the following files, as well as one subdirectory for each SPRING plot. 

`matrix.mtx`  
`counts_norm_sparse_cells.hdf5`  
`counts_norm_sparse_genes.hdf5`  
`genes.txt`  

Each subdirectory should contain:  

`categorical_coloring_data.json`  
`cell_filter.npy`  
`cell_filter.txt`  
`color_data_gene_sets.csv`  
`color_stats.json`  
`coordinates.txt`  
`edges.csv`  
`graph_data.json`  
`run_info.json`  

## Running SPRING Viewer

1. Open Terminal (Mac) or Anaconda Prompt (Windows) and change directories (`cd`) to the directory containing this README file (`SPRING_dev/`). 
2. Activate your conda environment for running SPRING; you can list all your conda environments by running `conda env list` if you need a reminder of what it's called. As an example, if your environment is called `spring_viewer`, you can activate it by entering the following: `conda activate spring_viewer`
3. Start a local server by entering the following: `./start_server.sh`
4. Open web browser (preferably Chrome; best to use incognito mode to ensure no cached data is used).
5. View data set by navigating to corresponding URL: http://localhost:8000/springViewer_1_6_dev.html?path_to/main/subplot. In the example above, if you wanted to view a SPRING plot called `FullDataset_v1` in the main directory `10X_PBMCs_Signac_GitHub`, then you would navigate to http://localhost:8000/springViewer_1_6_dev.html?datasets/10X_PBMCs_Signac_GitHub/FullDataset_v1

## Signac

To classify cellular phenotypes in single cell data, [SignacX](https://cran.r-project.org/web/packages/SignacX/) was integrated with the files output by SPRING (specifically, the matrix.mtx, genes.txt, edges.csv and categorical_coloring_data.json files), such that SPRING data can be classified by Signac in R with only a few lines of code. First, install SignacX in R:

```r
# load the Signac library
install.packages('SignacX')
library(SignacX)
```

Now classify the cellular phenotypes in R, which allows them to be visualized in SPRING Viewer:

```r
# dir is the subdirectory generated by the Jupyter notebook; it is the directory that contains the 'categorical_coloring_data.json' file.
dir = "./FullDataset_v1" 

# load the expression data
E = CID.LoadData(dir)

# generate cellular phenotype labels
labels = Signac(E, spring.dir = dir, num.cores = 4)
# alternatively, if you're in a hurry, use:
# labels = SignacFast(E, spring.dir = dir, num.cores = 4)
celltypes = GenerateLabels(labels, E = E, spring.dir = dir)

# write cell types and Louvain clusters to SPRING
dat <- CID.writeJSON(celltypes, spring.dir = dir)
```

Subsequently, the single cell data together with the cellular phenotypes (and Louvain clusters) can be visualized in SPRING Viewer (Signac writes the cell type and clustering information to the categorical_coloring_data.json file).
