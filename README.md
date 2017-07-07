## End-to-End Learning in a Driving Simulation

#### Abstract

This project implements NVIDIA's 2016 paper [End to End Learning for Self-Driving Cars](https://arxiv.org/pdf/1604.07316.pdf) in which 
Bojarski et al. describe a convolutional neural network architecture used to infer vehicle control inputs given a forward 
facing vehicle camera stream. I apply the techniques in this paper to a [driving simulation game](https://github.com/udacity/self-driving-car-sim) 
: a player inputs controls to the simulator's virtual vehicle, and the mapping between the 
scene in front of the car and the player's controls is modeled by a neural network to "autonomously drive" the virtual car. 

![nvidia demo](https://github.com/alexhagiopol/end_to_end_learning/blob/master/figures/nvidia_demo.gif)
![manual_diving_example](https://github.com/alexhagiopol/end_to_end_learning/blob/master/figures/manual_driving_example.gif)

#### Installation

This procedure was tested on Ubuntu 16.04 (Xenial Xerus) and Mac OS X 10.11.6 (El Capitan). GPU-accelerated training is supported on Ubuntu only.
Prerequisites: Install Python package dependencies using [my instructions.](https://github.com/alexhagiopol/deep_learning_packages) Then, activate the environment:

    source activate deep-learning
    
Acquiring the driving simulator and example dataset with camera stream images and steering control inputs by a human player:

    wget https://d17h27t6h515a5.cloudfront.net/topher/2017/February/58ae46bb_linux-sim/linux-sim.zip  # linux version of simulator
    unzip linux-sim.zip
    rm -rf linux-sim.zip
    wget -O udacity_dataset.tar.gz "https://www.dropbox.com/s/s4q0y0zq8xrxopi/udacity_dataset.tar.gz?dl=1"  # example dataset
    tar -xvzf udacity_dataset.tar.gz
    rm -rf udacity_dataset.tar.gz

Optional, but recommended on Ubuntu: Install support for NVIDIA GPU acceleration with CUDA v8.0 and cuDNN v5.1:

    wget -O cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb "https://www.dropbox.com/s/08ufs95pw94gu37/cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb?dl=1"
    sudo dpkg -i cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb
    sudo apt-get update
    sudo apt-get install cuda
    wget -O cudnn-8.0-linux-x64-v5.1.tgz "https://www.dropbox.com/s/9uah11bwtsx5fwl/cudnn-8.0-linux-x64-v5.1.tgz?dl=1"
    tar -xvzf cudnn-8.0-linux-x64-v5.1.tgz
    cd cuda/lib64
    export LD_LIBRARY_PATH=`pwd`:$LD_LIBRARY_PATH  # consider adding this to your ~/.bashrc
    cd ..
    export CUDA_HOME=`pwd`  # consider adding this to your ~/.bashrc
    sudo apt-get install libcupti-dev
    pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.1.0-cp35-cp35m-linux_x86_64.whl

#### Execution
### Model Training with `train_network.py`

This file is the core of this project. Use it to train the NVIDIA model on a driving dataset. Use `python train_network.py -h`
to see documentation information. Example command:
    
    python train_network.py -d udacity_dataset -m model.h5 -c 3000

The program assumes the following: 
1. The required `-d` argument is a path to the dataset directory.
    - The dataset directory contains a single .csv file with a driving log along with many .jpg images recorded from the player driving sequence associated with the log.
    - The image names in the driving log match the images names in the dataset directory. I.e. image names such as `dir/IMG/center_2016_12_01_13_30_48_287.jpg`
should be replaced in the driving log with simply `center_2016_12_01_13_30_48_287.jpg`. Use scripts such as `scripts/combine_datasets.py` and `scripts/fix_filenames.py`
to prepare your dataset to work with this pipeline. 
    - It's recommened that you start with the example dataset provided in the installation instructions.
2. The required `-m` argument is the name of the Keras model to export.
3. The optional `-c` argument is the CPU batch size i.e. the number of measurement groups that will fit in system RAM on your machine. The default value is 1000.
4. The optional `-g` argument is the GPU batch size i.e. the number of training images and training labels that will fit in VRAM on your machine.
5. The optional `-r` argument specifies whether or not to randomize the row order of the driving log.

#### Driving Autonomously with `drive.py` and the Unity Simulator

Use `drive.py` in conjunction with the simulator to autonomously drive a virtual car according to a neural network model.
 Usage of `drive.py` requires you to have saved the trained model as an h5 file, e.g. `model.h5`. This type of file is created 
by running `train_network.py` as described above. Once the model has been saved, it can be used with drive.py using this commands:

    python drive.py model.h5

This command prepares causes the driving program to start and wait for the simulator to be initialized. On Linux, initialize the 
simulator with the following (procedure is very similar on Mac):

    cd linux_sim
    ./linux_sim.x86_64

Select the lowest resolution and fastest graphical settings. Afterward select "autonomous mode" followed by the desired track. The car 
will now drive on its own. This procedure loads the trained model and uses the model to make predictions on individual images in real-time 
and send the predicted angle back to the simulator server via a websocket connection.

##### Saving a video of the autonomous agent

```sh
python drive.py model.h5 run1
```

The fourth argument, `run1`, is the directory in which to save the images seen by the agent. If the directory already exists, it'll be overwritten.

```sh
ls run1

[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_424.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_451.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_477.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_528.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_573.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_618.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_697.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_723.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_749.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_817.jpg
...
```

The image file name is a timestamp of when the image was seen. This information is used by `video.py` to create a chronological video of the agent driving.

##### `video.py`

```sh
python video.py run1
```

Creates a video based on images found in the `run1` directory. The name of the video will be the name of the directory followed by `'.mp4'`, so, in this case the video will be `run1.mp4`.

Optionally, one can specify the FPS (frames per second) of the video:

```sh
python video.py run1 --fps 48
```

Will run the video at 48 FPS. The default FPS is 60.
