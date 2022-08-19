# keras-yolo3

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE)

## Introduction

A Keras implementation of YOLOv3 (Tensorflow backend) inspired by [allanzelener/YAD2K](https://github.com/allanzelener/YAD2K).


## Blog

日本語での解説はこちらの記事をご確認ください。  
[【物体検出】keras−yolo3の学習方法](https://sleepless-se.net/2019/06/21/how-to-train-keras%e2%88%92yolo3/)  
[【物体検出】keras−yolo3で独自データセットの学習方法](https://sleepless-se.net/2019/06/21/how-to-train-keras%E2%88%92yolo3/)  
[【物体検出】(GPU版)YOLO独自モデル簡単学習方法](https://sleepless-se.net/2019/06/23/yolo-original-model-training-on-colab/)  

---


## dataset

麻雀牌を麻雀用マットの上に複数枚並べて撮影してできた49枚の画像  
1から9までのピンズ牌（以下、1p, 2p, 3p, ...,9Pと呼称）のみを対象  

## resize

撮影に使用したスマートフォンの都合上、画像サイズは4032×3024 pixel  
ただし、この後、Googl Colabo上で学習させることを想定し、  
画像サイズを216×162 pixelにリサイズ

## annotation

アノテーションとは、画像のどこに対象物があるのかを伝えること。  
VoTT(Visual Object Tagging Tool, Microsoft社)を用いてアノテーション 作業を実施  

VoTTの使い方については、以下を参照  
[ 【物体検出】アノテーションツールVoTTの使い方](https://sleepless-se.net/2019/06/21/how-to-use-vott/)  

## training

Google Colaboratoryを用いて、学習を行う  
以下、ソースコード  

学習終了後、学習済みモデルをローカルPC上に保存  

使用上の制約があるため、要注意  
1. 12時間ルール  
2. 90分セッション切れルール  


## test

ローカル環境でYOLOを動かす  

結果がこちら...  


全然だめでした。  
おそらく、原因として考えられるのは、    
・学習に使う画像データの枚数が不足  
 --> YOLOv3公式ドキュメントでは、1ラベルあたり100~200枚の画像データが必要とのこと。アノテーション作業で時間が潰れる...
・Google Colab上での学習が途中  
 --> 先述の制約が原因。対処方法検討中

## Quick Start

1. Download YOLOv3 weights from [YOLO website](http://pjreddie.com/darknet/yolo/).
2. Convert the Darknet YOLO model to a Keras model.
3. Run YOLO detection.

```
wget https://pjreddie.com/media/files/yolov3.weights
python convert.py yolov3.cfg yolov3.weights model_data/yolo.h5
python yolo_video.py [OPTIONS...] --image, for image detection mode, OR
python yolo_video.py [video_path] [output_path (optional)]
```

For Tiny YOLOv3, just do in a similar way, just specify model path and anchor path with `--model model_file` and `--anchors anchor_file`.

### Usage
Use --help to see usage of yolo_video.py:
```
usage: yolo_video.py [-h] [--model MODEL] [--anchors ANCHORS]
                     [--classes CLASSES] [--gpu_num GPU_NUM] [--image]
                     [--input] [--output]

positional arguments:
  --input        Video input path
  --output       Video output path

optional arguments:
  -h, --help         show this help message and exit
  --model MODEL      path to model weight file, default model_data/yolo.h5
  --anchors ANCHORS  path to anchor definitions, default
                     model_data/yolo_anchors.txt
  --classes CLASSES  path to class definitions, default
                     model_data/coco_classes.txt
  --gpu_num GPU_NUM  Number of GPU to use, default 1
  --image            Image detection mode, will ignore all positional arguments
```
---

4. MultiGPU usage: use `--gpu_num N` to use N GPUs. It is passed to the [Keras multi_gpu_model()](https://keras.io/utils/#multi_gpu_model).

## Training

1. Generate your own annotation file and class names file.  
    One row for one image;  
    Row format: `image_file_path box1 box2 ... boxN`;  
    Box format: `x_min,y_min,x_max,y_max,class_id` (no space).  
    For VOC dataset, try `python voc_annotation.py`  
    Here is an example:
    ```
    path/to/img1.jpg 50,100,150,200,0 30,50,200,120,3
    path/to/img2.jpg 120,300,250,600,2
    ...
    ```

2. Make sure you have run `python convert.py -w yolov3.cfg yolov3.weights model_data/yolo_weights.h5`  
    The file model_data/yolo_weights.h5 is used to load pretrained weights.

3. Modify train.py and start training.  
    `python train.py`  
    Use your trained weights or checkpoint weights with command line option `--model model_file` when using yolo_video.py
    Remember to modify class path or anchor path, with `--classes class_file` and `--anchors anchor_file`.

If you want to use original pretrained weights for YOLOv3:  
    1. `wget https://pjreddie.com/media/files/darknet53.conv.74`  
    2. rename it as darknet53.weights  
    3. `python convert.py -w darknet53.cfg darknet53.weights model_data/darknet53_weights.h5`  
    4. use model_data/darknet53_weights.h5 in train.py

---

## Some issues to know

1. The test environment is
    - Python 3.7.?
    - Keras 2.1.5
    - tensorflow 1.14.0

2. Default anchors are used. If you use your own anchors, probably some changes are needed.

3. The inference result is not totally the same as Darknet but the difference is small.

4. The speed is slower than Darknet. Replacing PIL with opencv may help a little.

5. Always load pretrained weights and freeze layers in the first stage of training. Or try Darknet training. It's OK if there is a mismatch warning.

6. The training strategy is for reference only. Adjust it according to your dataset and your goal. And add further strategy if needed.

7. For speeding up the training process with frozen layers train_bottleneck.py can be used. It will compute the bottleneck features of the frozen model first and then only trains the last layers. This makes training on CPU possible in a reasonable time. See [this](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html) for more information on bottleneck features.

