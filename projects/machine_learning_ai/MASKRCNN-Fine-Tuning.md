# Finetuning Segment Anything Model

## Project description
The intention of this project to provide a end-to-end training and inference script for MaskRCNN Model. Contributers to this project is welcomed. More Details on the Project Page.

[Project Page](https://github.com/aninda-ghosh/maskrcnn-rpn) | [Original Paper](https://openaccess.thecvf.com/content_ICCV_2017/papers/He_Mask_R-CNN_ICCV_2017_paper.pdf)

## Folder Structure 
```
├──  dataset
│    └── images  - Store the Images here
│    └── masks - Store the Masks here
│    └── data.txt - It stores the relative path of the images & masks 
│
├── train.py
├── train.ipynb
├── test.ipynb - Custom Testings
```


##  Backbone

The current backbone used for the initial feature extraction is ResNet50. This backbone is small enough to be trained end to end, at the same time also, large enough to find suitable features in 2D signals (especially Images).

## Evaluation

The evaluation happens in the COCO format, and the loss functions being used are `Log Loss` in the classification stage of the RPN along with a `Regression Loss function (Smoothed L1)`. 
