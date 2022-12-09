
In order to use the compute nodes for training your models, start interactive sessions or use the remote debugging on pycharm, you may want to use some system libraries specific to your application. On RHPC, this is managed by a `package manager <https://nki-research-it.atlassian.net/wiki/spaces/NKIAI/pages/1984233497/Compute+cluster+NKI+Kosmos#Installed-software>`_ called ``spack``. This article will guide you to properly load the system packages you want to use while using the compute nodes.

For illustration, we will consider three user-specific system packages ``pyvips``\ , ``openslide`` and ``pixman``\ , and CUDA drivers which are generally necessary for everyone.

/*<![CDATA[*/ div.rbtoc1670582130152 {padding: 0px;} div.rbtoc1670582130152 ul {list-style: disc;margin-left: 0px;} div.rbtoc1670582130152 li {margin-left: 0px;padding-left: 0px;} /*]]>*/


* `An important environment variable <#Loadingsystempackagesoncomputenodesusingspack-Animportantenvironmentvariable>`_
* `Loading the right CUDA drivers <#Loadingsystempackagesoncomputenodesusingspack-LoadingtherightCUDAdrivers>`_
* `Loading user-specific system packages <#Loadingsystempackagesoncomputenodesusingspack-Loadinguser-specificsystempackages>`_
* `Resetting the python interpreter <#Loadingsystempackagesoncomputenodesusingspack-Resettingthepythoninterpreter>`_
* `Setting the library paths in the pycharm remote debugger <#Loadingsystempackagesoncomputenodesusingspack-Settingthelibrarypathsinthepycharmremotedebugger>`_

An important environment variable
---------------------------------

On any system, the environment variable ``LD_LIBRARY_PATH`` is important. This is so that you know the exact paths to each of the system packages you need the computer to use while running your jobs. Usually, we set this variable in the ``~/.bashrc`` file with the paths to relevant packages installed within the system. With spack, we let the package manager automatically take care of this for us.

Before we start loading packages using spack, let us unset the ``LD_LIBRARY_PATH`` just to be sure. Although, this is not a necessary step.

.. code-block:: java

   unset LD_LIBRARY_PATH

Now, let us look at how to load some important system packages.

Loading the right CUDA drivers
------------------------------

RHPC has different GPU nodes with different architectures. It is important to know what version of CUDA is necessary to use a particular node. This can be found on this `page <https://nki-research-it.atlassian.net/wiki/spaces/NKIAI/pages/1984233497/Compute+cluster+NKI+Kosmos#GPU-nodes>`_.

To illustrate an example, let us consider an interactive session on aristarchus. Before loading this system library, we must first perform a search of all the CUDA libraries installed. Use the following in your bash terminal:

.. code-block:: bash

   spack find cuda


.. image:: attachments/2465595492/2465464589.png?width=340
   :target: attachments/2465595492/2465464589.png?width=340
   :alt: 


Aristarchus has eight A6000 cards compatible with CUDA versions higher than 11. So, we can choose any one of the libraries in principle but be mindful of other python libraries you may be using that is dependent on the CUDA drivers (for example, pytorch) and choose the right one.

To load your preferred CUDA driver, use:

.. code-block:: bash

   spack load cuda@version

Now, let’s look at the ``LD_LIBRARY_PATH`` environment variable to confirm if these changes have been properly made.

.. code-block:: java

   echo $LD_LIBRARY_PATH


.. image:: attachments/2465595492/2465497321.png?width=680
   :target: attachments/2465595492/2465497321.png?width=680
   :alt: 


Along with this, you can also load ``cudnn`` in a similar manner.

.. code-block:: bash

   spack find cudnn
   spack load cudnn@version

Loading user-specific system packages
-------------------------------------

If you want to use `dlup <https://github.com/NKI-AI/dlup>`_\ , you need specific versions of openslide, pixman and pyvips. Without these, your implementations won’t work.

We repeat the steps described above to find the available packages and load them:

.. code-block:: bash

   spack load openslide@3.4.1
   spack load pixman@0.40.0
   spack load libvips@8.13.0

Note, however, that if you wish to use the aiforoncology fork of openslide so be able to read ``.mrxs`` files from the 3dhistech scanner, you need to load pixman and libvips **first**\ , and the ``openslide-aifo@3.4.1-nki`` package **last**. It seems that otherwise libvips overwrites the openslide aifo fork with the normal installation.

.. code-block:: java

   # Install system dependencies
   spack load pixman@0.40.0
   spack load libvips@8.13.0
   spack load openslide-aifo@3.4.1-nki

Resetting the python interpreter
--------------------------------

spack offers a default python interpreter which may not be useful for you in all scenarios. If you have a particular python interpreter in a separate conda environment set up with special python libraries, you should first select it before continuing with your job submissions. For this, first do the following:

.. code-block:: bash

   spack unload py-pip py-wheel py-setuptools python

then, activate your conda environment normally and verify the interpreter path.

.. code-block:: java

   conda activate <env_name>
   which python


.. image:: attachments/2465595492/2465792078.png?width=544
   :target: attachments/2465595492/2465792078.png?width=544
   :alt: 


The path to the python interpreter inside your environment should be printed on the console.

Setting the library paths in the pycharm remote debugger
--------------------------------------------------------

If you’re someone like me, then you probably use a remote debugger for all the dirty work with your code. To debug remotely with the right system packages, load all the necessary packages described above and copy the contents in the ``LD_LIBRARY_PATH`` variable.

Paste this into the environment variables field in the pycharm debugger settings.


.. image:: attachments/2465595492/2465464647.png
   :target: attachments/2465595492/2465464647.png
   :alt: 


If you have done this, then congratulations! you have configured everything correctly! Good luck with training your models and debugging your code!

Note: Sometimes it helps to do the spack loading and unsetting, but not having the environment variable in pycharm
