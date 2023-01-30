.. _pycharm-remote-development-windows:

==============================================
Pycharm remote development on Windows
==============================================

.. contents::

Requirements
------------
You should already have the following set up before trying to set up remote development

- Connection to RHPC servers via authentication agent
- SSH config file with jumping between RHPC-servers
- PyCharm Professional Edition

Set up
------

#. Open PyCharm
#. Go to Tools → Deployment → Configuration
#. Click the + symbol, choose SFTP and choose a name (can be anything)
#. Select your SSH configuration or set one up in PyCharm as follows:
	#. Click the three dots next to <select configuration>
	#. Click the + symbol 
	#. Enter the hostname (i.e. rhpc-aristarchus) and username (i.e. initial.lastname)
	#. Change the Authentication type to "OpenSSH config and authentication agent"
	#. Click "Test Connection" to see if it works, this should show a popup saying "Successfully connected!"
	#. Close the SSH configurations tap by clicking OK in the bottom right corner and you will return to the deployment tab
#. Click "Test Connection" to make sure the connection works
#. Under Mappings check if the local path is correctly filled in and fill in the deployment path

You are now set up to use remote deployment


Usage
-----

To use remote deployment there are two methods. First make sure that the correct deployment server is selected by going to Tools → Deployment → Configuration or by clicking the SFTP symbol in the bottom bar.

The most straightforward way is to enable automatic deployment. Under Tools → Deployment select "Automatic Upload"

Another way to use deployment is to do it manually. You can either deploy multiple files via Tools → Deployment, or you can right click a file or directory → Deployment and select the appropriate option there.

Trouble Shooting
----------------

- When setting up an SSH connection in PyCharm the "test connection"-button does not work

    - Make sure your ssh config file is in the correct place and PyCharm can find this file.
    - Make sure your username and hostname are correctly specified and match your ssh config file
    - Try replacing ``ProxyJump rhpc`` by ``ProxyCommand -W %h:%p rhpc`` in the ssh config file.

- (Automatic) Deployment fails

    - Make sure the path mapping is set up correctly
    - Empty the project and reupload everything manually

