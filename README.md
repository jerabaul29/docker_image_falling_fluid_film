# docker_image_falling_fluid_film

All the instructions below are for Linux (typically, a modern Ubuntu). If you are on windows, you will have to adapt the instructions by yourself.

## What is contained in this repository?

This repository contains the docker container of the paper ```Exploiting locality and physical invariants to design effective Deep Reinforcement Learning control of the unstable falling liquid film```, Belus et. al., published at https://aip.scitation.org/doi/10.1063/1.5132378 .

![no-control](https://media.giphy.com/media/cO3IdQaGRyK0BYAslz/giphy.gif)
![control](https://media.giphy.com/media/dY0qmKEb5bXhf3gHvs/giphy.gif)

If you find this work useful and / or use it in your own research, please cite our works:

```
V. Belus, J. Rabault, J. Viquerat, Z. Che, E. Hachem, and U. Reglade
Exploiting locality and physical invariants to design effective Deep
Reinforcement Learning control of the unstable falling liquid film,
ArXiv (2019)
```

This is mostly a mirror of the initial repos at https://github.com/vbelus/falling-liquid-film-drl and https://github.com/vbelus/drl-fluid-film-notebook , with in addition the container image being loaded on the repo using git-lfs.

## Getting the container image

The docker image for the falling fluid film repo https://github.com/vbelus/falling-liquid-film-drl and the jupyter notebooks https://github.com/vbelus/drl-fluid-film-notebook , stored with git-lfs. The container has all the content of the corresponding repos already built in, and should be fully ready to use.

The container image is available in the ```container``` folder. It is split in segments of max size 500M and committed with git-lfs (in theory git-lfs could commit the whole container, but I encountered issues with large files even in git-lfs in the past, see discussion at **A note here** in the readme at https://github.com/jerabaul29/Cylinder2DFlowControlDRLParallel).

To download the segments, either clone the repository, or if this fails or times out, download each of the segment "by hand" (click on each file in https://github.com/jerabaul29/docker_image_falling_fluid_film/tree/main/container and use the "download" button).

Splitting the container image was performed with:

```
$ split -b 500M falling-fluid-film-drl-rebuilt__v2022_08_11.tar falling-fluid-film-drl-rebuilt__v2022_08_11.tar__part.
```

To re-create the container you should do:

```
$ cat falling-fluid-film-drl-rebuilt__v2022_08_11.tar__part.?? > falling-fluid-film-drl-rebuilt__v2022_08_11.tar
```

Once the reconstruction of the container is done, you should check its hash to verify its integrity:

```
$ sha256sum falling-fluid-film-drl-rebuilt__v2022_08_11.tar 
d30eda25473dd5d219dabeca1f8e540d2b6bcbd293b54b3fba5ee393f7c15d10  falling-fluid-film-drl-rebuilt__v2022_08_11.tar
```

## Using the container image

To use the container image:

- load the container image archive into docker

```
$ sudo docker image load < falling-fluid-film-drl-rebuilt__v2022_08_11.tar
```

- check that the image has well been loaded:

```
$ sudo docker image ls 
REPOSITORY                       TAG           IMAGE ID       CREATED         SIZE
falling-fluid-film-drl-rebuilt   v2022_08_11   751e15154f9d   4 minutes ago   2.48GB
```

- spin a new interactive container from the image, binding the TCP/UDP port:

```
~$ sudo docker run -it -p 8888:8888 falling-fluid-film-drl-rebuilt:v2022_08_11
```

- start a jupyter notebook inside the container:

```
root@2200b9f99d69:~# cd ~/drl-fluid-film-notebook/falling-liquid-film-drl/drl_fluid_film_python3/gym-film
root@2200b9f99d69:~/drl-fluid-film-notebook/falling-liquid-film-drl/drl_fluid_film_python3/gym-film# jupyter notebook --ip 0.0.0.0 --no-browser --allow-root
```

- open the link in your browser.

## Following the "tutorials"

There is a small "tutorials" following the M1, M2, M3 methods reported in the paper: see the ipynb jupyter notebooks **Introduction-and-method-M1.ipynb**, **Method-M2.ipynb**, **Method-M3.ipynb** that should be visible in the jupyter file listing.

## Copy paste of manual from https://github.com/vbelus/falling-liquid-film-drl

The paths are a bit messed up in the new image, and I recommend that you follow the jupyter notebooks to get understanding of the code, but reproducing some manual information for the sake of completeness:

The code is in `drl_fluid_film_python3/gym-film`. You find the following files:
- `param.py` contains the parameters that your next training will use (all the parameters of the project are centralized here)
- `train.py` is the script you will use to train or render a model

In `gym_film`, you will find the following directories:
- `envs`, where our basic environment class `film_env.py` is defined. It is a custom gym environment,and it interacts with the C++ simulation that is defined in `simulation_solver`.
- `model`, where the script to retrieve models is defined. These models are imported from the library stable_baselines, and only the PPO2 implementation has been used in the paper.
- `results`, where previously trained models are stored along with the parameters used for the training, and tensorboard logs to have insights on how the model performs during training

### Execute the main script

The basic command is `python3 train.py` but you will need to add some flags :
- `--train` or `-t` to train your model
- `--render` or `-r` if you want to render the simulation

If you use this flag with `-t`, it will render during the training as well. You can change how often the rendering is done with the `RENDER_PERIOD` parameter
- `--training_name` or `-tn` is the name of the training, default value is "test"
- `--load` or `-l` followed by an integer `n` to load a trained model. This will look for a model trained for n episodes under the training name you specified
- `--load_best` or `-lb` will load the "best" model under the training name you specified, "best" meaning the model trained on k episodes, where the maximum episode reward of the entire training was obtained during episode k
- `--port` or `-p`, followed by an integer between 1024 and 49151, which will be the port used when using the multi-environment method (Method M3 in the paper).

If you want to train a model with the default parameters and render it, you can do it with `python train.py --train --render --training_name my-first-training`

### Change the parameters of the training
All the parameters relevant to the training and simulation are in `param.py`. You will have to change the parameters in this file.

We will use the following syntax: `parameter` (type, typical low value - typical high value)

Some important parameters here are :
- The number of jets: `n_jets` (int, 1 - 20)
- The position of the jets: `jets_position` (array) OR `position_first_jet` (float) and `space_between_jets` (float, 5 - 20)
- The maximum power of the jets: `JET_MAX_POWER` (float, 1 - 15) and their width: `JET_WIDTH` (float, 2 - 10)
-> As it is done, the action of the agent is always between `-1` and `1` and is later normalized and multiplied by `JET_MAX_POWER` to be applied in the simulation. The default values are letting the jets modify q with the same amplitude as the waves naturally forming, so it should be enough for control.
- The size of the observation `size_obs` (float, 10 - 40), input of the agent, and the size of the observation on which the reward is calculated `size_obs_to_reward` (float, 5 - 20)
- The non-dimensional duration of one episode `simulation_time` (float, 5 - 20)
- The duration of the simulation before we begin any training `initial_waiting_time` (float). Default value is 200, letting the simulation get to a converged state before we do any training (the state is stored and not computed each time)
- `simulation_step_time` is the non-dimensional duration of one step of the simulation. In one such step, we do `n_step` steps of the numerical scheme
- Whether to render the simulation with matplotlib with `render` (bool). If True, render it every `RENDER_PERIOD` (int) environment steps

