====================================================================
Using Singularity to create the most comfortable working environment
====================================================================

Although the preferred way of any student to run things on the cluster is through Anaconda environments that are well groomed and setup, with loading the correct modules using spack,
it often comes to a moment in your research career when it is impossible to run everything with the existing tools on the cluster.

A sure way to get all you need is to use a container. However, docker is not suited for cluster environments, as it gives too much access to the users and risks create
a massive caos. Therefore, the cluster allows for Singularity to be used instead.

Containers are an incredible resoruce that basically allows you to have your own operating system on top of the hardware provided by the cluster. Everything from 
drivers to software can be used, without any worry about sudo rights or too much cleanliness because you are the only one using that container, and once you close it, 
it can do no harm to the rest of the cluster.

There are several ways to use singularity. One of these is to also just take your Docker image and convert it. However, I found that this created some bugs and it is a VERY VERY slow process
that I would rather avoid.

Instead, you ca  build your singularity image from scratch, and it's just as easy as using a Docker image. 

First install Singularity on your own machine, you will need `sudo` rights to build your Singularity image. Follow very carefully the steps at the following link (think step=by-step): https://docs.sylabs.io/guides/3.0/user-guide/installation.html .

In order to build a Singularity image you will need a Singularity definition file.
Here is one of these files, which builds on top of nvidia CUDA docker using CUDA 11.6.2 and CUDNN 8 on a Ubuntu 20.04:

There are 5 sections

.. code-block:: bash

   Bootstrap: docker  
   From: nvidia/cuda:11.6.2-cudnn8-devel-ubuntu20.04


This indicates that it should go to the docker hub and pull the container with tag `nvidia/cuda:11.6.2-cudnn8-devel-ubuntu20.04`. You can find the tags very easily just by googling (or go here https://hub.docker.com/r/nvidia/cuda/tags). This base version has cuda already installed and the correct drivers running on Ubuntu 20.04. 

.. code-block:: bash

   %labels  
      MANTAINER Nikita Moriakov, Samuele Papa


These are the people that maintain this singularity definition file, always useful to have but not essential for the thing to run.

.. code-block:: bash

   %environment  
       export CUDA_PATH=/usr/local/cuda  
       export CUDA_HOME=/usr/local/cuda  
       export CUDA_ROOT=/usr/local/cuda/bin  
       export LD_LIBRARY_PATH=/usr/local/nvidia/lib64  
       export PATH=$PATH:/usr/local/cuda/bin:/opt/miniconda3/bin


These are the environment variables that we want to set, to make sure that everything runs smoothly when we run our scripts from within this singularity image.

.. code-block:: bash

   %files  
       ./tomovis_project/* /opt/tomovis_project/  
       ./tomo_projector_library/* /opt/tomo_projector_library/  
       ./parametric_dataset_project/* /opt/parametric_dataset_project/  
       ./msd_pytorch/* /opt/msd_pytorch/


These are the local folders that we want to bind during building, as we cannot do this dynamically.

.. code-block:: bash

   %post  
      # Downloads the latest package lists (important).  
      apt-get update -y  

      #python3-dev \  
      #python3-tk \  
      #python3-pip \  
      #python3-setuptools \  

      # Install python and other tools  
      # Non-interactive is used to ensure prompts are omitted.  
      DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \  
      systemd \  
      libxext6 \  
      libsm6 \  
      bzip2 \  
      libxrender1 \  
      libgl1-mesa-glx \  
      build-essential \  
      automake \  
      libboost-all-dev \  
      git \  
      openssh-server \  
      wget \  
      nano \  
      libtool \  
      rsync  

      # Reduce image size  
      rm -rf /var/lib/apt/lists/*  

       # Miniconda  
       wget -q https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh  
       bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/miniconda3  
       /opt/miniconda3/bin/conda update conda && /opt/miniconda3/bin/conda update --all  
       # echo $PATH  
       PATH=$PATH:/usr/local/cuda/bin:/opt/miniconda3/bin  
       export PATH  
       conda install python=3.9  

      # Update pip  
      # python3 -m pip install --upgrade pip  

      # Install python libraries  
      conda install pytorch=1.12.1 torchvision=0.13.1 cudatoolkit=11.6 pytorch-lightning -c conda-forge -c pytorch  
      pip install numpy pyyaml mkl mkl-include setuptools==59.5.0 cmake cffi typing boost scipy pandas cython matplotlib tqdm pillow scikit-learn scikit-image==0.18.3 hydra-core einops h5py wandb deepdiff black isort dominate visdom runstats tb-nightly yacs xarray future packaging pytest coverage coveralls easydict tifffile demandimport future notebook pydicom  
      # Make directories  
      mkdir /opt/ITK  
      mkdir /opt/RTK  
      cd /opt  
      wget -q https://github.com/InsightSoftwareConsortium/ITK/releases/download/v5.3.0/InsightToolkit-5.3.0.tar.gz  
      tar -xzf InsightToolkit-5.3.0.tar.gz  
      mv InsightToolkit-5.3.0/* ITK/  
      wget -q https://github.com/RTKConsortium/RTK/archive/refs/tags/v2.4.1.tar.gz  
      tar -xzf v2.4.1.tar.gz -C RTK --strip-components 1  

      cd /opt/ITK  
      mkdir build  
      cd build  
      cmake -DITK_WRAP_PYTHON=TRUE ..  
      make -j 8  

      CUDAARCHS='80;86'  
      export CUDAARCHS  
      cd /opt/RTK  
      mkdir build  
      cd build  
      cmake -DCUDAARCHS="80;86" -DRTK_USE_CUDA=TRUE -DITK_DIR=/opt/ITK/build ..  
      make -j 8  

      cp /opt/ITK/build/Wrapping/Generators/Python/WrapITK.pth /opt/miniconda3/lib/python3.9/site-packages/WrapITK.pth  

      mkdir /code  
      mkdir /data  

      # Python module  
      cd /opt/msd_pytorch  
      TORCH_CUDA_ARCH_LIST="7.5 8.0 8.6" pip install -e .[dev]  
      cd /opt/tomo_projector_library/tomo_projector_installer  
      TORCH_CUDA_ARCH_LIST="7.5 8.0 8.6" python setup.py install  
      cd /opt/tomo_projector_library  
      python -m pip install -e .  
      cd /opt/tomovis_project  
      python -m pip install -e .  
      cd /opt/parametric_dataset_project  
      python -m pip install -e .

This is where all the things get installed. Notice how we are installing also basic `apt` packages, setting environment variables, and everything else we would normally do when using `bash` and preparing our machine, because that's exactly what we are doing. We are basically creating a whole new machine where to run our code.
We download miniconda, install it, setup the `PATH`, install python and all the other packages we might need. After that, we also configure and compile two whole packages from scratch
