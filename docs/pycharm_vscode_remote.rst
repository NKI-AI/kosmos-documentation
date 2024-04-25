==================================
Connecting Your IDE to the Cluster
==================================

Both PyCharm and Visual Studio Code can be configured for remote development on the cluster via SSH, allowing you to run/debug code directly on the RHPC cluster nodes. Below, you can find the steps for setting up remote development on either PyCharm or Visual Studio Code.

-----------------------
PyCharm and the Cluster
-----------------------

PyCharm can be tricky to set up properly, and the steps below should guide you through the process. Make sure to update PyCharm to the most recent version — the interface may be different for older versions (e.g. < 2022). 

SSH connections
^^^^^^^^^^^^^^^

First, you’ll want to set up (in PyCharm) the SSH connections that you’re going to need, which you can do in Preferences -> Tools -> SSH Configurations. You definitely need a setup for the node you wish to connect to (for example ``gaia``, ``hamilton``, etc.) in order to do the deployment/sync, and then some more connections to the compute nodes for connecting to a remote interpreter. In this example, we’ll use ``ptolemaeus`` as a target for that and ``gaia`` as the node I want to connect to.

So, we create two connections, which we call ``rhpc-gaia`` and ``rhpc-ptolemaeus``.  Use the local .ssh/config file for authentication, and keep all other default settings. These are then available globally in PyCharm (i.e. not project-specific). Test the connections with the button just to be sure. In my setup, I created a directory ~/pycharm-sync in my homefolder on the rhpc cluster to sync all projects to, but you may want to use a different method of course.

Next, create a pycharm project on your local machine (and add code if you wish). So far it’s all quite straightforward, now the actual deployment/interpreter setup starts.

Remote deployment/sync
^^^^^^^^^^^^^^^^^^^^^^

This creates a configuration for pycharm so that it knows where to upload the code you edit locally.  If you do this properly, every time you save a file locally it’ll get synced to the remote server instantly.

#. Go to Tools -> Deployment -> Configuration, and click the + button to create a new configuration (SFTP). Enter some name (``rhpc-gaia`` in this case, it’s not important).
#. Choose the ``gaia`` SSH configuration that you made earlier. Test the connection, this should work fine [#]_. You can autodetect the root path, which it’ll set to your home directory.
#. Check the ``rsync`` option if that works for you, apparently this is troublesome on Windows, but it’s super nice if you can use it.
#. Configure path mappings in the “Mappings” tab. The local path should point to your local project root, the deployment path should be the relative path to the folder your project should sync to (relative to the rooth path you specified earlier). In this case, the deployment path is simply ``pycharm-sync/<project-name>``. We don’t use the web path.
#. Now set this configuration as the default by clicking the check mark ✓ in the top left. PyCharm is a little stubborn here so you may need to do so again later on.
#. If it isn’t already enabled, make sure to set the project to “automatic upload”.
#. Upload the project. You can either do this through Tools -> Deployment -> Upload to ``rhpc-gaia``, or right click the root project folder in the project explorer and go to the deployment context menu.
#. Check whether all files made it to your filesystem on the rhpc cluster properly (after upload has completed).

.. [#] In case it doesn't your SSH is most likely not up to date, which happens most likely with pre-installed version on NKI-Windows systems. Should be fixed after the following steps

   #. Download and install the most recent version of SSH: https://github.com/PowerShell/Win32-OpenSSH/releases
   #. Change the Proxycommand for rhpc-gaia to use the newer SSH install:

.. code-block:: java

   Host rhpc
     User <username>
     HostName rhpc.nki.nl
     ServerAliveInterval 60
     ForwardAgent yes
     Compression yes
     ForwardX11 yes

   Host rhpc-gaia
     Hostname gaia
     User <username>
     ProxyCommand <path/to/new/install>/ssh.exe -W %h:%p rhpc


Remote interpeter
^^^^^^^^^^^^^^^^^

You'll need to add separate interpreters for every node that you want to use, for every project individually (assuming that you have different environments for different projects, which you probably should).

#. In Preferences -> Project -> Python Interpreter (or through the interpreter thing in the bottom right of the window), create the interpreter configuration. Click Add interpreter -> On SSH, choose “Existing” and select one of the slurm node SSH configs — ``rhpc-ptolemaeus`` in my case. It’ll do some introspection to check whether everything’s fine, there shouldn’t be any errors popping up here (otherwise you messed up somewhere else or the server is having issues).
#. Do Next, now you can select your interpreter of choice. I’m using conda, so I select “System interpreter” (not Virtualenv). You can probably make it work with virtualenv as well.
#. In the “Interpreter” box, navigate to the interpreter in your conda environment. As an example, for me it’s /home/j.brunekreef/miniconda3/envs/<env-name>/bin/python3.9 .
#. Remove the sync folder configuration. **This is important!!** It defaults to something like /tmp/pycharm_project_xxx, but you want to delete this. Open this configuration, click the single row, and click the - sign to remove the configuration. Confirm by clicking OK.
#. Finalize creating the interpreter by clicking Create.
#. Now verify whether your deployment configuration hasn’t been messed up by these previous steps. You should mostly check whether the ``rhpc-gaia`` configuration that you created for deployment is indeed still the default, and that it wasn’t overridden by some connection in the remote interpreter.


You can now check whether the remote interpreter works by going to the Python console in PyCharm, this should give you a console on the remote server. For example, enter

.. code-block:: python

   import socket
   socket.gethostname()

this should print ``ptolemaeus`` in my example configuration

Run/debug configurations
^^^^^^^^^^^^^^^^^^^^^^^^

Now the only thing left to do is to create run/debug configurations. Add a python run configuration.
The script path should be the path to your local python script. You’ll need to map it to the remote file manually (I didn’t find a smarter way of doing this), I’ll get to that.
Select the python interpreter you just created. The working directory should be the local path to the directory you want to use as your working directory — for example, the project root.
In the path mapping, specify that the local project root should be mapped to the remote project root. So, for example:

    ``/Users/joren/Code/<project-name> = /home/j.brunekreef/pycharm-sync/<project-name>``

All done! When connecting to you desired node, don't forgot to request a job for that node (``gaia`` in this example) by ssh-ing into kosmos via your terminal and doing the job request. 

----------------------------------
Visual Studio Code and the Cluster
----------------------------------

Setting up VS Code for remote development on the cluster is a rather straightforward process. The following instructions will guide you through the steps, assuming your VS Code version is up-to-date (>2023). 


Downloading the Remote-SSH Extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To connect VS Code to the cluster, we will use the convenient Remote-SSH extension created by Microsoft. There are two ways to get this extension: it can either be downloaded from the extensions menu of VS Code by searching for its name, or, it can automatically be downloaded as part of the Remote Development extension pack of VS code. To add this pack, simply click the blue "remote host" button (also called "open a remote window" in some versions) in the bottom left corner of your VS code window and select "SSH". The extension will now be installed. 

Setting up the Remote-SSH Extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we have downloaded the extension, we can set up our connection to RHPC. Connections can be added manually, but the extension also includes the option to load your ssh ``config`` file, allowing you to immediately add all necessary connections. Navigate to the Remote-SSH extension, either via the remote window-button, or via the command palette and select ``Remote-SSH: Open SSH Configuration File...``. You will now be prompted for the location of your config file, which by default (on Linux) is ``~/.ssh/config``. The config file should now successfully be loaded and from now on, a list of all host connections will appear when you select ``Remote-SSH: Connect to Host...`` from the Command Palette. 

Connections can also be added manually. For this, select ``Remote-SSH: Add New SSH Host...`` from the command palette. Next, input the command that you would usually use to connect to the cluster, i.e. ``ssh rhpc-gaia`` and the ``config`` file when prompted. Your host will now show up in the possible connections list. 

Connecting to the Cluster via the Remote-SSH Extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   **Warning** *: Make sure to* **never** *connect VS Code to* ``kosmos`` *directly. This will run a VS Code server on the login node that slows the server down for all users. Always request a job on a compute node (either CPU or GPU) first, and then connect VS code to that node using the instructions below. You risk a temporary cluster ban if you run VS Code directly on* ``kosmos`` *.*


Now that the hosts are set up, we can connect to the cluster. Select ``Remote-SSH: Connect to Host...`` from the Command Palette, and pick the host you want to connect to from the list. This can be any host that you have submitted a job request for on kosmos (and of course *never connect VS Code to kosmos itself*); ``gaia`` is a good choice for general purpose tasks, file-management, and projects where you need processor power, but you can also directly connect to any of the compute nodes (such as ``hamilton``, ``alanturing``) to run/debug from there. 
If asked to supply the platform type of the server, select Linux. VS Code will now attempt to connect to your selected host. Keep an eye out on your terminal and "output" tab, as you may be asked to input your username and/or password the first time you connect. 

If all went well, you will be ready for remote development! Any folders on the server can be opened as folders in VS code, saved code will be updated automatically, and any terminal you open will act on the server. 
