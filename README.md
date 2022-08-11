# docker_image_falling_fluid_film

All the instructions below are for Linux (typically, a modern Ubuntu). If you are on windows, you will have to adapt the instructions by yourself.

## Getting the container image

The docker image for the falling fluid film repo https://github.com/vbelus/falling-liquid-film-drl , stored with git-lfs.

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

