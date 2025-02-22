# Model Overview
A pre-trained model for the endoscopic inbody classification task and trained using the SEResNet50 structure, whose details can be found in [1]. All datasets are from private samples of [Activ Surgical](https://www.activsurgical.com/). Samples in training and validation dataset are from the same 4 videos, while test samples are from different two videos.

The [PyTorch model](https://drive.google.com/file/d/14CS-s1uv2q6WedYQGeFbZeEWIkoyNa-x/view?usp=sharing) and [torchscript model](https://drive.google.com/file/d/1fOoJ4n5DWKHrt9QXTZ2sXwr9C-YvVGCM/view?usp=sharing) are shared in google drive. Modify the `bundle_root` parameter specified in `configs/train.json` and `configs/inference.json` to reflect where models are downloaded. Expected directory path to place downloaded models is `models/` under `bundle_root`.

![image](https://developer.download.nvidia.com/assets/Clara/Images/monai_endoscopic_inbody_classification_workflow.png)

## Data
The datasets used in this work were provided by [Activ Surgical](https://www.activsurgical.com/).

We've provided a [link](https://github.com/Project-MONAI/MONAI-extra-test-data/releases/download/0.8.1/inbody_outbody_samples.zip) of 20 samples (10 in-body and 10 out-body) to show what this dataset looks like.

### Preprocessing
After downloading this dataset, python script in `scripts` folder naming `data_process` can be used to get label json files by running the command below and replacing datapath and outpath parameters.

```
python scripts/data_process.py --datapath /path/to/data/root --outpath /path/to/label/folder
```

After generating label files, please modify the `dataset_dir` parameter specified in `configs/train.json` and `configs/inference.json` to reflect where label files are.

The input label json should be a list made up by dicts which includes `image` and `label` keys. An example format is shown below.

```
[
    {
        "image":"/path/to/image/image_name0.jpg",
        "label": 0
    },
    {
        "image":"/path/to/image/image_name1.jpg",
        "label": 0
    },
    {
        "image":"/path/to/image/image_name2.jpg",
        "label": 1
    },
    ....
    {
        "image":"/path/to/image/image_namek.jpg",
        "label": 0
    },
]
```

## Training configuration
The training as performed with the following:
- GPU: At least 12GB of GPU memory
- Actual Model Input: 256 x 256 x 3
- Optimizer: Adam
- Learning Rate: 1e-3

### Input
A three channel video frame

### Output
Two Channels
- Label 0: in body
- Label 1: out body

## Performance
Accuracy was used for evaluating the performance of the model. This model achieves an accuracy score of 0.98

#### Training Loss
![A graph showing the training loss over 25 epochs.](https://developer.download.nvidia.com/assets/Clara/Images/monai_endoscopic_inbody_classification_train_loss.png)

#### Validation Accuracy
![A graph showing the validation accuracy over 25 epochs.](https://developer.download.nvidia.com/assets/Clara/Images/monai_endoscopic_inbody_classification_val_accuracy.png)

## MONAI Bundle Commands
In addition to the Pythonic APIs, a few command line interfaces (CLI) are provided to interact with the bundle. The CLI supports flexible use cases, such as overriding configs at runtime and predefining arguments in a file.

For more details usage instructions, visit the [MONAI Bundle Configuration Page](https://docs.monai.io/en/latest/config_syntax.html).

#### Execute training:

```
python -m monai.bundle run training \
    --meta_file configs/metadata.json \
    --config_file configs/train.json \
    --logging_file configs/logging.conf
```

#### Override the `train` config to execute multi-GPU training:

```
torchrun --standalone --nnodes=1 --nproc_per_node=2 -m monai.bundle run training \
    --meta_file configs/metadata.json \
    --config_file "['configs/train.json','configs/multi_gpu_train.json']" \
    --logging_file configs/logging.conf
```

Please note that the distributed training-related options depend on the actual running environment; thus, users may need to remove `--standalone`, modify `--nnodes`, or do some other necessary changes according to the machine used. For more details, please refer to [pytorch's official tutorial](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html).

#### Override the `train` config to execute evaluation with the trained model:

```
python -m monai.bundle run evaluating \
    --meta_file configs/metadata.json \
    --config_file "['configs/train.json','configs/evaluate.json']" \
    --logging_file configs/logging.conf
```

#### Execute inference:

```
python -m monai.bundle run evaluating \
    --meta_file configs/metadata.json \
    --config_file configs/inference.json \
    --logging_file configs/logging.conf
```

#### Export checkpoint to TorchScript file:

```
python -m monai.bundle ckpt_export network_def \
    --filepath models/model.ts \
    --ckpt_file models/model.pt \
    --meta_file configs/metadata.json \
    --config_file configs/inference.json
```

# References
[1] J. Hu, L. Shen and G. Sun, Squeeze-and-Excitation Networks, 2018 IEEE/CVF Conference on Computer Vision and Pattern Recognition, 2018, pp. 7132-7141. https://arxiv.org/pdf/1709.01507.pdf

# License
Copyright (c) MONAI Consortium

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
