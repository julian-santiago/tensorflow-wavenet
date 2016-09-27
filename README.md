# A TensorFlow implementation of DeepMind's WaveNet paper

[![Build Status](https://travis-ci.org/ibab/tensorflow-wavenet.svg?branch=master)](https://travis-ci.org/ibab/tensorflow-wavenet)

This is a TensorFlow implementation of the [WaveNet generative neural
network architecture](https://deepmind.com/blog/wavenet-generative-model-raw-audio/) for audio generation.

<table style="border-collapse: collapse">
<tr>
<td>
<p>
The WaveNet neural network architecture directly generates a raw audio waveform,
showing excellent results in text-to-speech and general audio generation (see the
DeepMind blog post and paper for details).
</p>
<p>
The network models the conditional probability to generate the next
sample in the audio waveform, given all previous samples and possibly
additional parameters.
</p>
<p>
After an audio preprocessing step, the input waveform is quantized to a fixed integer range.
The integer amplitudes are then one-hot encoded to produce a tensor of shape <code>(num_samples, num_channels)</code>.
</p>
<p>
A convolutional layer that only accesses the current and previous inputs then reduces the channel dimension.
</p>
<p>
The core of the network is constructed as a stack of <em>causal dilated layers</em>, each of which is a
dilated convolution (convolution with holes), which only accesses the current and past audio samples.
</p>
<p>
The outputs of all layers are combined and extended back to the original number
of channels by a series of dense postprocessing layers, followed by a softmax
function to transform the outputs into a categorical distribution.
</p>
<p>
The loss function is the cross-entropy between the output for each timestep and the input at the next timestep.
</p>
<p>
In this repository, the network implementation can be found in <a href="./wavenet.py">wavenet.py</a>.
</p>
</td>
<td width="300">
<img src="images/network.png" width="300"></img>
</td>
</tr>
</table>

## Requirements

TensorFlow needs to be installed before running the training script.
TensorFlow 0.10 and the current `master` version are supported.

In addition, [librosa](https://github.com/librosa/librosa) must be installed for reading and writing audio.

To install the required python packages (except TensorFlow), run
```bash
pip install -r requirements.txt
```

## Training the network

You can use any corpus containing `.wav` files.
We've mainly used the [VCTK corpus](http://homepages.inf.ed.ac.uk/jyamagis/page3/page58/page58.html) (around 10.4GB, [Alternative host](http://www.udialogue.org/download/cstr-vctk-corpus.html)) so far.

In order to train the network, execute
```bash
python train.py --data_dir=corpus
```
to train the network, where `corpus` is a directory containing `.wav` files.
The script will recursively collect all `.wav` files in the directory.

You can see documentation on each of the training settings by running
```bash
python train.py --help
```

You can find the configuration of the model parameters in [`wavenet_params.json`](./wavenet_params.json).
These need to stay the same between training and generation.

## Generating audio

[Example output](https://soundcloud.com/user-731806733/speaker-p280-from-vctk-corpus-1)
generated by @jyegerlehner based on speaker 280 from the VCTK corpus.

You can use the `generate.py` script to generate audio using a previously trained model.

Run
```
python generate.py --samples 16000 model.ckpt-1000
```
where `model.ckpt-1000` needs to be a previously saved model.
You can find these in the `logdir`.
The `--samples` parameter specifies how many audio samples you would like to generate (16000 corresponds to 1 second by default).

The generated waveform can be played back using TensorBoard, or stored as a
`.wav` file by using the `--wav_out_path` parameter:
```
python generate.py --wav_out_path=generated.wav --samples 16000 model.ckpt-1000
```

Passing `--save_every` in addition to `--wav_out_path` will save the in-progress wav file every n samples.
```
python generate.py --wav_out_path=generated.wav --save_every 2000 --samples 16000 model.ckpt-1000
```

Fast generation is enabled by default. It uses the implementation from the [Fast Wavenet](https://github.com/tomlepaine/fast-wavenet) repository. You can follow the link for an explanation of how it works. This reduces the time needed to generate samples to a few minutes.

To disable fast generation:
```
python generate.py --samples 16000 model.ckpt-1000 --fast_generation=false
```


## Missing features

Currently, there is no conditioning on extra information like the speaker ID.

