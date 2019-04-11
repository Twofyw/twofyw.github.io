---
layout: post
title:  "GSoC 2019 Proposal for Improving the Chinese ASR Pipeline"
date:   2019-05-12 00:00:00 +0800
categories: ML
---

# GSoC 2019 Proposal for Improving the Chinese ASR Pipeline

> Wenxiang Yang
>
> ywywywx@gmail.com

## Abstract
The automatic speech recognition (ASR) pipeline facilitates the study of multimodal communication by transcribing audio into a text-searchable form, and provide formatted data to downstream NLP tasks. Red Hen Lab has a working Chinese ASR pipeline, and it is reported that the output makes sense, but has copious errors and disfluencies. This proposal aims to improve the current pipeline to the best performance by utilizing as many resources that we could obtain as possible. The improvement to performance will consist of two parts: enhancement to the input data and the model. More Chinese training data will be collected and processed, which can be used to fine-tune the existing ASR model. Multiple techniques for generating clean data will be experimented. This proposal also includes introducing a newer ASR model (wav2letter++). Documentation for the existing pipeline will be fixed, and the documentation for new work will be generated along the way.

## Performance improvements

Both the input data and the model are equally important to the performance of the pipeline. Thus the improvement to performance will consist of two parts: enhancement to the input data and the model. These two parts can branch out and process independently, taking advantage of the existing modular pipeline, to avoid stall when training the models.

### Data

#### Data preprocessing

The audio processing can be improved in two stages: first use VAD ([a nice implementation](https://github.com/wiseman/py-webrtcvad)) to properly cut audio, after that try other advanced audio processing techniques, such as [music removal](https://github.com/andabi/music-source-separation) and optionally [Speech Enhancement](https://github.com/jtkim-kaist/Speech-enhancement). The VAD that Google developed for the [WebRTC](https://webrtc.org/) project is reportedly one of the best available, being fast, modern and free.

To accelerate the pre-processing, [paralleled code](https://docs.python.org/3/library/concurrent.futures.html) will be written to process the data.

#### Datasets

The current ASR model uses a large language model whose 70GB of weights are pre-trained with Baidu Internal Corpus they provided. Fine tuning will suffer from the lack of labeled TV shows in our dataset, but we can try to fine tune [on more open Chinese datasets](https://discourse.mozilla.org/t/training-chinese-model/27769/3) and see if it works. We could fine-tune this model on more Chinese datasets other than Baidu Internal Corpus and see if it improves performance. Fine tuning works better on training data similar to test data (Chinese TV shows in our case), but only if we can obtain labeled TV shows to fine-tune the model, otherwise we can only try to fine tune on other open Chinese datasets.

Here is a list of more Chinese voice corpora:

- [LDC paid corpora](https://catalog.ldc.upenn.edu/LDC2010S07)
- OpenSLR
  - http://www.openslr.org/18/
  - http://www.openslr.org/33/
  - http://www.openslr.org/38/
  - http://www.openslr.org/47/
  - http://www.openslr.org/50/

- [Chinese word vectors](https://github.com/Embedding/Chinese-Word-Vectors) (not sure if it's useful).

Mozilla's Common Voice Dataset is continuously collecting voice data for Chinese, but currently, [no timeline](https://discourse.mozilla.org/t/timeline-for-releasing-the-deepspeech-models-trained-with-the-common-voice-data/29574) is released for the process. When the dataset grows bigger, it can potentially be added to our training dataset.

### Model
In this year's project, the performance of the current implementation will be improved, and a new implementation (wav2letter++) will be added.

The paper of Deep Speech 3, the direct successor to the current model, was released. However, there is no open source implementation of this model. Alternatives to upgrading the model to Deep Speech 3 would be 1. to improve the current model by further data processing and fine-tuning, or 2. to train a new Chinese model using Facebook’s wav2letter++, which is newer than Deep Speech 2, and migrate to that if it performs better. 

If we want to migrate to wav2letter++, it is not guaranteed to outperform the model used by the current Deep Speech 2 system, trained with Baidu Internal Corpus, which we can not obtain, and we can only train it with open datasets. Discussions about training wav2letter++ with Chinese [can be found](https://github.com/facebookresearch/wav2letter/issues/167).

There is a noteworthy [complete ASR system](https://github.com/mozilla/DeepSpeech) from Mozilla which is an open ­source implementation based on Baidu's Deep Speech 1. The project is stable, under active development and used by many other open source projects. However, the model architecture used is older than Deep Speech 2 and wav2letter++. Moreover, trained weights for Chinese are not provided, like the weights our system already has been using that is trained with Baidu Internal Corpus. There are attempts to training a Chinese model using this project, but benchmarks or comparison with other implementations can not be found online, which makes it unclear what will be the optimal hyper­parameters suitable for this model given the difference in vocabulary size between Chinese and English. 

On the other hand, because this project is a complete implementation of an ASR pipeline, the preprocessing steps of the input audio has been well developed, such as VAD (voice activity detection, used to cut audio between sentences). We could also learn from other open source projects based on this project, for example, a good [python interface to the WebRTC VAD](https://github.com/wiseman/py-webrtcvad) can be discovered because it’s used by [a front­end of DeepSpeech](https://github.com/AccelerateNetworks/DeepSpeech_Frontend). 

Combinations of hyper-parameters can be experimented and evaluated.


## Documentation and Code Quality
From the user's perspective, the documentation of Chinese ASR has not been merged to or linked from the [Chinese Pipeline](https://sites.google.com/site/distributedlittleredhen/home/the-cognitive-core-research-topics-in-red-hen/chinese-pipeline) wiki page. Throughout this year's project, documentation for the existing pipeline will be fixed, and the documentation for any new work will be generated along the way. 

Red Hen aims to generate code that is as simple to maintain as possible, so the system will be designed so that it will be easy to maintain, modular in design where possible, and avoiding any layers of code that are not strictly necessary.

The impact on the performance after each experiment will be recorded.


## Timeline
Works on the data processing and the model can branch out and process independently, taking advantage of the existing modular pipeline, and avoid stall when training the models. Throughout this project, documentation for the existing pipeline will be fixed, and the documentation for any new work will be generated along the way. 

| Date                                | Data                                                         | Model                                                        |
| ----------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| May 7 - 27 (Community Bonding)      | Familiarize with the data formats, existing code and read papers to understand the theory behind ASR. Hand on Singularity and reproduce the last year's result. | Play with Deep Speech 2.                                     |
| Before June 25 (First Evaluation)   | Use VAD to properly cut audio.                               | Play with wav2letter++. Containerize wav2letter++ to run on Singularity. Train models on audio cut with the new method. |
| Before July 23 (Second Evaluation)  | Collect and process more Chinese corpora.                    | Fine-tune Deep Speech 2 and start training wav2letter++. Integrate trained models into the pipeline and evaluate their performance. |
| Before August 20 (Final Evaluation) | Evaluate other advanced pre-processing techniques listed above. | Stretch Goals.                                               |

## Stretch Goals
If all above goes well and I have remaining time during the summer, I will try to apply forced alignment ([implementation](https://github.com/pettarin/forced-alignment-tools), [paper](http://languagelog.ldc.upenn.edu/myl/MandarinPhoneticSegmentation.pdf)) and some basic NLP on the output of the ASR system. In the documentation, walk through of these basic applications will be added as examples.

## Biographical Information

I’m a third year CS student at Tongji University, Shanghai, supervised by [Prof. Yin Wang](http://web.eecs.umich.edu/~yinw/). My experience is mainly with computer vision, with a publication to CVPR workshop on image segmentation. My experience also includes NLP. Moreover, I can prototype a deep learning system using PyTorch fairly quickly. I’m proficient in Python, C++. Having been a daily Linux user for years, I'm used to getting work done programmatically using shell scripts. Here is a [link](https://github.com/twofyw) to my Github account.