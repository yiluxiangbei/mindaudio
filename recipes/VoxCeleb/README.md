# Speaker recognition experiments with VoxCeleb.
This folder contains scripts for running speaker identification and verification experiments with the VoxCeleb dataset(http://www.robots.ox.ac.uk/~vgg/data/voxceleb/).

# Speaker verification using ECAPA-TDNN embeddings
Run the following command to train speaker embeddings using [ECAPA-TDNN](https://arxiv.org/abs/2005.07143):

`python train_speaker_embeddings.py`

The speaker-id accuracy should be around 98-99% for both voxceleb1 and voceleb2.

After training the speaker embeddings, it is possible to perform speaker verification using cosine similarity, The pretrained model ckpt path is at(https://download.mindspore.cn/toolkits/mindaudio/ecapatdnn/). You can download the pretrained model ckpt, run it with the following command:

`python speaker_verification_cosine.py`

This system achieves:
- EER = 1.50% (voxceleb1 + voxceleb2) with s-norm
- EER = 1.70% (voxceleb1 + voxceleb2) without s-norm

These results are all obtained with the official verification split of voxceleb1 (veri_test2.txt).

We use five-times data augment to obtain this precision, which need 2.6T disk space. If you want to achieve EER(0.8%), you should use fifty-times data augment, which you can just change the hyper-param `number_of_epochs` in `ecapatdnn.yaml` to 10.
However, fifty-times data augment need 26T disk space.

Below you can find the results from model trained on VoxCeleb 2 dev set and tested on VoxSRC derivatives. Note that however, the models are trained on Ascend910 with 8 cards.

# VoxCeleb1 + VoxCeleb2 preparation
Voxceleb2 audio files are released in m4a format. All the files must be converted in wav files before
feeding them is MindAudio. Please, follow these steps to prepare the dataset correctly:

1. Download both Voxceleb1 and Voxceleb2.
You can find download instructions here: http://www.robots.ox.ac.uk/~vgg/data/voxceleb/
Note that for the speaker verification experiments with Voxceleb2 the official split of voxceleb1 is used to compute EER.


2. Convert .m4a to wav.
Voxceleb2 stores files with the m4a audio format. To use them within MindAudio you have to convert all the m4a files into wav files.
You can do the conversion using ffmpeg(https://gist.github.com/seungwonpark/4f273739beef2691cd53b5c39629d830). This operation might take several hours and should be only once.


3. Put all the wav files in a folder called wav. You should have something like `voxceleb12/wav/id*/*.wav` (e.g, `voxceleb12/wav/id00012/21Uxsk56VDQ/00001.wav`)


4. copy the `voxceleb1/vox1_test_wav.zip` file into the voxceleb12 folder.


5. Unpack voxceleb1 test files(verification split). Go to the voxceleb2 folder and run `unzip vox1_test_wav.zip`.


6. copy the `voxceleb1/vox1_dev_wav.zip` file into the voxceleb12 folder.


7. Unpack voxceleb1 dev files, Go to the voxceleb12 folder and run `unzip vox1_dev_wav.zip`.


8. Unpack voxceleb1 dev files and test files in dir `voxceleb1/`. You should have something like `voxceleb1/wav/id*/*.wav`.


9. The data augment needs to use `rirs_noises.zip` file which you can download from here:http://www.openslr.org/resources/28/rirs_noises.zip and put it in the `voxceleb12/` dir.

# Train
After Voxceleb1 + Voxceleb2 dataset is prepared, you can run the code below to generate preprocessed training audio data and train speaker embeddings with single card:

`python train_speaker_embeddings.py`

Attention:Voxceleb1 + Voxceleb2 dataset is so big so that the time of processing audio data is so long, so we use multiprocess with 30 processes to precess audio data.

If you mechine is not supported, please change the var `data_process_num` in the `ecapatdnn.yaml` file to the value your mechine support.

Also, when the preprocessed audio data is generated, you can run the code below only to train speaker embeddings with single card:

`python train_speaker_embeddings.py --need_generate_data=False`

Also when the preprocessed audio data is generated, you can run the code below only to train speaker embeddings with multi cards through this sh code:

`bash ./run_distribute_train_ascend.sh hccl.json`

The hccl.json is generated by hccl tools, you can refer to this article (https://gitee.com/mindspore/models/tree/master/utils/hccl_tools)

# Evaluate
After your model is trained,  you can run the code below to generate preprocessed testing audio data and evalute speaker verification:

`python speaker_verification_cosine.py`

Also, when the preprocessed testing audio data is generated, you can run the code below only to evalute speaker verification:

`python speaker_verification_cosine.py --need_generate_data=False`
