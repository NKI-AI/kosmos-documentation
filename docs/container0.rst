.. role:: raw-html-m2r(raw)
   :format: html


/*<![CDATA[*/ div.rbtoc1670582120051 {padding: 0px;} div.rbtoc1670582120051 ul {list-style: disc;margin-left: 0px;} div.rbtoc1670582120051 li {margin-left: 0px;padding-left: 0px;} /*]]>*/


* `1. From building the Dockerfile to using the singularity container <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-1.FrombuildingtheDockerfiletousingthesingularitycontainer>`_

  * `1. Build the docker container with docker build <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-1.Buildthedockercontainerwithdockerbuild>`_
  * `2. Save the docker container as an image <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-2.Savethedockercontainerasanimage>`_

    * `To test that the docker container is what we want, we can test it in the shell. <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-Totestthatthedockercontaineriswhatwewant,wecantestitintheshell.>`_
    * `To delete unnecessary docker images to reduce memory: <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-Todeleteunnecessarydockerimagestoreducememory:>`_

  * `3. Build singularity image from docker image <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-3.Buildsingularityimagefromdockerimage>`_
  * `4. Upload the singularity image to the server <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-4.Uploadthesingularityimagetotheserver>`_
  * `5. Using the singularity image <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-5.Usingthesingularityimage>`_

* `2. Dockerfile example with explanations <#CompleteInstructions:FromcreatingtheDockerfiletousingthesingularitycontainer-2.Dockerfileexamplewithexplanations>`_

1. From building the Dockerfile to using the singularity container
==================================================================

1. Build the docker container with docker build
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java

   docker build -t docker_name:version_name /output/path/ -f path_to/Dockerfile 
   specfici example:
   docker build -t drop_dlup:latest . -f docker/Dockerfile_orig


* 
  You can also give the Dockerfile another name, there is no extension

* 
  version name can be something like ‘1.0’ or also ‘latest’

2. Save the docker container as an image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* The container_id can be found by using **docker ps**

.. code-block:: java

   docker save container_id -o docker_image_name.tar
   spcific example:
   docker save a2fbf0a33294 -o docker_new26-12.tar

To test that the docker container is what we want, we can test it in the shell.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


* This way we can gradually add things and understand and see how the docker container changes.

.. code-block:: java

   docker run -ti image_id /bin/bash

To delete unnecessary docker images to reduce memory:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: java

   # to remove all dangling images
   docker image prune -a
   #to remove all stopped containers, networks that are not used by at least one container, dangling images, build cache
   docker system prune

3. Build singularity image from docker image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  The container_id can be found by using ``docker ps``

* 
  See also above the information on: Converting docker to singularity- Method 1: Docker is already built

.. code-block:: java

   singularity build singularity_image_name.sif docker-archive://docker_image_name.tar
   specific example:
   singularity build drop_new_20211226.sif docker-archive://docker_new26-12.tar

4. Upload the singularity image to the server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* This can take a long time (3 hours)

.. code-block:: java

    rsync -azv --progress=’info2’  drop_new_20211226.sif user_nki@rhpc-server_name:/path/to/singularity_image.sif

5. Using the singularity image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  A bash script can be made for launching the singularity image in a container and mapping the required GPUs to the singularity image.

* 
  With slurm, GPU server numbers are automatically 0-N where N are the number of GPUs requested. Previously the exact GPU numbers needed to be provided in the format ``"1,3,5"`` as an example for GPUs 1, 3 and 5.

* 
  Executing with sh is sufficient

* 
  You can bind the locations you need to access in the singularity container with the --bind flag:\ :raw-html-m2r:`<br>`
  --bind path/to/loc/on/server:path/to/loc/on/singularity, path/to/loc2/on/server:path/to/loc2/on/singularity

* 
  The bound locations have no space in between the items. There is a space after the --bind flag as this is followed by the singularity image location

.. code-block:: java

   #!/bin/bash
   read -p 'GPU indices to use (format example: "1,3,5"): ' GPU_IDX

   general:
   SINGULARITYENV_CUDA_VISIBLE_DEVICES="${GPU_IDX}" singularity shell --nv \
   --bind path/to/loc/on/server:path/to/loc/on/singularity,\
   path/to/loc2/on/server:path/to/loc2/on/singularity \
   /path/to/singularity_image.sif


   specific: 
   SINGULARITYENV_CUDA_VISIBLE_DEVICES="${GPU_IDX}" singularity shell --nv \
   --bind /mnt/archive/projectdata/drop:/mnt/archive/projectdata/drop,\
   /mnt/archive/data/pathology:/mnt/archive/data/pathology,\
   /processing/"$USER":/scratch \
   /mnt/archive/projectdata/drop/containers/drop_20210713.sif

2. Dockerfile example with explanations
=======================================

.. code-block:: py

   #install ubuntu base os with cuda and cudnn installed \
   # (for accesing the GPUs (cuda) and performing cuda-backed deep learning (cudnn))
   ARG CUDA="11.1"
   ARG CUDNN="8"
   FROM nvidia/cuda:${CUDA}-cudnn${CUDNN}-devel-ubuntu18.04

   # define username, pytorch and python version to use
   ARG CUDA
   ARG CUDNN
   ARG PYTORCH="1.9"
   ARG PYTHON="3.8"
   ARG UNAME="user"

   # Set cuda path environment variable with ENV (not export)
   ENV CUDA_PATH /usr/local/cuda
   # Define the architecture of our GPUs (rtx8000 = Turing, a6000 = Ampere) (for cudnn)
   ENV TORCH_CUDA_ARCH_LIST="Turing;Ampere" 
   # set cuda root environment variable 
   ENV CUDA_ROOT /usr/local/cuda/bin
   # set LD_LIBRARY_PATH environment variable tells Linux applications \
   # where to find shared libraries when they are located in a different directory \
   #from the directory that is specified in the header section of the program.
   ENV LD_LIBRARY_PATH /usr/local/nvidia/lib64

   #install dependencies for dlup
   #first run apt-get update
   #for testing the docker container you could also just install some necessary libraries
   #like nano and sudo 
   #potentially also install ssh for usage of debugger (tbc)

   RUN apt-get update && apt-get install -y libxrender1 build-essential sudo \
       autoconf automake libtool pkg-config libtiff-dev libopenjp2-7-dev libglib2.0-dev \
       libxml++2.6-dev libsqlite3-dev libgdk-pixbuf2.0-dev libgl1-mesa-glx git wget rsync \
       fftw3-dev liblapacke-dev libpng-dev libopenblas-dev libxext-dev jq sudo \
       libfreetype6 libfreetype6-dev \
       # Purge pixman and cairo to be sure they are removed (reducing container size)
       && apt-get remove libpixman-1-dev libcairo2-dev \
       && apt-get purge libpixman-1-dev libcairo2-dev \
       && apt-get autoremove && apt-get clean \
       && rm -rf /var/lib/apt/lists/*

   # Install pixman 0.40, as Ubuntu repository holds a version with a bug which can cause difficulties reading thumbnails
   RUN cd /tmp \
       && wget https://www.cairographics.org/releases/pixman-0.40.0.tar.gz \
       && tar xvf pixman-0.40.0.tar.gz && rm pixman-0.40.0.tar.gz && cd pixman-0.40.0 \
       && ./configure && make -j$BUILD_WORKERS && make install \
       && cd /tmp && rm -rf pixman-0.40.0

   # Install cairo 1.16
   RUN cd /tmp \
       && wget https://www.cairographics.org/releases/cairo-1.16.0.tar.xz \
       && tar xvf cairo-1.16.0.tar.xz && rm cairo-1.16.0.tar.xz && cd cairo-1.16.0 \
       && ./configure && make -j$BUILD_WORKERS && make install \
       && cd /tmp && rm -rf cairo-1.16.0

   # Install OpenSlide for NKI-AI repository.
   RUN git clone https://github.com/NKI-AI/openslide.git /tmp/openslide \
       && cd /tmp/openslide \
       && autoreconf -i \
       && ./configure && make -j$BUILD_WORKERS && make install && ldconfig \
       && cd /tmp && rm -rf openslide

   # Make a user (we are currently root user) 
   # disabledd-password means that no password can be set for user
   # gecos is also a sort of linux password. gecos field exists in /etc/passwd file on unix
   # we set home dir to /users (otherwise it would be automatically set to /home). This is to prevent issues with singularity
   # lastly the name of the user to add is given - in our case $UNAME
   # user needs to be added as sudoer by writing to file /etc/sudoers

   RUN mkdir /users && echo $UNAME \
       && adduser --disabled-password --gecos '' --home /users/$UNAME $UNAME \
       && adduser $UNAME sudo \
       && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

   #change from root user to the new user and set new working directory 
   USER $UNAME
   WORKDIR /users/$UNAME

   #install miniconda 
   RUN cd /tmp && wget -q https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh \
       && bash Miniconda3-latest-Linux-x86_64.sh -b \
       && rm Miniconda3-latest-Linux-x86_64.sh
   # declare environment variable PATH. Set miniconda as first path variable \
   #(to first check this location when conda is executed), \
   #then the old path vars and then cuda root 
   ENV PATH "/users/$UNAME/miniconda3/bin:$PATH:$CUDA_ROOT"


   # Setup python packages
   RUN conda update -n base conda -yq \
       && conda install python=${PYTHON} \
       && conda install astunparse ninja setuptools cmake future requests dataclasses \
       && conda install pyyaml mkl mkl-include setuptools cmake cffi typing boost \
       && conda install tqdm jupyter matplotlib scikit-image pandas joblib -yq \
       && conda install typing_extensions \
       && conda clean -ya \
       && python -m pip install numpy==1.20 tifftools -q \
       && conda install pytorch torchvision cudatoolkit=${CUDA} -c pytorch -c nvidia \
       && conda install pytorch-lightning -c conda-forge \
       && python -m pip install pycocotools tensorboardX tabulate -q \
       # Install openslide-python from NKI-AI
       && python -m pip install git+https://github.com/openslide/openslide-python.git

   # Install jupyter config to be able to run in the docker environment
   RUN jupyter notebook --generate-config
   ENV CONFIG_PATH "/users/$UNAME/.jupyter/jupyter_notebook_config.py"
   COPY "docker/jupyter_notebook_config.py" ${CONFIG_PATH}

   # install detectron2 in /users/user/. Miniconda is also there.
   WORKDIR /users/$UNAME
   RUN python -m pip install 'git+https://github.com/facebookresearch/fvcore'
   RUN git clone https://github.com/facebookresearch/detectron2
   ENV FORCE_CUDA="1"
   RUN python -m pip install -e detectron2
   ENV FVCORE_CACHE="/tmp"

   # Copy drop files from local machine repo (we are running this from \
   #the relevant folder "DROP") into the docker container into /drop
   COPY [".", "/drop"]

   #change to root user to have full permissions
   USER root
   # Alternative: we can give permissions to our user in /drop  with:
   COPY --chown=$UNAME:$UNAME . /drop 

   ## install dlup from our local copy of dlup, the -e flag makes the repo editable 
   WORKDIR /drop/third_party/dlup
   RUN python -m pip install -e .


   ### install deformable detr
   ##WORKDIR /drop/third_party/Deformable-DETR/models/ops
   ##RUN sh ./make.sh

   USER $UNAME

   ## Verify installation
   RUN python -c 'import openslide'
   RUN python -c 'import dlup'

   # Provide an open entrypoint for the docker
   ENTRYPOINT $0 $@
