&nbsp;
<p align="center"><img src="images/synergy_sportradar_logo.png" width="640"></p>

# SoccerNet Re-Identification
Welcome to the Development Kit for the SoccerNet Re-Identification Task and Challenge.
This kit is meant as a help to get started working with the SoccerNet data and the proposed task.
In this task, participants will have to re-identify soccer players across multiple camera viewpoints.
More general information about the SoccerNet dataset can be found on our [official website](https://www.soccer-net.org/).
In this repository, you will find all the instructions and codebase for participating in this challenge and get a chance to win the 1000$ prize money!
The winner of the competition will be announced in June 2022.

&nbsp;
<p align="center"><img src="images/multiview_links.png" width="640"></p>
&nbsp;

SoccerNet Re-Identification (ReID) is part of the SoccerNet-v3 dataset, which is an extension of SoccerNet-v2 with new and challenging tasks including 
action spotting, camera shot segmentation with boundary detection, replay grounding, calibration, multi-view player re-identification and multi-player tracking.
The participation deadline is fixed at the 30th of May 2022.
The official rules and guidelines are available on [ChallengeRules.md](ChallengeRules.md).


Make sure to watch our announcement video on Youtube:

&nbsp;
<a href="https://youtu.be/tA9E1hkiyB0">
<p align="center"><img src="images/Thumbnail.png" width="520"></p>
</a>

## Challenge description
The SoccerNet Re-Identification (ReID) dataset is composed of 340.993 players thumbnails extracted from image frames of broadcast videos from 400 soccer games within 6 major leagues.
The goal of the challenge is to re-identify soccer players across multiple camera viewpoints depicting the same action during the game.

We call the first frame in which the action has been seen in the broadcast video as the "action frame".
The other frames depicting the same action (at roughly the same timestamp), and appearing later in the broadcast video, are called the "replay frame".
The goal of the challenge is to re-identify each soccer player in the action frame across the different replay frames.
For that purpose, each bounding box from a given action frame will be considered as a query, and bounding boxes from the corresponding replay frames will be ranked according to their distance to each query.
The SoccerNet ReID dataset is divided into training, validation, test and challenge sets.
Labels for the challenge set are kept secret as participants to the challenge will be ranked according to their performance on it.
The winner of the challenge will be the participant with the highest mAP score on the challenge set. 

Compared to traditional street surveillance type re-identification dataset, the SoccerNet-v3 ReID dataset is particularly
challenging because soccer players from the same team have very similar appearance, which makes it hard to tell them apart.
On the other hand, each identity has a few number of samples, which render the model harder to train.
There is also a big diversity in image resolution.

**Important note**: Player identity labels are derived from links between bounding boxes within an action and are therefore only valid within the given action.
Consequently, player identity labels do not hold across actions and a given player has a different identity for each action he has been spotted in.
For that reason, during the evaluation process, only samples within the same action are matched against each other.
During the training stage, it is up to the participants to implement strategies to mitigate the negative impact of image samples from different actions depicting the same player but labeled with different identities.

&nbsp;

![Alt Text](images/soccernet_correspondences_half_fr_compressed.gif)
&nbsp;


### Dataset structure
ReID samples are organized following the original SoccerNet dataset structure.
The Soccernet-v3 ReID dataset is thus organized within a tree folder structure :

`root` -> `{train, valid, test, challenge}` -> `championship` -> `season` -> `game` -> `action` -> `image files`

Each action contains a set of samples, which are images files of players or referees cropped from the original Soccernet-v3 dataset using the bounding boxes annotations.
Annotations for each sample are provided within the filename, as described in the next section.


### Filename convention
#### Train/valid/test sets
Filename convention for samples in the train/valid/test sets:
```
<bbox_idx>_<action_idx>_<person_uid>_<frame_idx>_<class>_<ID>_<UAI>_<height>x<width>.png
```
e.g. '54404-1940-33397-5089-Player_team_right-9-014r002_00858c3b0003-775x449.png'

1. `bbox_idx`: bounding box (or sample) index within the current train/query/gallery list of samples.
2. `action_idx`: action index within the current set (train/valid/test/challenge). An action corresponds to an action frame and its related replay frames.
3. `person_uid`: person identifier or PID, unique across all sets. Be aware that the same player/referee will be assigned different pids in different actions. All samples with a given 'person_uid' will therefore have the same 'action_idx'.     
4. `frame_idx`: frame index (action frame or replay frame) within the current set (train/valid/test/challenge).
5. `class`: person class within :
    1. "Player team left"
    2. "Player team right"
    3. "Goalkeeper team left"
    4. "Goalkeeper team right"
    5. "Main referee"
    6. "Side referee"
    7. "Staff members"
    8. "Player team unknown 1"
    9. "Player team unknown 2"
    10. "Goalkeeper team unknown"
6. `ID`: this field corresponds to the "ID" field from the original soccernet-v3 annotations and the 'person_uid' field is derived from it. It refers to the identifier of the player within the action:
   1. It corresponds to the player jersey number when it is given as a number. The jersey number is provided for a player if it can be seen at least once within one frame of the action. 
   2. If the jersey number is not visible in any frame of the action, then this field is given as a letter.
   3. The field is set to 'None' if the player cannot be seen in any other frame of the action.
8. `UAI`: item UAI from original soccernet-v3 annotations.
9. `height`: image height.
10. `width`: image width.

Participants are encouraged to use any provided information (image height/width, action_idx, class, ID) to develop better training strategies.

#### Challenge set
Annotations for the challenge set are kept secret, the filename convention for challenge set samples is therefore:
```
<bbox_idx>_<action_idx>_<height>x<width>.png
```

### Evaluation
Similar to standard ReID datasets, each valid/test/challenge set is split into two subsets: query and gallery.
Query samples are bounding boxes from action frames with at least one match in the replay frames.
Gallery samples are bounding boxes from replay frames, or bounding boxes from action frames with no match.
For each query, we compute a ranking of gallery samples **from the same action** and compute corresponding ranking performance: rank-1 and average precision.
Query sample are therefore only matched against gallery samples from the same action. 
This is different from classic street surveillance ReID datasets, where query samples are only matched against gallery samples from different camera viewpoints. 
Our process for building the query/gallery split is illustrated in the next figure.

&nbsp;
<p align="center"><img src="images/soccernet-v3-reid-illustration.png" width="850"></p>

## Instructions
This repo is a fork of the [Torchreid](https://github.com/KaiyangZhou/deep-person-reid) framework for person re-identification.
Make sure to have a look at the original [Torchreid Readme](TORCHREID_README.rst) and the official [Torchreid documentation](https://kaiyangzhou.github.io/deep-person-reid/.) for detailed how-to instructions.

### How to install the SoccerNet ReID development kit
For installation guidelines, we adapt the instructions from the original [Torchreid Readme](TORCHREID_README.rst) :

First, make sure [conda](https://www.anaconda.com/distribution/) is installed.

```
    # cd to your preferred directory and clone this repo
    git clone https://github.com/SoccerNet/sn-reid.git

    # create environment
    cd sn-reid/
    conda create --name sn-reid python=3.7
    conda activate sn-reid

    # install dependencies
    # make sure `which python` and `which pip` point to the correct path
    pip install -r requirements.txt

    # install torch and torchvision (select the proper cuda version to suit your machine)
    conda install pytorch torchvision cudatoolkit=9.0 -c pytorch

    # install torchreid (don't need to re-build it if you modify the source code)
    python setup.py develop
```

### How to train and test a baseline model
We provide a basic script in [benchmarks/baseline](benchmarks/baseline) for training a baseline model on the Soccernet-v3 training set, evaluate rank-1 and mAP performance on the Soccernet-v3
validation and test set, and finally export ranking results on the Soccernet-v3 challenge set for external evaluation.

To train the baseline model, run:

```
python benchmarks/baseline/main.py --config-file benchmarks/baseline/configs/baseline_config.yaml
```

Running this script will automatically download the dataset in the folder specified by the `data.root` config.

Have a look at the YAML configuration file [baseline_config.yaml](benchmarks/baseline/configs/baseline_config.yaml) and related default configuration [default_config.py](benchmarks/baseline/default_config.py) for more information about the available options.

### How to manually download the dataset
The SoccerNet ReID dataset is automatically downloaded and extracted by Torchreid upon first usage, 
but you can still use the [SoccerNet pip package](https://pypi.org/project/SoccerNet/) to easily download the data and annotations manually. 

To install the pip package simply run:

```
pip install SoccerNet
```

Then use the API to download the data of interest:

```
from SoccerNet.Downloader import SoccerNetDownloader
mySoccerNetDownloader = SoccerNetDownloader(LocalDirectory="/path/to/project/datasets/soccernetv3")
mySoccerNetDownloader.downloadDataTask(task="reid", split=["train", "valid", "test", "challenge"])
```

This will download four zip files under `/path/to/project/datasets/soccernetv3/reid/{train.zip, valid.zip, test.zip, challenge.zip}`.
Unzip these four zip files and put four resulting folders under the same path, i.e. `/path/to/project/datasets/soccernetv3/reid/{train, valid, test, challenge}`.
The Torchreid config `data.root` should then point to `/path/to/project/datasets` or simply `datasets`, the framework will then look for the four sets under `/path/to/project/datasets/soccernetv3/reid/`.
In above instructions, replace `/path/to/project/datasets` by any path of your choice, but keep the `soccernetv3/reid/{train, valid, test, challenge}` folder structure.

For further information, have a look at [soccernetv3.py](torchreid/data/datasets/image/soccernetv3.py)).

| Dataset       | Size |
| ---           |  ---    |
| train.zip     | 12.12 GB
| valid.zip     | 2.35 GB
| test.zip      | 2.41 GB
| challenge.zip | 1.75 GB

### How to export your ranking results for the challenge set
Enable the `test.export_ranking_results` config in [default_config.py](benchmarks/baseline/default_config.py) or [baseline_config.yaml](benchmarks/baseline/configs/baseline_config.yaml) 
to export all ranking results for each target dataset. 
Ranking results will be exported to a JSON file `ranking_results_***.json` in the directory specified by the `data.save_dir` config.

To export ranking results for the SoccerNet ReID challenge set, make sure to add `'soccernetv3_challenge'` to the list of target datasets in the `data.targets` config.

### How to test the exported ranking results locally
To evaluate locally the ranking performance on the exported `ranking_results_***.json`, you can use the [evaluation script](tools/evaluate_soccernetv3_reid.py) provided in the tools directory:

```
python tools/evaluate_soccernetv3_reid.py -rs ranking_results_***.json -gt /path/to/project/datasets/soccernetv3/reid/test/test_bbox_info.json
```

For that purpose, you'll need to provide the location of the corresponding ground truth file, i.e. `test_bbox_info.json` for the test set, which is part of the dataset you downloaded before.
Make sure to use the correct ground truth file (valid or test) with respect to the set on which the ranking result file was computed.
Exported ranking results cannot be evaluated locally for the challenge set because the corresponding ground truth file is kept private from the participants. 

### How to submit your ranking results to participate in the challenge
In order to participate in the challenge, the exported ranking results for the challenge set should be submitted as a JSON file on the online evaluation platform [EvalAI](https://eval.ai/web/challenges/challenge-page/1538/evaluation).
Further information can be found on the EvalAI website.


## Future improvements
This repository will be actively maintained during the course of the challenge, make sure to subscribe and come back frequently to get the latest updates!
Here are some of the things we plan to add or improve:
- Some stats about the dataset size.
- Faster ranking script in rank.py to assess model performance.
- ...

Feel free to suggest any changes.


## Our other Challenges
Check out our other challenges related to SoccerNet:
- [Action Spotting](https://github.com/SoccerNet/sn-spotting)
- [Replay Grounding](https://github.com/SoccerNet/sn-grounding)
- [Calibration](https://github.com/SoccerNet/sn-calibration)
- [Re-Identification](https://github.com/SoccerNet/sn-reid)
- [Tracking](https://github.com/SoccerNet/sn-tracking)


## Citation
For further information check out the paper and supplementary material:
https://openaccess.thecvf.com/content/CVPR2021W/CVSports/papers/Deliege_SoccerNet-v2_A_Dataset_and_Benchmarks_for_Holistic_Understanding_of_Broadcast_CVPRW_2021_paper.pdf

Please cite our work if you use the SoccerNet dataset:
```bibtex
@InProceedings{Deliège2020SoccerNetv2,
      title={SoccerNet-v2 : A Dataset and Benchmarks for Holistic Understanding of Broadcast Soccer Videos}, 
      author={Adrien Deliège and Anthony Cioppa and Silvio Giancola and Meisam J. Seikavandi and Jacob V. Dueholm and Kamal Nasrollahi and Bernard Ghanem and Thomas B. Moeslund and Marc Van Droogenbroeck},
      year={2021},
      booktitle = {The IEEE Conference on Computer Vision and Pattern Recognition (CVPR) Workshops},
      month = {June},
}
```


## Useful links
- [Soccernet official website](https://soccer-net.org/)
- [ACAD Research](https://www.youtube.com/channel/UCYkYA7OwnM07Cx78iZ6RHig)
- [Torchreid](https://github.com/KaiyangZhou/deep-person-reid)


## Questions and Remarks
If you have any question or remark regarding the challenge and related materials, please raise a GitHub issue in this repository, or contact us directly on [Discord](https://discord.gg/SM8uHj9mkP).


## Sponsorship
This challenge is officially sponsored by [Synergy Sports](https://synergysports.com/), a division of [Sportradar](https://www.sportradar.com/).

