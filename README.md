# TF Keras YOLOv3 Modelset

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE)

## Introduction

tf.keras implementation of a common YOLOv3 object detection architecture with different technologies support:

#### Backbone
- [x] Darknet53/Tiny Darknet
- [x] MobilenetV1
- [x] MobilenetV2
- [x] VGG16
- [x] Xception

#### Head
- [x] YOLOv3
- [x] YOLOv3 Lite
- [x] YOLOv3 spp
- [x] YOLOv3 Lite spp
- [x] Tiny YOLOv3
- [x] Tiny YOLOv3 Lite

#### Loss
- [x] Standard YOLOv3 loss
- [x] Binary focal classification loss
- [x] Softmax focal classification loss
- [x] GIoU localization loss
- [x] Binary focal loss for objectness (experimental)

#### Postprocess
- [x] Numpy YOLOv3 postprocess implementation
- [x] TFLite/MNN C++ YOLOv3 postprocess implementation
- [x] TF YOLOv3 postprocess model
- [x] tf.keras batch-wise YOLOv3 postprocess Lambda layer

#### Train tech
- [x] Transfer training from imagenet
- [x] Singlescale image input training
- [x] Multiscale image input training

#### On-device deployment
- [x] Tensorflow-Lite Float32/UInt8 model inference
- [x] MNN Float32/UInt8 model inference


## Quick Start

1. Install the requirements:

```
# pip install -r requirements.txt
```

2. Download Darknet/YOLOv3/YOLOv3-spp/Tiny YOLOv3 weights from [YOLO website](http://pjreddie.com/darknet/yolo/).
3. Convert the Darknet YOLO model to a Keras model.
4. Run YOLO detection on your image or video, default using Tiny YOLOv3 model.

```
# wget -O weights/darknet53.conv.74.weights https://pjreddie.com/media/files/darknet53.conv.74
# wget -O weights/yolov3.weights https://pjreddie.com/media/files/yolov3.weights
# wget -O weights/yolov3-tiny.weights https://pjreddie.com/media/files/yolov3-tiny.weights
# wget -O weights/yolov3-spp.weights https://pjreddie.com/media/files/yolov3-spp.weights

# python tools/convert.py cfg/yolov3.cfg weights/yolov3.weights weights/yolov3.h5
# python tools/convert.py cfg/yolov3-tiny.cfg weights/yolov3-tiny.weights weights/yolov3-tiny.h5
# python tools/convert.py cfg/yolov3-spp.cfg weights/yolov3-spp.weights weights/yolov3-spp.h5
# python tools/convert.py cfg/darknet53.cfg weights/darknet53.conv.74.weights weights/darknet53.h5

# python yolo.py --image
# python yolo.py --input=<your video file>
```
For other model, just do in a similar way, but specify different model path and anchor path with `--model_path` and `--anchors_path`.


## Guide of train/evaluate/demo

### Train
1. Generate train/val/test annotation file and class names file.
    * One row for one image in annotation file;
    * Row format: `image_file_path box1 box2 ... boxN`;
    * Box format: `x_min,y_min,x_max,y_max,class_id` (no space).
    * Here is an example:
    ```
    path/to/img1.jpg 50,100,150,200,0 30,50,200,120,3
    path/to/img2.jpg 120,300,250,600,2
    ...
    ```
    1. For VOC style dataset, you can use [voc_annotation.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/tools/voc_annotation.py) to convert original dataset to our annotation file:
       ```
       # cd tools && python voc_annotation.py -h
       usage: voc_annotation.py [-h] [--dataset_path DATASET_PATH]
                                [--output_path OUTPUT_PATH]
                                [--classes_path CLASSES_PATH] [--include_difficult]

       optional arguments:
         -h, --help            show this help message and exit
         --dataset_path DATASET_PATH
                               path to PascalVOC dataset, default is ../VOCdevkit
         --output_path OUTPUT_PATH
                               output path for generated annotation txt files,
                               default is ./
         --classes_path CLASSES_PATH
                               path to class definitions
         --include_difficult   to include difficult object
       ```
       By default, the VOC convert script will try to go through both VOC2007/VOC2012 dataset dir under the dataset_path and generate train/val/test annotation file separately, like:
       ```
       2007_test.txt  2007_train.txt  2007_val.txt  2012_train.txt  2012_val.txt
       ```
       You can merge these train & val annotation file as your need. For example, following cmd will creat 07/12 combined trainval dataset:
       ```
       # cp 2007_train.txt trainval.txt
       # cat 2007_val.txt >> trainval.txt
       # cat 2012_train.txt >> trainval.txt
       # cat 2012_val.txt >> trainval.txt
       ```

    2. For COCO style dataset, you can use [coco_annotation.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/tools/coco_annotation.py) to convert original dataset to our annotation file:
       ```
       # cd tools && python coco_annotation.py -h
       usage: coco_annotation.py [-h] [--dataset_path DATASET_PATH]
                                 [--output_path OUTPUT_PATH]

       optional arguments:
         -h, --help            show this help message and exit
         --dataset_path DATASET_PATH
                               path to MSCOCO dataset, default is ../mscoco2017
         --output_path OUTPUT_PATH
                               output path for generated annotation txt files,
                               default is ./
       ```
       This script will try to convert COCO instances_train2017 and instances_val2017 under dataset_path. You can change the code for your dataset

   If you want to download PascalVOC or COCO dataset, refer to [Dockerfile](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/Dockerfile) for cmd

   For class names file format, refer to  [coco_classes.txt](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/configs/coco_classes.txt)

2. If you're training Darknet YOLOv3/Tiny YOLOv3, make sure you have converted pretrain model weights as in "Quick Start" part

3. [train.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/train.py)
```
# python train.py -h
usage: train.py [-h] [--model_type MODEL_TYPE] [--tiny_version]
                [--model_image_size MODEL_IMAGE_SIZE]
                [--weights_path WEIGHTS_PATH] [--freeze_level FREEZE_LEVEL]
                [--annotation_file ANNOTATION_FILE]
                [--val_annotation_file VAL_ANNOTATION_FILE]
                [--val_split VAL_SPLIT] [--classes_path CLASSES_PATH]
                [--learning_rate LEARNING_RATE] [--batch_size BATCH_SIZE]
                [--init_epoch INIT_EPOCH] [--total_epoch TOTAL_EPOCH]
                [--multiscale] [--rescale_interval RESCALE_INTERVAL]
                [--data_shuffle] [--gpu_num GPU_NUM]

optional arguments:
  -h, --help            show this help message and exit
  --model_type MODEL_TYPE
                        YOLO model type: mobilenet_lite/mobilenet/darknet/vgg1
                        6/xception/xception_lite, default=mobilenet_lite
  --tiny_version        Whether to use a tiny YOLO version
  --model_image_size MODEL_IMAGE_SIZE
                        Initial model image input size as <num>x<num>, default
                        416x416
  --weights_path WEIGHTS_PATH
                        Pretrained model/weights file for fine tune
  --freeze_level FREEZE_LEVEL
                        Freeze level of the model in initial train stage.
                        0:NA/1:backbone/2:only open prediction layer
  --annotation_file ANNOTATION_FILE
                        train annotation txt file, default=trainval.txt
  --val_annotation_file VAL_ANNOTATION_FILE
                        val annotation txt file, default=None
  --val_split VAL_SPLIT
                        validation data persentage in dataset if no val
                        dataset provide, default=0.1
  --classes_path CLASSES_PATH
                        path to class definitions,
                        default=configs/voc_classes.txt
  --learning_rate LEARNING_RATE
                        Initial learning rate, default=0.001
  --batch_size BATCH_SIZE
                        Initial batch size for train, default=16
  --init_epoch INIT_EPOCH
                        Initial stage training epochs, default=20
  --total_epoch TOTAL_EPOCH
                        Total training epochs, default=300
  --multiscale          Whether to use multiscale training
  --rescale_interval RESCALE_INTERVAL
                        Number of epoch interval to rescale input image,
                        default=10
  --data_shuffle        Whether to shuffle train/val data for cross-validation
  --gpu_num GPU_NUM     Number of GPU to use, default=1
```
Checkpoints during training could be found at logs/000/. Choose a best one as result

MultiGPU usage: use `--gpu_num N` to use N GPUs. It is passed to the [Keras multi_gpu_model()](https://keras.io/utils/#multi_gpu_model).

Loss type couldn't be changed from CLI options. You can try them by changing params in [model.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/yolo3/model.py)

### Model dump
We need to dump out inference model from training checkpoint for eval or demo. Following script cmd work for that.

```
# python yolo.py --model_type=mobilenet_lite --model_path=logs/000/<checkpoint>.h5 --anchors_path=configs/yolo_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --dump_model --output_model_file=model.h5
```

Change model_type, anchors file & class file for different training mode

### Evaluation
Use [eval.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/eval.py) to do evaluation on the inference model with your test data. It support following metrics:

1. Pascal VOC mAP: draw rec/pre curve for each class and AP/mAP result chart in "result" dir with default 0.5 IOU or specified IOU, and optionally save all the detection result on evaluation dataset as images

2. MS COCO AP evaluation. Will draw overall AP chart and AP on different scale (small, medium, large) as COCO standard. It can also optionally save all the detection result

```
# python eval.py --model_path=model.h5 --anchors_path=configs/yolo_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --eval_type=VOC --iou_threshold=0.5 --conf_threshold=0.001 --annotation_file=2007_test.txt --save_result
```

Following is a sample result trained on Mobilenet YOLOv3 Lite model with PascalVOC dataset (using a reasonable score threshold=0.1):
<p align="center">
  <img src="assets/mAP.jpg">
  <img src="assets/COCO_AP.jpg">
</p>

Some experiment on PascalVOC dataset(with download link) and comparison:

| Model name | InputSize | TrainSet | TestSet | mAP | Size | Speed | Ps |
| ----- | ------ | ------ | ------ | ----- | ----- | ----- | ----- |
| [YOLOv3 Lite-Mobilenet](https://pan.baidu.com/s/1kVEkmtKcM95RvpLAXK1k9g) | 320x320 | VOC07+12 | VOC07 | 73.31% | 31.8MB | 17ms | Keras on Titan XP |
| [YOLOv3 Lite-Mobilenet](https://pan.baidu.com/s/1kVEkmtKcM95RvpLAXK1k9g) | 416x416 | VOC07+12 | VOC07 | 75.93% | 31.8MB| 20ms | Keras on Titan XP |
| [YOLOv3 Lite-SPP-Mobilenet](https://drive.google.com/file/d/1tvZDWCYDbBGc9SOJKVdGt5G3O87UucdQ/view?usp=sharing) | 416x416 | VOC07+12 | VOC07 | 75.93% | 34MB | 22ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet](https://drive.google.com/file/d/1C4hviceMxQEWDIwQUOoRGoK-BMADHjfZ/view?usp=sharing) | 320x320 | VOC07+12 | VOC07 | 69.12% | 20.1MB | 9ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet](https://drive.google.com/file/d/1C4hviceMxQEWDIwQUOoRGoK-BMADHjfZ/view?usp=sharing) | 416x416 | VOC07+12 | VOC07 | 72.60% | 20.1MB | 11ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet with GIoU loss](https://drive.google.com/file/d/1m6f0EWc0P3LwJVzwd6yPCZsSjL7gKIFp/view?usp=sharing) | 416x416 | VOC07+12 | VOC07 | 72.73% | 20.1MB | 11ms | Keras on Titan XP |
| [YOLOv3-Xception](https://pan.baidu.com/s/1bB1v755_Oj5546Ej3cll3Q) | 512x512 | VOC07+12 | VOC07 | 78.89% | 419.8MB | 48ms | Keras on Titan XP |
| [YOLOv3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet) | 320x320 | VOC07 | VOC07 | 64.22% || 29fps | Keras on 1080Ti |
| [YOLOv3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet) | 320x320 | VOC07+12 | VOC07 | 74.56% || 29fps | Keras on 1080Ti |
| [YOLOv3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet) | 416x416 | VOC07+12 | VOC07 | 76.82% || 25fps | Keras on 1080Ti |
| [MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD) | 300x300 | VOC07+12+coco | VOC07 | 72.7% | 22MB |||
| [MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD) | 300x300 | VOC07+12 | VOC07 | 68% | 22MB |||
| [Faster RCNN, VGG-16](https://github.com/ShaoqingRen/faster_rcnn) | ~1000x600 | VOC07+12 | VOC07 | 73.2% || 151ms | Caffe on Titan X |
|[SSD,VGG-16](https://github.com/pierluigiferrari/ssd_keras) | 300x300 | VOC07+12 | VOC07	| 77.5% | 201MB | 39fps | Keras on Titan X |

And some unsuccessful experiment...

| Model name | InputSize | TrainSet | TestSet | mAP | Size | Speed | Ps |
| ----- | ------ | ------ | ------ | ----- | ----- | ----- | ----- |
| YOLOv3-VGG16 | 416x416 | VOC07+12 | VOC07 | 68.03% | 172MB |||

### Demo
1. [yolo.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/yolo.py)
> * Demo script for trained model

image detection mode
```
# python yolo.py --model_type=mobilenet_lite --model_path=model.h5 --anchors_path=configs/yolo_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --image
```
video detection mode
```
# python yolo.py --model_type=mobilenet_lite --model_path=model.h5 --anchors_path=configs/yolo_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --input=test.mp4
```
For video detection mode, you can use "input=0" to capture live video from web camera and "output=<video name>" to dump out detection result to another video

### Tensorflow model convert
Using [keras_to_tensorflow](https://github.com/amir-abdi/keras_to_tensorflow) to convert the keras .h5 model to tensorflow model (frozen pb):
```
# python keras_to_tensorflow.py
    --input_model="path/to/keras/model.h5"
    --output_model="path/to/save/model.pb"
```

### Inference model deployment
See [on-device inference](https://github.com/david8862/keras-YOLOv3-model-set/tree/master/inference) for TFLite & MNN model deployment


### TODO
- [ ] support pruned model training with [tensorflow_model_optimization](https://github.com/tensorflow/model-optimization/blob/master/tensorflow_model_optimization/g3doc/guide/pruning/train_sparse_models.md)
- [ ] provide more imagenet pretrained backbone (e.g. shufflenet, shufflenetv2), see [Training backbone](https://github.com/david8862/keras-YOLOv3-model-set/tree/master/yolo3/models/backbones/imagenet_training)


## Some issues to know

1. The test environment is
    - Python 3.6.8
    - tensorflow 2.0.0-rc2
    - tf.keras 2.2.4-tf

2. Default YOLOv3 anchors are used. If you want to use your own anchors, probably some changes are needed. tools/kmeans.py could be used to do K-Means anchor clustering on your dataset

3. Always load pretrained weights and freeze layers in the first stage of training.

4. The training strategy is for reference only. Adjust it according to your dataset and your goal. And add further strategy if needed.


# Citation
Please cite keras-YOLOv3-model-set in your publications if it helps your research:
```
@article{MobileNet-Yolov3,
     Author = {Adam Yang},
     Year = {2018}
}
@article{keras-yolo3,
     Author = {qqwweee},
     Year = {2018}
}
@article{yolov3,
     title={YOLOv3: An Incremental Improvement},
     author={Redmon, Joseph and Farhadi, Ali},
     journal = {arXiv},
     year={2018}
}
@article{Focal Loss,
     title={Focal Loss for Dense Object Detection},
     author={Tsung-Yi Lin, Priya Goyal, Ross Girshick, Kaiming He, Piotr Dollár},
     journal = {arXiv},
     year={2017}
}
@article{GIoU,
     title={Generalized Intersection over Union: A Metric and A Loss for Bounding Box
Regression},
     author={Hamid Rezatofighi, Nathan Tsoi1, JunYoung Gwak1, Amir Sadeghian, Ian Reid, Silvio Savarese},
     journal = {arXiv},
     year={2019}
}
```
