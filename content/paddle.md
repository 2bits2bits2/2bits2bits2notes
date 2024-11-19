---
title: "PaddleOCR scratch"
draft: false
tags:
  - 
---

 [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) is basically framework for creating and finetuning ocr models. It has some problems like: 
* most of discussion and documentation is in Chinese 
* English docs are old and sometimes have confusing information
* you need specific combination of python libraries to make it work
* PaddleLabel that can be used for creating dataset for it doesn't work out of box (so you can't just pip install it and except it to work)

But it is probably most powerfull framwework for creating more traditional OCR models. So if this doesnt scare you here are some resource to help you get started:

* [Fine-Tuning PaddleOCR’s Recognition Model For Dummies by A Dummy](https://anushsom.medium.com/finetuning-paddleocrs-recognition-model-for-dummies-by-a-dummy-89ac7d7edcf6)
* [Training PaddleOCR Scene Text Recognition in the Wild with Custom Dataset](https://medium.com/@prishanga1/paddleocr-scene-text-recognition-in-the-wild-with-custom-dataset-59fd5f5cf6c3)
* [Synthtiger for creating synthetic OCR dataset](https://github.com/clovaai/synthtiger)
* [Paddle ocr download page](https://www.paddlepaddle.org.cn/en/install/quick?docurl=/documentation/docs/en/install/pip/linux-pip_en.html) - use this, github page is useless for installation
* [Paddle ocr docs](https://github.com/PaddlePaddle/PaddleOCR/blob/main/doc/doc_en/training_en.md)

