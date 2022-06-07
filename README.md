## Introduction
In this notebook you will download and preprocess the data for the segmentation task for cardiac mri images:<br />
The data is provided by the medical segmentation decathlon (http://medicaldecathlon.com/)<br /> (Data License: https://creativecommons.org/licenses/by-sa/4.0/)<br />

For inspecting the data we use the sagittal view for this task as it provides the "nicest" images<br />
We import HTML from IPython.display to create a video of the volume. Example: <br/>

![alt text]([https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/Images_1.mp4?raw=true](https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/Images_1.mp4?raw=true))

![alt text](https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/Images_1.mp4?raw=true)

## Preprocessing
We perform the following preprocessing steps:

1. Normalization per subject. We compute mean and sigma of the full 3d volume $X$ and then perform z-normalization:
$$X_n = \frac{X-\mu}{\sigma}$$
2. There is a plenty of empty space around the image, so we can crop the image (e.g 32 px from top and bottom). Additionally, we can crop away non-cardiac regions as they will definitely not contain the left atrium. This will  reduce training time due to the smaller size and will simplify the segmentation. It is important to crop first and to normalize afterwards. Otherwise the included zero values introduce skewness to the intesity distribution.
3. Standardize the normalized subject into the 0-1 range:
$$X_s = \frac{X_n - min(X_n)}{max(X_n)-min(X_n)} $$
4. This task will be performed on a slice level (2D) and not on a subject level (3D). In order to reduce the computational cost we store the preprocessed data as npy files of 2d slices. Reading a single slice is much faster than loading the complete NIfTI file every time.

Data after preprocessing: <br/> ![alt text](https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/images_2.png?raw=true)

## DataSet Creation
We need to implement the following functionality:
1. Create a list of all 2D slices. To do so we need to extract all slices from all subjects
2. Extract the corresponding label path for each slice path
3. Load slice and label
4. Data Augmentation. Make sure that slice and mask are augmented identically. imgaug handles this for us, thus we will not use torchvision.transforms for that
5. Return slice and mask

Visualize the dataset with sample augmentations: <br/>
![alt text](https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/images_3.png?raw=true)

## Model
then, we will create the model for the atrium segmentation! <br />
We will use the most famous architecture for this task, the U-NET (https://arxiv.org/abs/1505.04597). <br/>

The idea behind a UNET is the Encoder-Decoder architecture with additional skip-connctions on different levels:
The encoder reduces the size of the feature maps by using downconvolutional layers.
The decoder reconstructs a mask of the input shape over several layers by upsampling.
Additionally skip-connections allow a direct information flow from the encoder to the decoder on all intermediate levels of the UNET.
This allows for a high quality of the produced mask and simplifies the training process.<br />
![alt text](https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/unet.png?raw=true)

## Training
We will implement full segmentaion model with pytorch-lightning.
Computed Dice-score: 0.93

## Visualization
We will load a test subject from the dataset and estimate the position of the left atrium.
After preprocessing the scan and croping 32 px from top, bottom, back and front, the results will be like this: <br/>
![alt text](https://github.com/fshnkarimi/Atrium-Segmentation/blob/main/Images/Images_4.mp4?raw=true)

