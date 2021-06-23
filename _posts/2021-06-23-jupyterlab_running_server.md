---
title: "Jupyter lab & Swin OD model Inference"
excerpt: "노트북은 불편하니 jupyter lab을 써보고, Swin Inference model serving도 해보자!"

# permalink: /categories/kubernetes/4/

categories:
  - develop
  - deep learning
  - framework
tags:
  - python
  - jupyterlab
  - GCP
last_modified_at: 2021-06-23T00:13:00-05:00

comments: true
---

# jupyterlab

- 놀랄 만큼 사용이 간단하다

```bash
# venv activate
source venv/bin/activate

# install jupyter lab
pip install jupyter lab

# use config file - run jupyter lab
jupyter lab config=jupyter_notebook_config.py
```
- 끝!


# Model Serving

## Swin Transformer - Object Detection
- https://github.com/SwinTransformer/Swin-Transformer-Object-Detection
  - 위 코드를 활용하여 inference 하고싶음!

- https://github.com/open-mmlab/mmdetection/blob/master/docs/get_started.md
  - 일단 여기서 하라는 설치 다 해주고

``` bash
# model file download
wget https://github.com/SwinTransformer/storage/releases/download/v1.0.3/moby_mask_rcnn_swin_tiny_patch4_window7_1x.pth

# coco annotations download
wget http://images.cocodataset.org/annotations/annotations_trainval2017.zip

# unzip
unzip annotations_trainval2017.zip

# and make data/coco/annotation dir in root of Swin-Transformer dir
mkdir data
mkdir data/coco
cd data/coco
cp -r Swin-Transformer-Object-Detection/annotations .

# test inference execution : 근데 아직 에러남
python tools/test.py configs/swin/mask_rcnn_swin_tiny_patch4_window7_mstrain_480-800_adamw_1x_coco.py mask_r_cnn_swinT_Lr1x/mask_rcnn_swin_tiny_patch4_window7_1x.pth --eval bbox segm
```
- 다 하고 위 inference command 수행했지만
  - KeyError: "MaskRCNN: 'SwinTransformer is not in the models registry'"
  - 뭔가 설치가 덜 되었나..
