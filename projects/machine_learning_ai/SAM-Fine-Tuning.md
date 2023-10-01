# Finetuning Segment Anything Model

## Project description
The intention of this project to provide a end-to-end training and inference script for Segment Anything Model. Contributers to this project is welcomed. More Details on the Project Page.

[Project Page](https://github.com/aninda-ghosh/SAM-FineTuning) | [Original Paper](https://ai.meta.com/research/publications/segment-anything/) | [1 Billion Dataset](https://ai.meta.com/datasets/segment-anything-downloads/)

## Folder Structure 
```
├── config
│   └── defaults.py         - here's the default config file. 
├── data  
│   └── datasets            - here's the datasets folder that is responsible for all data handling.
│   └── transforms          - here's the data preprocess folder that is responsible for all data augmentation.
│   └── build.py  	        - here's the file to make dataloader.
│   └── collate_batch.py    - here's the file that is responsible for merges a list of samples to form a mini-batch.
├── engine
│   ├── trainer.py          - this file contains the train loops.
│   └── inference.py        - this file contains the inference process.
├── modeling                - this folder contains segment anything model definitions.
│   └── segment_anything
├── solver                  - this folder contains optimizer of the project.
│   └── build.py
│   └── lr_scheduler.py
├── tools                   - here's the train/test model of the project.
│   └── train_net.py        - here's an example of train model that is responsible for the whole pipeline.
└── utils
    └── logger.py
```


## Loss Functions

```python
class FocalLoss(nn.Module):

    def __init__(self, weight=None, size_average=True):
        super().__init__()

    def forward(self, inputs: torch.Tensor, targets: torch.Tensor, alpha=ALPHA, gamma=GAMMA):
        smooth=1e-6   # Used to prevent division by zero.

        targets = targets.view(-1)
        focal_loss = []
        
        for input in inputs:
            #flatten label and prediction tensors
            input = input.view(-1)
            BCE = F.binary_cross_entropy(input, targets, reduction='mean')
            BCE_EXP = torch.exp(-BCE)
            focal_loss.append(alpha * (1 - BCE_EXP)**gamma * BCE)

        return focal_loss


class DiceLoss(nn.Module):

    def __init__(self, weight=None, size_average=True):
        super().__init__()

    def forward(self, inputs: torch.Tensor, targets: torch.Tensor):
        smooth=1e-6   # Used to prevent division by zero.

        targets = targets.view(-1)
        dice_loss = []
        for input in inputs:
            #flatten label and prediction tensors
            input = input.view(-1)
        
            intersection = (input * targets).sum()
            dice_score = (2. * intersection + smooth) / (input.sum() + targets.sum() + smooth)

            dice_loss.append(1 - dice_score)

        return dice_loss
    

class MSELoss(nn.Module):

    def __init__(self, weight=None, size_average=True):
        super().__init__()

    def forward(self, inputs: torch.Tensor, targets: torch.Tensor):
        targets = targets.view(-1)
        mse_loss = []
        for input in inputs:
            input = input.view(-1)
            MSE_LOSS = nn.MSELoss()
            mse_loss.append(MSE_LOSS(input, targets))
        
        return mse_loss


class IoULoss(nn.Module):

    def __init__(self, weight=None, size_average=True):
        super().__init__()

    def forward(self, inputs: torch.Tensor, targets: torch.Tensor, pred_iou: torch.Tensor):
        smooth=1e-6   # Used to prevent division by zero.

        #flatten label and prediction tensors
        inputs = inputs.view(-1)
        targets = targets.view(-1)
        
        intersection = (inputs * targets).sum()
        union = inputs.sum() + targets.sum() - intersection
        iou = (intersection + smooth) / (union + smooth)
        
        loss = iou - pred_iou

        return loss

```

