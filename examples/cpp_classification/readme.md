---
title: CaffeNet C++ Classification example
description: A simple example performing image classification using the low-level C++ API.
category: example
include_in_docs: true
priority: 10
---

# Classifying ImageNet: using the C++ API

Caffe, at its core, is written in C++. It is possible to use the C++
API of Caffe to implement an image classification application similar
to the Python code presented in one of the Notebook examples. To look
at a more general-purpose example of the Caffe C++ API, you should
study the source code of the command line tool `caffe` in `tools/caffe.cpp`.

## Presentation

A simple C++ code is proposed in
`examples/cpp_classification/classification.cpp`. For the sake of
simplicity, this example does not support oversampling of a single
sample nor batching of multiple independent samples. This example is
not trying to reach the maximum possible classification throughput on
a system, but special care was given to avoid unnecessary
pessimization while keeping the code readable.

## Compiling

The C++ example is built automatically when compiling Caffe. To
compile Caffe you should follow the documented instructions. The
classification example will be built as `examples/classification.bin`
in your build directory.

## Usage

To use the pre-trained CaffeNet model with the classification example,
you need to download it from the "Model Zoo" using the following
script:
```
./scripts/download_model_binary.py models/bvlc_reference_caffenet
```
The ImageNet labels file (also called the *synset file*) is also
required in order to map a prediction to the name of the class:
```
./data/ilsvrc12/get_ilsvrc_aux.sh
```
Using the files that were downloaded, we can classify the provided cat
image (`examples/images/cat.jpg`) using this command:
```
예제
./build/examples/cpp_classification/classification.bin \
  models/bvlc_reference_caffenet/deploy.prototxt \
  models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel \
  data/ilsvrc12/imagenet_mean.binaryproto \
  data/ilsvrc12/synset_words.txt \
  examples/images/cat.jpg

내꺼(하기전에 메이크 다시 해야함)
./build/examples/cpp_classification/classification_mnist.bin \
  examples/mnist/lenet.prototxt \
  examples/mnist/lenet_iter_10000.caffemodel \
  examples/mnist/label.txt \
  examples/mnist/seven.png

설명
./build/examples/cpp_classification/classification.bin \
  models/bvlc_reference_caffenet/deploy.prototxt \네트워크 레이어에 대한 정보(카페넷에 대한 정보)가 들어있음
  models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel \이미 누군가 돌려서 나온 데이터 결과값
  data/ilsvrc12/imagenet_mean.binaryproto
\중간 값을 뺌(이미지 전처리) 데이터들의 차이에 집중하기 위해 / 이건 숫자예제 할떄는 지워야 함
데이터 1000개 처리 - 데이터를 각각 보는것/ ex 파란 빛 밑에서 사진 을 찎었으면 그 사진은 전체적으로 파란색이 많이 포함
구별은 차이를 구하는 것이므로 같은 파란색이 모두 포함돼있으면 의미 없는 값이므로 없에버리는게 좋음
이런 걸 방지하기 위해 모든 값의 평균값을 구해 민감도를 낮춘다.
일반적으로 학습시에 중간 값을 뺴는데, 엠니스트는 중간값을 뺴지 않는 모델이라 
모든 데이터의 평균값을 뺌

  data/ilsvrc12/synset_words.txt \나온 결과를 말로 어떻게 설명할껀
  examples/images/cat.jpg


``

The output should look like this:
```
---------- Prediction for examples/images/cat.jpg ----------
0.3134 - "n02123045 tabby, tabby cat"
0.2380 - "n02123159 tiger cat"
0.1235 - "n02124075 Egyptian cat"
0.1003 - "n02119022 red fox, Vulpes vulpes"
0.0715 - "n02127052 lynx, catamount"
```

## Improving Performance

To further improve performance, you will need to leverage the GPU
more, here are some guidelines:

* Move the data on the GPU early and perform all preprocessing
operations there.
* If you have many images to classify simultaneously, you should use
batching (independent images are classified in a single forward pass).
* Use multiple classification threads to ensure the GPU is always fully
utilized and not waiting for an I/O blocked CPU thread.
