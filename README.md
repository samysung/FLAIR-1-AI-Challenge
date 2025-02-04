# FLAIR #1 : Semantic segmentation and domain adaptation of land-cover
### Challenge proposed by the French National Institute of Geographical and Forest Information (IGN).

Code : [![license](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/IGNF/FLAIR-1-AI-Challenge/blob/master/LICENSE) &emsp; Dataset : [![license](https://img.shields.io/badge/License-IO%202.0-green.svg)](https://github.com/etalab/licence-ouverte/blob/master/open-licence.md)


Participate in obtaining more accurate maps for a more comprehensive description and a better understanding of our environment! Come push the limits of state-of-the-art semantic segmentation approaches on a large and challenging dataset. Get in touch at ai-challenge@ign.fr



![Alt bandeau FLAIR-IGN](images/flair_bandeau.jpg?raw=true)



<div style="border-width:1px; border-style:solid; border-color:#d2db8c; padding-left: 1em; padding-right: 1em; ">
  
<h2 style="margin-top:5px;">Links</h2>


- **Datapaper :** https://arxiv.org/pdf/2211.12979.pdf

- **Dataset links :** https://ignf.github.io/FLAIR/ or https://huggingface.co/datasets/IGNF/FLAIR

- **Challenge page :**  https://codalab.lisn.upsaclay.fr/competitions/8769 [🛑 closed!]

</div>


## Context & Data

The FLAIR #1 dataset is sampled countrywide and is composed of over 20 billion annotated pixels, acquired over three years and different months (spatio-temporal domains). The dataset is available to download <a href="https://ignf.github.io/FLAIR/">here.</a> It consists of 512 x 512 patches with 13 (baselines) or 19 (full) semantic classes (see associated datapaper). Each patch has 5 channels (RVB-Infrared-Elevation). 

<br>

<figure>
  <img
  src="images/flair-1_spatiotemporal.png"
  alt="ortho image and train/test geographical repartition">
  <figcaption>ORTHO HR® aerial image cover of France (left), train and test spatial domains of the dataset (middle) and acquisition months defining temporal domains (right).</figcaption>
</figure>


<br>
<br>

<p align="center">
  <img width="70%" src="images/flair-1_patches.png">
  <br>
  <em>Example of input data (first three columns) and corresponding supervision masks (last column).</em>
</p>

```
flair_data = {
1   : ['building','#db0e9a'] ,
2   : ['pervious surface','#938e7b'],
3   : ['impervious surface','#f80c00'],
4   : ['bare soil','#a97101'],
5   : ['water','#1553ae'],
6   : ['coniferous','#194a26'],
7   : ['deciduous','#46e483'],
8   : ['brushwood','#f3a60d'],
9   : ['vineyard','#660082'],
10  : ['herbaceous vegetation','#55ff00'],
11  : ['agricultural land','#fff30d'],
12  : ['plowed land','#e4df7c'],
13  : ['swimming_pool','#3de6eb'],
14  : ['snow','#ffffff'],
15  : ['clear cut','#8ab3a0'],
16  : ['mixed','#6b714f'],
17  : ['ligneous','#c5dc42'],
18  : ['greenhouse','#9999ff'],
19  : ['other','#000000'],
}
```


<br>


## Baseline model 

A U-Net architecture with a pre-trained ResNet34 encoder from the pytorch segmentation models library is used for the baselines. The used architecture allows integration of patch-wise metadata information and employs commonly used image data augmentation techniques. It has about 24.4M parameters and it is implemented using the _segmentation-models-pytorch_ library. The results are evaluated with an Intersection Over Union (IoU) metric and a single mIoU is reported (see associated datapaper).

The _metadata_ strategy refers encoding metadata with a shallow MLP and concatenate this encoded information to the U-Net encoder output. The _augmentation_ strategy employs three typical geometrical augmentations (see associated datapaper).


Example of a semantic segmentation of an urban and coastal area in the D076 spatial
domain, obtained with the baseline trained model:


<p align="center">
  <img width="100%" src="images/flair-1_predicted.png">
  <br>
  <em>Example of a semantic segmentation result using the baseline model.</em>
</p>


<br>

## Usage 

The file `requirement.txt` is listing used libraries for the baselines.

### Configuration 

The pipeline is configured using a YAML file (`flair-1-config.yml`). The configuration file includes sections for data paths, tasks, model configuration, hyperparameters and computational resources.

`out_folder`: The path to the output folder where the results will be saved.<br>
`out_model_name`: The name of the output model.<br>
`train_csv`: Path to the CSV file containing paths to image-mask pairs for training.<br>
`val_csv`: Path to the CSV file containing paths to image-mask pairs for validation.<br>
`test_csv`: Path to the CSV file containing paths to image-mask pairs for testing.<br>
`ckpt_model_path`: The path to the checkpoint file of the model for prediction if train is disabled.<br>

`train`: If set to True, the model will be trained.<br>
`train_load_ckpt`: Initialize model with given weights.<br>
`predict`: If set to True, predictions will be made using the model.<br>
`metrics`: If set to True, metrics will be calculated.<br><br>
`delete_preds`: Remove prediction files after metrics calculation.<br><br>

`model_architecture`: The architecture of the model to be used (e.g., ‘unet’).<br>
`encoder_name`: The name of the encoder to be used in the model (e.g., ‘resnet34’).<br>
`use_augmentation`: If set to True, data augmentation will be applied during training.<br>

`use_metadata`: If set to True, metadata will be used. If other than the FLAIR dataset, see structure to be provided.<br>
`path_metadata_aerial`: The path to the aerial metadata JSON file.<br><br>

`channels`: The channels opened in your input images. Images are opened with rasterio which starts at 1 for the first channel.<br>
`seed`: The seed for random number generation to ensure reproducibility.<br><br>

`batch_size`: The batch size for training.<br>
`learning_rate`: The learning rate for training.<br>
`num_epochs`: The number of epochs for training.<br><br>

`use_weights`: If set to True, class weights will be used during training.<br>
`classes`: Dict of semantic classes with value in images as key and list [weight, classname] as value. See config file for an example.<br>

`norm_type`: Normalization to be applied: scaling (linear interpolation in the range [0,1]), custom (center-reduced with provided means and standard deviantions), without.<br><br>
`norm_means`: If custom, means for each input band.<br><br>
`norm_stds`: If custom standard deviation for each input band.<br><br>

`georeferencing_output`: If set to True, the output will be georeferenced.<br><br>

`accelerator`: The type of accelerator to use (‘gpu’ or ‘cpu’).<br>
`num_nodes`: The number of nodes to use for training.<br>
`gpus_per_node`: The number of GPUs to use per node for training.<br>
`strategy`: The strategy to use for distributed training (‘auto’,‘ddp’,...).<br>
`num_workers`: The number of workers to use for data loading.<br><br>

`cp_csv_and_conf_to_output`: Makes a copy of paths csv and config file to the output directory.<br>
`enable_progress_bar`: If set to True, a progress bar will be displayed during training and inference.<br>
`progress_rate`: The rate at which progress will be displayed.<br>

### Input CSV files

The input CSV files for training, validation, and testing are provided in a folder with the official split. Each CSV file should contain the paths to the image-mask pairs for the corresponding dataset.

### Usage

To use the pipeline, run the main script with the configuration file as an argument:


```
python main.py --config_file=./flair-1-config.yml
```

The script will perform the tasks specified in the configuration file. If ‘train’ is enabled, it will train the model and save the trained model to the output folder. If ‘predict’ is enabled, it will load the trained model (or a specified checkpoint if ‘train’ is not enabled) and perform prediction on the test data. If ‘metrics’ is enabled, it will calculate the mean Intersection over Union (mIoU) and other IoU metrics for the predicted and ground truth masks.

A toy dataset (reduced size) is available to check that your installation and the information in the configuration file are correct.

<em>Note: </em> A notebook is available in the legacy-torch branch (which uses different libraries versions and structure) that was used during the challenge. 

<br>

## Leaderboard

| Model | mIoU 
------------ | ------------- 
| baseline U-Net (ResNet34) | 0.5443±0.0014
| baseline U-Net (ResNet34) + _metadata + augmentation_ | 0.5570±0.0027

If you want to submit a new entry, you can open a new issue.
<b> Results of the challenge will be reported soon! </b>

The baseline U-Net with ResNet34 backbone obtains the following confusion matrix: 


<p align="center">
  <img width="50%" src="images/flair-1_heatmap.png">
  <br>
  <em>Baseline confusion matrix of the test dataset normalized by rows.</em>
</p>


## Reference
Please include a citation to the following article if you use the FLAIR #1 dataset:

```
@article{garioud2022flair1,
  doi = {10.13140/RG.2.2.30183.73128/1},
  url = {https://arxiv.org/pdf/2211.12979.pdf},
  author = {Garioud, Anatol and Peillet, Stéphane and Bookjans, Eva and Giordano, Sébastien and Wattrelos, Boris},
  title = {FLAIR #1: semantic segmentation and domain adaptation dataset},
  publisher = {arXiv},
  year = {2022}
}
```

## Acknowledgment
This work was performed using HPC/AI resources from
GENCI-IDRIS (Grant 2022-A0131013803).

## Dataset license

The "OPEN LICENCE 2.0/LICENCE OUVERTE" is a license created by the French government specifically for the purpose of facilitating the dissemination of open data by public administration. 
If you are looking for an English version of this license, you can find it on the official GitHub page at the [official github page](https://github.com/etalab/licence-ouverte).

As stated by the license :

### Applicable legislation

This licence is governed by French law.

### Compatibility of this licence

This licence has been designed to be compatible with any free licence that at least requires an acknowledgement of authorship, and specifically with the previous version of this licence as well as with the following licences: United Kingdom’s “Open Government Licence” (OGL), Creative Commons’ “Creative Commons Attribution” (CC-BY) and Open Knowledge Foundation’s “Open Data Commons Attribution” (ODC-BY).
