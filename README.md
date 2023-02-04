# PopV

PopV uses popular vote of a variety of cell-type transfer tools to classify cell-types in a query dataset based on a test dataset.
Using this variety of algorithms, we compute the agreement between those algorithms and use this agreement to predict which cell-types are with a high likelihood the same cell-types observed in the reference.

## Algorithms

Currently implemented algorithms are:

-   K-nearest neighbor classification after dataset integration with [BBKNN](https://github.com/Teichlab/bbknn)
-   K-nearest neighbor classification after dataset integration with [SCANORAMA](https://github.com/brianhie/scanorama)
-   K-nearest neighbor classification after dataset integration with [scVI](https://github.com/scverse/scvi-tools)
-   Random forest classification
-   Support vector machine classification
-   [OnClass](https://github.com/wangshenguiuc/OnClass) cell type classification
-   [scANVI](https://github.com/scverse/scvi-tools) label transfer
-   [Celltypist](https://www.celltypist.org) cell type classification

All algorithms are implemented as a class in [popv/algorithms](popv/algorithms/__init__.py).
To implement a new method, a class has to have several methods:

-   algorithm.compute_integration: Computes dataset integration to yield an integrated latent space.
-   algorithm.predict: Computes cell-type labels based on the specific classifier.
-   algorithm.compute_embedding: Computes UMAP embedding of previously computed integrated latent space.

We highlight the implementation of a new classifier in a [scaffold](popv/algorithms/_scaffold.py). Adding a new class with those methods will automatically tell PopV to include this class into its classifiers and will use the new classifier as another expert.

All algorithms that allow for pre-training are pre-trained. This excludes by design BBKNN and SCANORAMA as both construct a new embedding space. Pretrained models are stored on Zenodo and are automatically downloaded in the Colab notebook linked below. We encourage pre-training models when implementing new classes.

All input parameters are defined during initial call to [Process_Query](popv/preprocessing.py) and are stored in the unstructured field of the generated AnnData object. PopV has three prediction complexities:

-   retrain will train all classifiers from scratch. For 50k cells this takes up to an hour of computing time using a GPU.
-   inference will use pretrained classifiers to annotate query as well as reference cells and construct a joint embedding using all integration methods from above. For 50k cells this takes in our hands up to half an hour of computing time using a GPU.
-   fast will use only methods with pretrained classifiers to annotate only query cells. This will take 2 minutes without a GPU.

A user-defined selection of classification algorithms can be defined when calling [annotate_data](popv/annotation.py). Additionally advanced users can define her non-standard parameters for the integration methods as well as the classifiers.

## Output

PopV will output a cell-type classification for each of the used classifiers, as well as the majority vote across all classifiers. Additionally, PopV uses the ontology to go through the full ontology descendants for the OnClass prediction (disabled in fast mode). This method will be further described when PopV is published. PopV additionally outputs a score, which counts the number of experts that agreed upon the PopV prediction. This columns can be seen as the certainty that the current prediction is correct. We generally found disagreement of a single expert to be still high quality prediction while disagreement of more than 2 experts signifies unreliable results and should lead to further manual inspection of the respective cluster. The aim of PopV is not to fully annotate a data set but to highlight cells that potentially benefit from further manual careful annotation.
Additionally, PopV outputs UMAP embeddings of all integrated latent spaces if _compute_embedding==True_ in [Process_Query](popv/preprocessing.py) and computes certainties for every used classifier if _return_probabilities==True_ [Process_Query](popv/preprocessing.py).

## Installation

We suggest using a package manager like conda or mamba to install the package. Due to various requirements of the different algorithms underlying PopV are strict, we recommend installing PopV in a new environment, while we found installing it in existing environments to work. OnClass files for annotation based on Tabula sapiens are deposited in popv/ontology. We use [Cell Ontology](https://obofoundry.org/ontology/cl.html) as an ontology throughout our experiments. PopV will automatically look for the ontology in this folder. If you want to provide your user-edited ontology, we will provide notebooks to create the Natural Language Model used in OnClass for this user-defined ontology.

    conda create -n yourenv python=3.8
    conda activate yourenv
    pip install git+https://github.com/czbiohub/PopV

## Example notebook

We deposited an example notebook in Google Colab:

-   [Tutorial](https://colab.research.google.com/drive/1mVf4Ksb9WQJ77wEFFduNTHLjON4FlsGc#scrollTo=ZnoRUg58Aq4-)

This notebook will guide you through annotating a dataset based on the annotated [Tabula sapiens reference](https://tabula-sapiens-portal.ds.czbiohub.org) and demonstrates how to run annotation on your own query dataset. This notebook requires that all cells are annotated based on a cell ontology. We strongly encourage the use of a common cell ontology, see also [David Osumi-Sutherland et al](https://www.nature.com/articles/s41556-021-00787-7). Using a cell ontology is a requirement to run OnClass as a prediction algorithm. We highlight the usefulness as OnClass is able to predict cells, that were not part of the reference and this can give a hint on missing granularity of cell-type resolution.

However, for other organisms than human no ontology exists. We therefore allow running PopV without using a cell ontology. A second notebook highlighting using PopV without an existing ontology is currently planned and will be released here.

Memory requirements exceed the free limit in Colab and we recommend a Pro access to run the notebook.
