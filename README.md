# NeRF_Holo
* NeRF+Hololens

## Hardware

* Prosessor 12th Gen Intel® Core™ i5-12400F × 12
* Graphics NVIDIA Corporation GA104 [GeForce RTX 3070] 8GB

## Software

### Ubuntu 22.04
* Nvidia Driver Version: 515
* CUDA 11.2
* Clone this repo by `git clone --recursive https://github.com/CrystalWlz/nerf_holo`
* Python>=3.6 (installation via [anaconda](https://www.anaconda.com/distribution/) is recommended, use `conda create -n nerf_holo python=3.6` to create a conda environment and activate it by `conda activate nerf_pl`)
* Python libraries
    * Install core requirements by `pip install -r requirements.txt`
    * Install `torchsearchsorted` by `cd torchsearchsorted` 
    * then `sudo /home/wlz/anaconda3/envs/nerf_holo/bin/python -m pip install .`
      * (sodo is followed by `whereis python`)
    
# :key: Training

Please see each subsection for training on different datasets. Available training datasets:

* [Blender](#blender) (Realistic Synthetic 360)
* [LLFF](#llff) (Real Forward-Facing)
* [Your own data](#your-own-data) (Forward-Facing/360 inward-facing)

## Blender

   
### Data download

Download `nerf_synthetic.zip` from [here](https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1)

### Training model

Run (example)
```
python train.py \
   --dataset_name blender \
   --root_dir /home/data3/wlz/nerf-master/data/nerf_synthetic/lego \
   --N_importance 64 --img_wh 400 400 --noise_std 0 \
   --num_epochs 16 --batch_size 1024 \
   --optimizer adam --lr 5e-4 \
   --lr_scheduler steplr --decay_step 2 4 8 --decay_gamma 0.5 \
   --exp_name exp
```

These parameters are chosen to best mimic the training settings in the original repo. See [opt.py](opt.py) for all configurations.

NOTE: the above configuration doesn't work for some scenes like `drums`, `ship`. In that case, consider increasing the `batch_size` or change the `optimizer` to `radam`. I managed to train on all scenes with these modifications.

You can monitor the training process by `tensorboard --logdir logs/` and go to `localhost:6006` in your browser.


## LLFF

   
### Data download

Download `nerf_llff_data.zip` from [here](https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1)

### Training model

Run (example)
```
python train.py \
   --dataset_name llff \
   --root_dir /home/data3/wlz/nerf-master/data/nerf_llff_data/fern \
   --N_importance 64 --img_wh 504 378 \
   --num_epochs 30 --batch_size 1024 \
   --optimizer adam --lr 5e-4 \
   --lr_scheduler steplr --decay_step 10 20 --decay_gamma 0.5 \
   --exp_name exp
```

These parameters are chosen to best mimic the training settings in the original repo. See [opt.py](opt.py) for all configurations.

You can monitor the training process by `tensorboard --logdir logs/` and go to `localhost:6006` in your browser.

## Your own data

   
1. Install [COLMAP](https://github.com/colmap/colmap) following [installation guide](https://colmap.github.io/install.html)
```
conda create -n colmap
conda activate colmap
sudo apt-get install \
    git \
    cmake \
    ninja-build \
    build-essential \
    libboost-program-options-dev \
    libboost-filesystem-dev \
    libboost-graph-dev \
    libboost-system-dev \
    libboost-test-dev \
    libeigen3-dev \
    libflann-dev \
    libfreeimage-dev \
    libmetis-dev \
    libgoogle-glog-dev \
    libgflags-dev \
    libsqlite3-dev \
    libglew-dev \
    qtbase5-dev \
    libqt5opengl5-dev \
    libcgal-dev \
    libceres-dev
```
Under Ubuntu 22.04, there is a problem when compiling with Ubuntu’s default CUDA package and GCC, and you must compile against GCC 10:
```
sudo apt-get install gcc-10 g++-10
export CC=/usr/bin/gcc-10
export CXX=/usr/bin/g++-10
export CUDAHOSTCXX=/usr/bin/g++-10
```
add `set(CMAKE_CUDA_ARCHITECTURES 70)` in the CMakeLists.txt file
```
git clone https://github.com/colmap/colmap.git
cd colmap
git checkout dev
mkdir build
cd build
cmake .. -GNinja
ninja
sudo ninja install
```
2. Prepare your images in a folder (around 20 to 30 for forward facing, and 40 to 50 for 360 inward-facing)
3. Clone [LLFF](https://github.com/Fyusion/LLFF) and run `python imgs2poses.py $your-images-folder`
4. Train the model using the same command as in [LLFF](#llff). If the scene is captured in a 360 inward-facing manner, add `--spheric` argument.
(example)
```
python train.py \
   --dataset_name llff \
   --root_dir test2 \
   --N_importance 64 --img_wh 800 800 \
   --num_epochs 30 --batch_size 2048 \
   --optimizer adam --lr 1e-3 \
   --lr_scheduler steplr --decay_step 10 20 --decay_gamma 0.5 \
   --exp_name exp \
   --spheric
```
