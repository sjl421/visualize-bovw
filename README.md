visualize-BoVW
==============

These are the scripts I used for my master's thesis on *Visualizing Bag-of-Visual-Words for Visual Concept Detection*.
The code should only depend on [scikit-learn](https://github.com/scikit-learn/scikit-learn) (>= 0.14) and the
[Python Imaging Library](http://www.pythonware.com/products/pil/) (for the visualizations).

## Training classifiers and grid search

### DataManagers
Data managers provide an interface to work with different datasets. Their main purpose is to expose methods to
- get the input data matrix for a specific category
- get the vector of class labels for a specific category

for either a subset of the data or the whole dataset.

To implement a `DataManager` for a new dataset, you will have to inherit from `DataManager` and implement
three methods `_build_sample_matrix`, `_build_class_vector` and `_get_positive_samples`.
Each of these methods accepts two parameters, `dataset` and `category`. `dataset` can be either of the
strings "all", "train", or "test", specifying if the data of all images, or only the training or test set
should be returned. The `DataManager` takes care of loading all needed data points.
`category` is the name of the category, that will be trained.

Two datamanagers for the ImageCLEF 2011 and Caltech101 dataset are included in this repository.
Both expect the data to be organized in different folders. The absolute paths to these folders are
stored in the `DataManager.PATHS` dictionary and can be changed programmatically and separately. The base path
to all of these folders can also be changed simultaneously via `DataManager.change_base_path()`.
The most important locations include:

- KEYPOINTS: Directory containing the keypoint information for each image
- BOW: Directory containing the extracted Bag-of-Visual-Words-vectors (.descr_bowdescr.bin) + the mapping of keypoints
to visual words (.descr_indices.bin)
- IMG: Directory containing the actual images. All images should exist on the same level in this directory.
The `CaltechManager` expects all image filenames to be in the form *category*_*img_name*.
- CLASSIFIER: This is the directory into which classifiers will be serialized after the training step.
- RESULTS: The final visualizations will be placed into this directory.
- LOGS: If logging to file is enabled, the logfiles will be saved in this directory.

## HOWTO

### Prepare the data
For ImageCLEF it should be enough to extract the images into a directory, together with the metadata on
image categories (`concepts_2011.txt`, `trainset_gt_annotations_corrected.txt`, `testset_GT_annotations.txt`).
Generate the BoVW and don't forget to configure the paths to the data directories.
Either change the code of the `ClefManager` class, or do something like:

```python
datamanager = ClefManager()
datamanager.PATHS['RESULTS'] = '/path/to/visualization/results'
```
in your calling code.

To prepare the Caltech101 dataset, a few extra steps are necessary.
First of all, the data needs to be split into a training- and test-set.
To separate a fixed percentage of each category, you can use the `caltech_choose_testset.py` script.
Just set the desired `TEST_SET_RATE` and run it in your Caltech101 folder (`101_ObjectCategories`).
It will create a new subfolder `test` in each category folder and move randomly selected images into it.
Secondly, for the datamanagers to work, you will need a directory with all the images in one place.
As in the Caltech101 dataset, images are named similarly in different categories, you need to move and rename them.
The `CaltechManager` expects a folder with the original image names and a `test` folder in each category
(the result of running `caltech_choose_testset.py`) and a folder `images`, where each image is named
like this: `{category name}_{original image name}`

