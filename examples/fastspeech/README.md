# Fastspeech
Paddle fluid implementation of Fastspeech, a feed-forward network based on Transformer. The implementation is based on [FastSpeech: Fast, Robust and Controllable Text to Speech](https://arxiv.org/abs/1905.09263).

We implement Fastspeech model in paddle fluid with dynamic graph, which is convenient for flexible network architectures.

## Installation

### Install paddlepaddle
This implementation requires the latest develop version of paddlepaddle. You can either download the compiled package or build paddle from source.
1. Install the compiled package, via pip, conda or docker. See [**Installation Mannuals**](https://www.paddlepaddle.org.cn/documentation/docs/en/beginners_guide/install/index_en.html) for more details.

2. Build paddlepaddle from source. See [**Compile From Source Code**](https://www.paddlepaddle.org.cn/documentation/docs/en/beginners_guide/install/compile/fromsource_en.html) for more details. Note that if you want to enable data parallel training for multiple GPUs, you should set `-DWITH_DISTRIBUTE=ON` with cmake.

### Install parakeet
You can choose to install via pypi or clone the repository and install manually.

1. Install via pypi.
   ```bash
   pip install parakeet
   ```

2. Install manually.
   ```bash
   git clone <url>
   cd Parakeet/
   pip install -e .

### Download cmudict for nltk
You also need to download cmudict for nltk, because convert text into phonemes with `cmudict`.

```python
import nltk
nltk.download("punkt")
nltk.download("cmudict")
```

If you have completed all the above installations, but still report an error at runtime：

``` OSError: sndfile library not found ```

You need to install ```libsndfile``` using your distribution’s package manager. e.g. install via:

``` sudo apt-get install libsndfile1 ```

## Dataset

We experiment with the LJSpeech dataset. Download and unzip [LJSpeech](https://keithito.com/LJ-Speech-Dataset/).

```bash
wget https://data.keithito.com/data/speech/LJSpeech-1.1.tar.bz2
tar xjvf LJSpeech-1.1.tar.bz2
```

## Model Architecture

![FastSpeech model architecture](./images/model_architecture.png)

FastSpeech is a feed-forward structure based on Transformer, instead of using the encoder-attention-decoder based architecture. This model extract attention alignments from an encoder-decoder based teacher model for phoneme duration prediction, which is used by a length
regulator to expand the source phoneme sequence to match the length of the target
mel-spectrogram sequence for parallel mel-spectrogram generation. We use the TransformerTTS as teacher model.
The model consists of encoder, decoder and length regulator three parts.

## Project Structure
```text
├── config                 # yaml configuration files
├── synthesis.py           # script to synthesize waveform from text
├── train.py               # script for model training
```

## Train Transformer

FastSpeech model can train with ``train.py``.
```bash
python train.py \
--use_gpu=1 \
--use_data_parallel=0 \
--data_path=${DATAPATH} \
--transtts_path='../transformer_tts/checkpoint' \
--transformer_step=160000 \
--config_path='config/fastspeech.yaml' \
```
or you can run the script file directly.
```bash
sh train.sh
```
If you want to train on multiple GPUs, you must set ``--use_data_parallel=1``, and then start training as follow:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3
python -m paddle.distributed.launch --selected_gpus=0,1,2,3 --log_dir ./mylog train.py \
--use_gpu=1 \
--use_data_parallel=1 \
--data_path=${DATAPATH} \
--transtts_path='../transformer_tts/checkpoint' \
--transformer_step=160000 \
--config_path='config/fastspeech.yaml' \
```

if you wish to resume from an exists model, please set ``--checkpoint_path`` and ``--fastspeech_step``

For more help on arguments: 
``python train.py --help``.

## Synthesis
After training the FastSpeech, audio can be synthesized with ``synthesis.py``.
```bash
python synthesis.py \
--use_gpu=1 \
--alpha=1.0 \
--checkpoint_path='checkpoint/' \
--fastspeech_step=112000 \
```

or you can run the script file directly.
```bash
sh synthesis.sh
```

For more help on arguments: 
``python synthesis.py --help``.