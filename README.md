[![DOI](https://zenodo.org/badge/261506912.svg)](https://zenodo.org/badge/latestdoi/261506912)

# GEBI: Global Explanations for Bias Identification

Code and data for the paper [Towards Explainable Classifiers Using the Counterfactual Approach – Global Explanations for Discovering Bias in Data](https://doi.org/10.2478/jaiscr-2021-0004) (Journal of Artificial Intelligence and Soft Computing Research, 2021; [arXiv](https://arxiv.org/abs/2005.02269)).

GEBI finds possible sources of bias in an image dataset. It groups images by both what they show and where the classifier looks (its attention maps), so a person can inspect each group and spot artifacts such as black frames or rulers. Each suspected artifact is then tested by inserting it into images and checking how many predictions change.

> **Note:** this is a project from 2020. In 2026 it got small updates only: a clearer README, a new figure, a `requirements.txt`, a license, and fixes so the notebooks run on current Python and library versions. The method, data and results are unchanged.

![](images/anim.gif)

## Table of Contents

- [Abstract](#abstract)
- [How GEBI works](#how-gebi-works)
- [Bias in the skin lesion dataset](#bias-in-the-skin-lesion-dataset)
- [Bias insertion](#bias-insertion)
- [Results](#results)
- [Code and data](#code-and-data)
- [Running the notebooks](#running-the-notebooks)
- [Citing](#citing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Abstract

Agnieszka Mikołajczyk, Michał Grochowski, Arkadiusz Kwasigroch

In the paper, we propose attention-based summarized post-hoc explanations for detection and identification of bias in data. We propose a global explanation and introduce a step-by-step framework on how to detect and test bias. Then, the bias is evaluated with proposed counterfactual approach to bias insertion. Because removing the unwanted bias is often a complicated and tremendous task, we automatically insert it, instead. We validate our results on the example of the skin lesion dataset. Using the method, we successfully identified and confirmed part of the possible bias-causing artifacts in dermoscopy images. We confirmed that the commonplace black frames in the training dataset images have a strong influence on the Convolutional Neural Network's prediction. After artificially adding a black frame to all images, around 22% of them changed the prediction from benign to malignant. We have shown that bias detection is an important step of making more robust models, and we discuss how to improve them.

## How GEBI works

The framework has three steps:

1. **Discover bias** with global explanations ([GEBI notebook](notebooks/GEBI.ipynb)).
2. **Insert the suspected bias** into all images in the dataset ([frame](notebooks/add_bias_to_images-frame.ipynb), [ruler](notebooks/add_bias_to_images-ruler.ipynb) and [red circle](notebooks/add_bias_to_images.ipynb) notebooks).
3. **Check how the predictions changed.**

Step 1 in detail:

* Step 0. Select samples for analysis.
* Step 1. Compute attention maps for the samples of one class.
* Step 2. Normalize and preprocess the input samples and their attention maps in the same way.
* Step 3. Reduce the dimension of each input sample and attention map with a dimensionality reduction algorithm, for example Isomap.
* Step 4. Join each reduced sample with its reduced attention map.
* Step 5. Run spectral clustering on the joined vectors.
* Step 6. Visualize and analyze the clusters.
* Step 7. Formulate a hypothesis and test it with the bias insertion algorithm.

![GEBI flowchart](images/flowchart-gebi.png)

GEBI builds on SpRay (spectral relevance analysis), which clusters attention maps only. The repository includes SpRay and a version with Isomap for comparison:

![What each notebook clusters on](images/gebi-vs-spray.svg)

## Bias in the skin lesion dataset

The method found four clusters. Each cluster shows its own characteristics related to skin tone and skin lesions, and also to unwanted artifacts. The first and second clusters seem to group images by how similar the skin lesions are, which is the result we want. The third cluster mostly gathers images with round or rectangular black frames. The fourth cluster contains mostly light skin lesions, very often with a visible ruler.

The method is semi-automated, so a field expert should analyze the clusters. We focused on clusters 3 and 4, where artifacts seemed to group the images. This led to the hypothesis that black frames and ruler marks might cause bias in models. To check whether these features have a significant influence on the prediction, we inserted them into images and tested their influence.

![Clusters](images/clusters.PNG)

## Bias insertion

To test how the prediction changes when a feature is present, we compare the model's outputs for the same image with and without that feature. Removing artifacts from images is very hard, so we insert them instead. We copied artifacts found in the dataset, and added one new artifact for comparison.

* **Black frames** were added in the same way to all images, with the same size and position. Frames are common in the dataset and are usually considered unwanted artifacts.
* **Ruler marks** were placed on the images in slightly different sizes, angles and positions. Doctors use rulers to show the size of a skin lesion on dermoscopic images.
* **Red circles** do not appear in the ISIC archive, SD-198 or Derm7pt datasets. We added them for comparison. A single red circle was placed at a random position, on the skin or on the lesion.

![Bias insertion examples](images/bias_insertion.PNG)

## Results

We added each artifact to all images and measured how the predictions changed:

![Idea behind counterfactual bias insertion](images/bias_insertion2.PNG)

Change in prediction, in percentage points:

| Added feature | Class | Average change | Maximum change |
|---|---|---:|---:|
| Ruler | malignant | 2.21 | 22.01 |
| | benign | 1.23 | 19.91 |
| **Frame** | **malignant** | **30.77** | **62.43** |
| | **benign** | **32.04** | **63.66** |
| Red circle | malignant | 2.27 | 15.51 |
| | benign | 1.50 | 12.78 |

Black frames have by far the strongest effect on the classifier's predictions.

## Code and data

### Notebooks

| Notebook | What it does |
|---|---|
| [GEBI.ipynb](notebooks/GEBI.ipynb) | The GEBI method on the skin lesion data: clusters the attention maps, the images, and both joined. Uses the precomputed attention maps in `data/`. |
| [spray.ipynb](notebooks/spray.ipynb) | The original SpRay method, for comparison. |
| [spray_isomap.ipynb](notebooks/spray_isomap.ipynb) | SpRay with an extra Isomap dimensionality reduction step. |
| [add_bias_to_images-frame.ipynb](notebooks/add_bias_to_images-frame.ipynb) | Adds a black frame to benign images. Needs a frame mask `frame.png` (not included). |
| [add_bias_to_images-ruler.ipynb](notebooks/add_bias_to_images-ruler.ipynb) | Adds a ruler with a random rotation and position. Needs a ruler mask `ruler.png` (not included). |
| [add_bias_to_images.ipynb](notebooks/add_bias_to_images.ipynb) | Adds a red circle at a random position. |
| [generate_3D_visualizations.ipynb](notebooks/generate_3D_visualizations.ipynb) | 3D plots of the clusters, with an image thumbnail at each point. |

The clustering notebooks copy each image into a folder per cluster under `experiments/`, so you can look through the clusters.

The bias insertion notebooks were written for the training images, which are not in this repository. Set `DATA_DIR` at the top of each notebook to a folder with `skin_lesions/train/ben/`.

### Data

`data/skin_lesions/` contains the skin lesion images used in the experiments, already preprocessed, with their attention maps:

| Folder | Contents | Files |
|---|---|---:|
| `org-mal/` | malignant images | 884 |
| `h-mal/` | attention maps for the malignant images | 884 |
| `org-ben/` | benign images | 4246 |
| `h-ben/` | attention maps for the benign images | 4245 |

Images and their attention maps share a number: `org-mal/im0_original.png` goes with `h-mal/im0_deeptaylor.png`. The attention maps were computed with Deep Taylor decomposition.

## Running the notebooks

```bash
git clone https://github.com/AgaMiko/GEBI.git
cd GEBI
pip install -r requirements.txt
cd notebooks
jupyter notebook
```

The repository is about 800 MB, because it includes the images.

The interactive image annotations in `generate_3D_visualizations.ipynb` use internal Matplotlib functions, so that part may need an older Matplotlib version.

## Citing

```bibtex
@article{mikolajczyk2021towards,
  title   = {Towards Explainable Classifiers Using the Counterfactual Approach -- Global Explanations for Discovering Bias in Data},
  author  = {Miko{\l}ajczyk, Agnieszka and Grochowski, Micha{\l} and Kwasigroch, Arkadiusz},
  journal = {Journal of Artificial Intelligence and Soft Computing Research},
  volume  = {11},
  number  = {1},
  pages   = {51--67},
  year    = {2021},
  doi     = {10.2478/jaiscr-2021-0004}
}
```

## License

The code is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Acknowledgments

The research reported in this publication was supported by Polish National Science Centre (Grant Preludium No: UMO-2019/35/N/ST6/04052). The authors wish to express their thanks for the support.
