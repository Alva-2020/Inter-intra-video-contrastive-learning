Official code for paper, Self-supervised Video Representation Learning Using Inter-intra Contrastive Framework [ACMMM'20].

[Arxiv paper](https://arxiv.org/abs/2008.02531) [Project page](https://bestjuly.github.io/Inter-intra-video-contrastive-learning/)

## Requirements
> This is my experimental enviroment. 

- PyTorch 1.3.0
- python  3.7.4
- accimage 

## Inter-intra contrastive framework
For samples, we have
- [ ] Inter-positives: samples with **same labels**, not used for self-supervised learning;
- [x] Inter-negatives: **different samples**, or samples with different indexes;
- [x] Intra-positives: data from the **same sample**, in different views / from different augmentations; 
- [x] Intra-negatives: data from the **same sample** while some kind of information has been broken down. In video case, temporal information has been destoried.

Our work makes use of all usable parts (in this classification category) to form an inter-intra contrastive framework. The experiments here are mainly based on Contrastive Multiview Coding. 

It is flexible to extend this framework to other contrastive learning methods which use negative samples, such as MoCo and SimCLR.

![image](https://github.com/BestJuly/Inter-intra-video-contrastive-learning/blob/master/fig/general.png)

## Highlights
### Make the most of data for contrastive learning.
Except for inter-negative samples, all possible data are used to help train the network. This **inter-intra learning framework** can make the most use of data in contrastive learning.

### Flexibility of the framework
The **inter-intra learning framework** can be extended to
- Different contrastive learning methods: CMC, MoCo, SimCLR ...
- Different intra-negative generation methods: frame repeating, frame shuffling ...
- Different backbone: C3D, R3D, R(2+1)D, I3D ...


## Usage of this repo
### Data preparation
You can download UCF101/HMDB51 dataset from official website: [UCF101](http://crcv.ucf.edu/data/UCF101.php) and [HMDB51](http://serre-lab.clps.brown.edu/resource/hmdb-a-large-human-motion-database/). Then decoded videos to frames.    
I highly recommend the pre-comupeted optical flow images and resized RGB frames in this [repo](https://github.com/feichtenhofer/twostreamfusion).

### Train self-supervised learning part
```
python train_ssl.py --dataset=ucf101
```

This equals to

```
python train_ssl.py --dataset=ucf101 --model=r3d --modality=res --neg=repeat
```

This default setting uses frame repeating as intra-negative samples for videos. R3D is used.

We use two views in our experiments. View #1 is a RGB video clip, View #2 can be RGB/Res/Optical flow video clip. Residual video clips are default modality for View #2. You can use `--modality` to try other modalities. Intra-negative samples are generated from View #1. 

It may be wired to use only one optical flow channel *u* or *v*. We use only one channel to make it possible for **only one model** to handle inputs from different modalities. It is also an optional setting that using different models to handle each modality.

### Retrieve video clips
```
python retrieve_clips.py --ckpt=/path/to/your/model --dataset=ucf101 --merge=True
```
One model is used to handle different views/modalities. You can set `--modality` to decide which modality to use. When setting `--merge=True`, RGB for View #1 and the specific modality for View #2 will be jointly used for joint retrieval.

By default training setting, it is very easy to get over 30%@top1 for video retrieval in ucf101 and around 13%@top1 in hmdb51 without joint retrieval.

### Fine-tune model for video recognition
```
python ft_classify.py --ckpt=/path/to/your/model --dataset=ucf101
```
Testing will be automatically conducted at the end of training.

Or you can use
```
python ft_classify.py --ckpt=/path/to/your/model --dataset=ucf101 --mode=test
```
In this way, only testing is conducted using the given model.

**Note**: The accuracies using residual clips are not stable for validation set (this may also caused by), the final testing part will use the best model on validation set.

If everything is fine, you can achieve around 70% accuracy on UCF101. The results will vary from each other with different random seeds.

## Results
### Retrieval results
The table lists retrieval results on UCF101 *split* 1. We reimplemented CMC and report its results. Other results are from corresponding paper.

Method | top1 | top5 | top10 | top20 | top50
---|---|---|---|---|---
Jigsaw  | 19.7 | 28.5 | 33.5 | 40.0 | 49.4
OPN  | 19.9 | 28.7 | 34.0 | 40.6 | 51.6
R3D (random)  | 9.9 | 18.9 | 26.0 | 35.5 | 51.9
VCOP  | 14.1  |  30.3 | 40.4 | 51.1 | 66.5
VCP | 18.6 | 33.6 | 42.5 | 53.5 | 68.1
CMC  |  26.4  |  37.7  |  45.1  |  53.2  |  66.3 
Ours (repeat + res)  |  36.5  |  54.1  |  62.9  |  72.4  |  83.4 
Ours (repeat + u)  |  41.8  |  60.4  |  **69.5**  |  **78.4**  |  **87.7** 
Ours (shuffle + res)  |  34.6  |  53.0  |  62.3  |  71.7  |  82.4 
Ours (shuffle + v)  |  **42.4**  |  **60.9**  |  69.2  |  77.1  |  86.5 
RTT | 26.1 | 48.5	| 59.1 | 69.6 | 82.8
MemDPC | 20.2 |	40.4 | 52.4 | 64.7 | -


### Recognition results
We only use R3D as our network backbone. Usually, using Resnet-18-3D, R(2+1)D or deeper networks can yield better performance.

Method | UCF101 | HMDB51
---|---|---
Jigsaw |  51.5  |  22.5 
O3N (res)  |  60.3  |  32.5 
OPN  |  56.3  |  22.1
OPN (res)  |  71.8  |  36.7
CrossLearn  |  58.7  |  27.2 
CMC (3 views)  |  59.1  |  26.7
R3D (random)  | 54.5 | 23.4
ImageNet-inflated  |  60.3  |  30.7
3D ST-puzzle  |  65.8  |  33.7
VCOP  | 64.9 |  29.5 
VCP  |  66.0 |  31.5 
Ours (repeat + res) |  72.8  |  35.3 
Ours (repeat + u)  |  72.7  |  36.8 
Ours (shuffle + res) |  **74.4**  |  **38.3**
Ours (shuffle + v)  |  67.0  |  34.0 

**Residual clips + 3D CNN** The residual clips with 3D CNNs are effective. More information about this part can be found in [Rethinking Motion Representation: Residual Frames with 3D ConvNets for Better Action Recognition](https://arxiv.org/abs/2001.05661) (previous but more detailed version) and [Motion Representation Using Residual Frames with 3D CNN](https://arxiv.org/abs/2006.13017) (short version with better results).

The key code for this part is 
```
shift_x = torch.roll(x,1,2)
x = ((shift_x -x) + 1)/2
```
Which is slightly different from that in papers.

We also reimplement VCP in this [repo](https://github.com/BestJuly/VCP). By simply using residual cliops, significant improvements can be obtained for both video retrieval and video recognition.


## Citation
If you find our work helpful for your research, please consider citing the paper
```
@article{tao2020rethinking,
  title={Rethinking Motion Representation: Residual Frames with 3D ConvNets for Better Action Recognition},
  author={Tao, Li and Wang, Xueting and Yamasaki, Toshihiko},
  journal={arXiv preprint arXiv:2001.05661},
  year={2020}
}
```

If you find the residual input helpful for video-related tasks, please consider citing the paper
```
@article{tao2020rethinking,
  title={Rethinking Motion Representation: Residual Frames with 3D ConvNets for Better Action Recognition},
  author={Tao, Li and Wang, Xueting and Yamasaki, Toshihiko},
  journal={arXiv preprint arXiv:2001.05661},
  year={2020}
}

@article{tao2020motion,
  title={Motion Representation Using Residual Frames with 3D CNN},
  author={Tao, Li and Wang, Xueting and Yamasaki, Toshihiko},
  journal={arXiv preprint arXiv:2006.13017},
  year={2020}
}
```

## Acknowledgements
Part of this code is inspired by [CMC](https://github.com/HobbitLong/CMC) and [VCOP](https://github.com/xudejing/video-clip-order-prediction).
