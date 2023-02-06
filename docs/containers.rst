.. _containers:

==========
Containers
==========

**THIS DOCUMENT IS OUTDATED AND REQUIRES A MAJOR REVISION. DO NOT FOLLOW THESE INSTRUCTIONS**

The preferred way of executing programs on the RHPC cluster is through the use of containers. Containers enable you to package up software (code and dependencies) in a portable and reproducible fashion.

We support several container platforms: Singularity, Enroot (experimental) and Docker. The use of docker requires specific privileges, and Singularity is recommended. Singularity and Enroot containers can also be run on Surf’s Lisa cluster.


Singularity
-----------

Using Singularity you can package your complete code and dependencies in one image (a single file). In contrast to Docker singularity containers are unprivileged (they do not require root permissions).

A Singularity recipe (a file describing how the container should be built) can be written, but Docker containers can readily converted to Singularity containers.

Converting Docker to Singularity
--------------------------------

There are several ways to convert your Docker into a Singularity container.  Check the `Singularity docs <https://singularity-userdoc.readthedocs.io/en/latest/troubleshooting.html>`_ for further reference beyond the summary belo.

Method 1: Docker container is already built
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A convenient method to convert an existing docker container into a singularity container is:


# Find the image ID of your container using ``docker images``\ , for instance ``ef70ddca9b03``.

# Dump the container to disk ``docker save ef70ddca9b03 -o container_name.tar``. Consider a descriptive name for the container, for ``instance code_YYYYMMDD.tar``. If you do not have singularity installed you can upload this to the RHPC, otherwise you can convert it on your own machine.

# Convert to singularity using ``singularity build code_YYYYMMDD.sif docker-archive://code_YYYYMMDD.tar``. This will create an image ``code_YYYYMMDD.sif``

# Note that the ``docker-archive://`` is required even though you point to a local file, see `this page <https://sylabs.io/guides/3.1/user-guide/singularity_and_docker.html>`_

# Then it’s explicit that it won’t load anything from dockerhub, and sudo is not required.

# Note that if you do this on LISA, the process will probably be killed if you run it on the login node since it’s too compute heavy and/or takes too long. Be sure to request a node to perform this on, or do it on one of the NKI servers.


# If you get a “no space left on device” error, this can be resolved by setting a different cache dir that does have enough space, like

.. code-block:: shell

    SINGULARITY_TMPDIR=/scratch/tmp SINGULARITY_CACHEDIR=/scratch/cache singularity build <rest of options>

# If you want to do this locally on MacOS or Windows, you need to use a VM that runs singularity.


# Install the required `software <https://sylabs.io/guides/3.0/user-guide/installation.html#mac>`_

# After you make the ``vm-singularity`` directory, place your ``.tar`` in this directory. This is the directory that will be mounted and mapped to ``/vagrant`` in the container

# If your image is large , the VM might not have enough disk space (20480MB standard, so if your image is much larger than 20GB there might be issues).

# Easiest is likely to increase this by following the ``vagrant-disksize`` plugin route from `https://askubuntu.com/questions/317338/how-can-i-increase-disk-size-on-a-vagrant-vm <https://askubuntu.com/questions/317338/how-can-i-increase-disk-size-on-a-vagrant-vm>`_

# You also want to change the ``SINGULARITY_TMPDIR`` and ``SINGULARITY_CACHEDIR`` and ``SINGULARITY_LOCALCACHEDIR``\ directories, since this is where the final image is intermediately saved. That is: ``export THE_VAR=/vagrant``\ , since ``/vagrant`` is the the mount of your local disk. see `https://github.com/hpcng/singularity/issues/1052 <https://github.com/hpcng/singularity/issues/1052>`_ and `https://sylabs.io/guides/3.5/user-guide/build\_env.html#temporary-folders <https://sylabs.io/guides/3.5/user-guide/build_env.html#temporary-folders>`_

# Then, run

.. code-block:: bash

    export VM=sylabs/singularity-3.0-ubuntu-bionic64 && \
    vagrant init $VM && \
    vagrant up && \
    vagrant ssh

# Run the singularity converting line as stated above. Note that the ``vm-vagrant`` dir on your local machine is mapped to ``/vagrant`` on the container

Method 2: Convert Dockerfile into a Singularity recipe.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Using the `https://singularityhub.github.io/singularity-cli/ <https://singularityhub.github.io/singularity-cli/>`_ toolkit, a Dockerfile can be converted to a Singularity recipe.

**Advantage:** You do not require root or docker group permissions and this can therefore be built on a login node.

**Disadvantage:** A 1-1 translation is typically not possible, so will require some edits. For instance, ``ARGS`` are not supported in Singularity recipes and switching to the ``root`` user can only be done using fakeroot techniques.

Method 3: Importing from a Dockerhub
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Alternatively, Singularity can import from a dockerhub. As of writing we do not have our own dockerhub, so this method will most likely be least useful.

Adjusting your container after building 
---------------------------------------

If you want to make changes after building the container, for instance to make the image smaller you can use the command (take note of the ``sudo``\ )\ ``sudo singularity build --sandbox code_YYYYMMDD docker-archive://code_YYYYMMDD.tar``.

Rather than creating an image, this creates a folder. You can enter the sandbox using ``sudo singularity shell --writable code_YYYYMMDD``.

Finding large packages could for instance be done using ``dpkg-query -W --showformat='${Installed-Size;10}\t${Package}\n' | sort -k1,1n``. ``apt remove gcc *-dev && apt autoremove && apt clean``.

The container can subsequently be saved using ``singularity build code_YYYYMMDD.sif code_YYYYMMDD/``.

Another example when this can be useful is **installing additional pip packages**. To do that:


# Save your current singularity image as a sandbox: ``singularity build --sandbox <your-sandbox-folder> <your-current-singularity-image>.sif`` . This can take a couple of minutes.

# Open the shell of the created sandbox: ``singularity shell --no-umask --writable <your-sandbox-folder>``. It is very important to add the argument ``--no-umask`` when using Lisa.
   **Details**\ : The value of ``umask`` is related to file permission. The default value of ``umask`` in Lisa is ``0077`` . However, these permissions are quite restrictive, allowing read access only for the file owner. When you install the package in singularity shell of the sandbox, you create a folder with this package. After creating the image and running a container with it, the owner of the folder changes, so inside the container you don’t have access to the installed package and you will not be able to use it. By using ``--no-umask`` , ``umask`` is set to less strict permissions, ``0022``. If you want even more details, see: `https://github.com/sylabs/singularity/issues/506 <https://github.com/sylabs/singularity/issues/506>`_

# Inside the shell, install the packages with pip: ``pip install <necessary-package>``.

# Close the shell

# Build the updated image with the pip package: ``singularity build <your-updated-image>.sif <your-sandbox-folder>/`` . Remember to add the slash ``/`` at the end. If you’re using Lisa, you might have to run this command as a job instead of the terminal in login node (due to time or memory limitations). For my image, it takes about 25 minutes.

Enroot
------

Enroot is a container platform developed by Nvidia with a similar goal as Singularity. Support in NKI-AI is experimental, and currently only installed on ptolemaeus. Find more information here: `https://github.com/NVIDIA/enroot <https://github.com/NVIDIA/enroot>`_.

Docker
------

We do not run docker images directly. If your application specifically requires Docker, submit a well-motivated request to your supervisor. Nevertheless, a Dockerfile is useful to describe your environment. Nevertheless, you can find instructions to install Docker at the `official documentation site <https://docs.docker.com/engine/install>`_.

Docker tips & tricks
--------------------

Since many repositories still have a Dockerfile to describe the environment, and might be used to create a docker image which is then converted to a singularity image, here is some general knowledge on how to write a good docker file:

Use few ``RUN`` commands
^^^^^^^^^^^^^^^^^^^^^^^^

Each ``RUN`` command creates a layer, and using many ``RUN`` commands for tiny installs may lead to a large Docker image. So instead of issuing many ``RUN`` commands like so:

.. code-block:: bash

    # setup python packages
    RUN conda update -n base conda -yq
    RUN conda install python=${PYTHON}
    RUN conda install pyyaml mkl mkl-include setuptools cmake cffi typing boost cython scipy -yq
    RUN conda install cudatoolkit=${CUDA} -c nvidia
    RUN conda install pytorch torchvision -c pytorch
    RUN conda install tqdm jupyter matplotlib scikit-image pandas joblib -yq
    RUN python -m pip install openslide-python opencv-python numpy==1.20 tifftools pycocotools -q/

Chain them together like so:

.. code-block:: bash

    # setup python packages
    RUN conda update -n base conda -yq \
    && conda install python=${PYTHON} \
    pyyaml mkl mkl-include setuptools cmake cffi typing boost cython scipy \
    tqdm jupyter matplotlib scikit-image pandas joblib -yq \
    && conda install cudatoolkit=${CUDA} -c nvidia \
    && conda install pytorch torchvision -c pytorch \
    && python -m pip install openslide-python opencv-python numpy==1.20 tifftools pycocotools -q \
    && conda install pytorch-lightning -c conda-forge \
    && python -m pip install tensorboardX tabulate -q


Changing projects and debugging
-------------------------------

* ``python -m pip install .`` or ``pip install .`` or ``python setup.py install`` copies the current state of the current directory, and requires a rebuild of the image when the project or dependency changes.

* ``python -m pip install -e .`` or ``python setup.py develop`` maps the repository name to the location of the directory.

* If you then start your container while binding your repository to that location, all the imports will reflect the code that you bind in the repository.

* This is especially useful if you are developing a new project. You can bind this project in your container, and your code changes are continuously reflected when running the container

* You have dependencies as submodules that you are co-developing with your current project. Instead of installing them from the git repo or from PyPI, you can add them as a submodule in your project, bind the project to the container, and install these dependencies in editable mode so that you don’t need to rebuild the image any time the submodule dependency changes.

Tips to reduce the docker image size
------------------------------------

It is preferable to have a small docker image. <1GB is considered small. <5GB is considered good. <10 GB is considered acceptable. >10GB is not considered acceptable.

``--squash``
------------

The ``--squash`` flag (\ `https://docs.docker.com/engine/reference/commandline/image\_build/ <https://docs.docker.com/engine/reference/commandline/image_build/>`_ ) squashes all layers into a single layer. The downside of this is that the layers can not be shared, and might increase build time when changing the ``Dockerfile``.


From building the Dockerfile to using the singularity container
---------------------------------------------------------------

Build the docker container with docker build
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   docker build -t docker_name:version_name /output/path/ -f path_to/Dockerfile
  
specific example:

.. code-block:: bash

   docker build -t drop_dlup:latest . -f docker/Dockerfile_orig


* You can also give the Dockerfile another name, there is no extension

* version name can be something like ‘1.0’ or also ‘latest’

Save the docker container as an image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* The container_id can be found by using **docker ps**

.. code-block:: bash

   docker save container_id -o docker_image_name.tar
   spcific example:
   docker save a2fbf0a33294 -o docker_new26-12.tar

To test that the docker container is what we want, we can test it in the shell.

* This way we can gradually add things and understand and see how the docker container changes.

.. code-block:: bash

   docker run -ti image_id /bin/bash

To delete unnecessary docker images to reduce memory:

.. code-block:: bash

   # to remove all dangling images
   docker image prune -a
   #to remove all stopped containers, networks that are not used by at least one container, dangling images, build cache
   docker system prune

Build singularity image from docker image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* The container_id can be found by using ``docker ps``

* See also above the information on: Converting docker to singularity- Method 1: Docker is already built

.. code-block:: bash

   singularity build singularity_image_name.sif docker-archive://docker_image_name.tar

specific example:

.. code-block:: bash

   singularity build drop_new_20211226.sif docker-archive://docker_new26-12.tar

Upload the singularity image to the server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* This can take a long time (3 hours)

.. code-block:: bash

    rsync -azv --progress=’info2’  drop_new_20211226.sif user_nki@rhpc-server_name:/path/to/singularity_image.sif

Using the singularity image
^^^^^^^^^^^^^^^^^^^^^^^^^^^

* A bash script can be made for launching the singularity image in a container and mapping the required GPUs to the singularity image.

* With slurm, GPU server numbers are automatically 0-N where N are the number of GPUs requested. Previously the exact GPU numbers needed to be provided in the format ``"1,3,5"`` as an example for GPUs 1, 3 and 5.

* Executing with sh is sufficient

* You can bind the locations you need to access in the singularity container with the --bind flag:

.. code-block:: bash

  --bind path/to/loc/on/server:path/to/loc/on/singularity, path/to/loc2/on/server:path/to/loc2/on/singularity

* The bound locations have no space in between the items. There is a space after the --bind flag as this is followed by the singularity image location

.. code-block:: bash

   #!/bin/bash
   read -p 'GPU indices to use (format example: "1,3,5"): ' GPU_IDX

   general:
   SINGULARITYENV_CUDA_VISIBLE_DEVICES="${GPU_IDX}" singularity shell --nv \
   --bind path/to/loc/on/server:path/to/loc/on/singularity,\
   path/to/loc2/on/server:path/to/loc2/on/singularity \
   /path/to/singularity_image.sif


specific example:

.. code-block:: bash

   SINGULARITYENV_CUDA_VISIBLE_DEVICES="${GPU_IDX}" singularity shell --nv \
   --bind /mnt/archive/projectdata/drop:/mnt/archive/projectdata/drop,\
   /mnt/archive/data/pathology:/mnt/archive/data/pathology,\
   /processing/"$USER":/scratch \
   /mnt/archive/projectdata/drop/containers/drop_20210713.sif

Dockerfile example with explanations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

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
