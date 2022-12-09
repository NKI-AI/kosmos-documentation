
The preferred way of executing programs on the RHPC cluster is through the use of containers. Containers enable you to package up software (code and dependencies) in a portable and reproducible fashion.

We support several container platforms: Singularity, Enroot (experimental) and Docker. The use of docker requires specific privileges, and Singularity is recommended. Singularity and Enroot containers can also be run on Surf’s Lisa cluster.

/*<![CDATA[*/ div.rbtoc1670582120005 {padding: 0px;} div.rbtoc1670582120005 ul {list-style: disc;margin-left: 0px;} div.rbtoc1670582120005 li {margin-left: 0px;padding-left: 0px;} /*]]>*/


* `Singularity <#Containers-Singularity>`_

  * `Converting Docker to Singularity <#Containers-ConvertingDockertoSingularity>`_

    * `Method 1: Docker container is already built <#Containers-Method1:Dockercontainerisalreadybuilt>`_
    * `Method 2: Convert Dockerfile into a Singularity recipe. <#Containers-Method2:ConvertDockerfileintoaSingularityrecipe.>`_
    * `Method 3: Importing from a Dockerhub <#Containers-Method3:ImportingfromaDockerhub>`_

  * `Adjusting your container after building (creating a smaller container; adding software) <#Containers-Adjustingyourcontainerafterbuilding(creatingasmallercontainer;addingsoftware>`_\ )

* `Enroot <#Containers-Enroot>`_
* `Docker <#Containers-Docker>`_

  * `Docker tips & tricks <#Containers-Dockertips&tricks>`_

    * `Use few RUN commands <#Containers-UsefewRUNcommands>`_
    * `Use RUN cd /path/to/project python -m pip install -e . for changing projects and debugging <#Containers-UseRUNcd/path/to/projectpython-mpipinstall-e.forchangingprojectsanddebugging>`_

  * `Tips to reduce the docker image size <#Containers-Tipstoreducethedockerimagesize>`_

    * `--squash <#Containers---squash>`_
    * `Reduce RUN commands <#Containers-ReduceRUNcommands>`_

Singularity
===========

Using Singularity you can package your complete code and dependencies in one image (a single file). In contrast to Docker singularity containers are unprivileged (they do not require root permissions).

A Singularity recipe (a file describing how the container should be built) can be written, but Docker containers can readily converted to Singularity containers.

Converting Docker to Singularity
--------------------------------

There are several ways to convert your Docker into a Singularity container.

( Check `https://singularity-userdoc.readthedocs.io/en/latest/troubleshooting.html <https://singularity-userdoc.readthedocs.io/en/latest/troubleshooting.html>`_ if you have questions about Singularity )

Method 1: Docker container is already built
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A convenient method to convert an existing docker container into a singularity container is:


#. 
   Find the image ID of your container using ``docker images``\ , for instance ``ef70ddca9b03``.

#. 
   Dump the container to disk ``docker save ef70ddca9b03 -o container_name.tar``. Consider a descriptive name for the container, for ``instance code_YYYYMMDD.tar``. If you do not have singularity installed you can upload this to the RHPC, otherwise you can convert it on your own machine.

#. 
   Convert to singularity using ``singularity build code_YYYYMMDD.sif docker-archive://code_YYYYMMDD.tar``. This will create an image ``code_YYYYMMDD.sif``


   #. 
      ``!!`` note that the ``docker-archive://`` is required even though you point to a local file, see `https://sylabs.io/guides/3.1/user-guide/singularity\_and\_docker.html <https://sylabs.io/guides/3.1/user-guide/singularity_and_docker.html>`_


      #. Then it’s explicit that it won’t load anything from dockerhub, and sudo is not required

   #. 
      ``!!`` Note that if you do this on LISA, the process will probably be killed if you run it on the login node since it’s too compute heavy and/or takes too long. Be sure to request a node to perform this on, or do it on one of the NKI servers.


      #. 
         If you get a “no space left on device” error, this can be resolved by setting a different cache dir that does have enough space, like

         .. code-block:: shell

            SINGULARITY_TMPDIR=/scratch/tmp SINGULARITY_CACHEDIR=/scratch/cache singularity build <rest of options>

   #. 
      :apple: If you want to do this locally on MacOS or Windows, you need to use a VM that runs singularity.


      #. 
         Install the required software: `https://sylabs.io/guides/3.0/user-guide/installation.html#mac <https://sylabs.io/guides/3.0/user-guide/installation.html#mac>`_

      #. 
         After you make the ``vm-singularity`` directory, place your ``.tar`` in this directory. This is the directory that will be mounted and mapped to ``/vagrant`` in the container

      #. 
         If your image is large , the VM might not have enough disk space (20480MB standard, so if your image is much larger than 20GB there might be issues).


         #. 
            Easiest is likely to increase this by following the ``vagrant-disksize`` plugin route from `https://askubuntu.com/questions/317338/how-can-i-increase-disk-size-on-a-vagrant-vm <https://askubuntu.com/questions/317338/how-can-i-increase-disk-size-on-a-vagrant-vm>`_

         #. 
            You also want to change the ``SINGULARITY_TMPDIR`` and ``SINGULARITY_CACHEDIR`` and ``SINGULARITY_LOCALCACHEDIR``\ directories, since this is where the final image is intermediately saved. That is: ``export THE_VAR=/vagrant``\ , since ``/vagrant`` is the the mount of your local disk. see `https://github.com/hpcng/singularity/issues/1052 <https://github.com/hpcng/singularity/issues/1052>`_ and `https://sylabs.io/guides/3.5/user-guide/build\_env.html#temporary-folders <https://sylabs.io/guides/3.5/user-guide/build_env.html#temporary-folders>`_

      #. 
         Then, run

         .. code-block:: java

            $ export VM=sylabs/singularity-3.0-ubuntu-bionic64 && \
                vagrant init $VM && \
                vagrant up && \
                vagrant ssh

      #. 
         Run the singularity converting line as stated above. Note that the ``vm-vagrant`` dir on your local machine is mapped to ``/vagrant`` on the container

Method 2: Convert Dockerfile into a Singularity recipe.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Using the `https://singularityhub.github.io/singularity-cli/ <https://singularityhub.github.io/singularity-cli/>`_ toolkit, a Dockerfile can be converted to a Singularity recipe.

**Advantage:** You do not require root or docker group permissions and this can therefore be built on a login node.

**Disadvantage:** A 1-1 translation is typically not possible, so will require some edits. For instance, ``ARGS`` are not supported in Singularity recipes and switching to the ``root`` user can only be done using fakeroot techniques.

Method 3: Importing from a Dockerhub
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Alternatively, Singularity can import from a dockerhub. As of writing we do not have our own dockerhub, so this method will most likely be least useful.

Adjusting your container after building (creating a smaller container; adding software)
---------------------------------------------------------------------------------------

If you want to make changes after building the container, for instance to make the image smaller you can use the command (take note of the ``sudo``\ )\ ``sudo singularity build --sandbox code_YYYYMMDD docker-archive://code_YYYYMMDD.tar``.

Rather than creating an image, this creates a folder. You can enter the sandbox using ``sudo singularity shell --writable code_YYYYMMDD``.

Finding large packages could for instance be done using ``dpkg-query -W --showformat='${Installed-Size;10}\t${Package}\n' | sort -k1,1n``. ``apt remove gcc *-dev && apt autoremove && apt clean``.

The container can subsequently be saved using ``singularity build code_YYYYMMDD.sif code_YYYYMMDD/``.

Another example when this can be useful is **installing additional pip packages**. To do that:


#. 
   Save your current singularity image as a sandbox: ``singularity build --sandbox <your-sandbox-folder> <your-current-singularity-image>.sif`` . This can take a couple of minutes.

#. 
   Open the shell of the created sandbox: ``singularity shell --no-umask --writable <your-sandbox-folder>``. It is very important to add the argument ``--no-umask`` when using Lisa.\ :raw-html-m2r:`<br>`
   **Details**\ : The value of ``umask`` is related to file permission. The default value of ``umask`` in Lisa is ``0077`` . However, these permissions are quite restrictive, allowing read access only for the file owner. When you install the package in singularity shell of the sandbox, you create a folder with this package. After creating the image and running a container with it, the owner of the folder changes, so inside the container you don’t have access to the installed package and you will not be able to use it. By using ``--no-umask`` , ``umask`` is set to less strict permissions, ``0022``. If you want even more details, see: `https://github.com/sylabs/singularity/issues/506 <https://github.com/sylabs/singularity/issues/506>`_

#. 
   Inside the shell, install the packages with pip: ``pip install <necessary-package>``.

#. 
   Close the shell

#. 
   Build the updated image with the pip package: ``singularity build <your-updated-image>.sif <your-sandbox-folder>/`` . Remember to add the slash ``/`` at the end. If you’re using Lisa, you might have to run this command as a job instead of the terminal in login node (due to time or memory limitations). For my image, it takes about 25 minutes.

Enroot
======

Enroot is a container platform developed by Nvidia with a similar goal as Singularity. Support in NKI-AI is experimental, and currently only installed on ptolemaeus. Find more information here: `https://github.com/NVIDIA/enroot <https://github.com/NVIDIA/enroot>`_.

Docker
======

We do not run docker images directly. If your application specifically requires Docker, submit a well-motivated request to your supervisor. Nevertheless, a Dockerfile is useful to describe your environment. Nevertheless, you can find instructions to install Docker at the `official documentation site <https://docs.docker.com/engine/install>`_.

Docker tips & tricks
--------------------

Since many repositories still have a Dockerfile to describe the environment, and might be used to create a docker image which is then converted to a singularity image, here is some general knowledge on how to write a good docker file:

Use few ``RUN`` commands
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each ``RUN`` command creates a layer, and using many ``RUN`` commands for tiny installs may lead to a large Docker image. So link them together with ``&&`` :

# setup python packages

RUN conda update -n base conda -yq

RUN conda install python=${PYTHON}

RUN conda install pyyaml mkl mkl-include setuptools cmake cffi typing boost cython scipy -yq

RUN conda install cudatoolkit=${CUDA} -c nvidia

RUN conda install pytorch torchvision -c pytorch

RUN conda install tqdm jupyter matplotlib scikit-image pandas joblib -yq

RUN python -m pip install openslide-python opencv-python numpy==1.20 tifftools pycocotools -q/

# setup python packages

RUN conda update -n base conda -yq \

&& conda install python=${PYTHON} \

pyyaml mkl mkl-include setuptools cmake cffi typing boost cython scipy \

tqdm jupyter matplotlib scikit-image pandas joblib -yq \

&& conda install cudatoolkit=${CUDA} -c nvidia \

&& conda install pytorch torchvision -c pytorch \

&& python -m pip install openslide-python opencv-python numpy==1.20 tifftools pycocotools -q \

&& conda install pytorch-lightning -c conda-forge \

# ^-- dependencies precision-dcis project \

&& python -m pip install tensorboardX tabulate -q

Use ``RUN cd /path/to/project python -m pip install -e .`` for changing projects and debugging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  ``python -m pip install .`` or ``pip install .`` or ``python setup.py install`` copies the current state of the current directory, and requires a rebuild of the image when the project or dependency changes.

* 
  ``python -m pip install -e .`` or ``python setup.py develop`` maps the repository name to the location of the directory.


  * 
    If you then start your container while binding your repository to that location, all the imports will reflect the code that you bind in the repository.

  * 
    This is especially useful if


    * 
      You are developing a new project. You can bind this project in your container, and your code changes are continuously reflected when running the container

    * 
      You have dependencies as submodules that you are co-developing with your current project. Instead of installing them from the git repo or from PyPI, you can add them as a submodule in your project, bind the project to the container, and install these dependencies in editable mode so that you don’t need to rebuild the image any time the submodule dependency changes.

Tips to reduce the docker image size
------------------------------------

It is preferable to have a small docker image. <1GB is considered small. <5GB is considered good. <10 GB is considered acceptable. >10GB is not considered acceptable.

``--squash``
~~~~~~~~~~~~~~~~

The ``--squash`` flag (\ `https://docs.docker.com/engine/reference/commandline/image\_build/ <https://docs.docker.com/engine/reference/commandline/image_build/>`_ ) squashes all layers into a single layer. The downside of this is that the layers can not be shared, and might increase build time when changing the ``Dockerfile``.

Reduce ``RUN`` commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~

:point_up: see section above
