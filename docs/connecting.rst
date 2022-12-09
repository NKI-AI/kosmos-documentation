
Get access and login to the servers
-----------------------------------

You can get access to the system by sending an e-mail to the RHPC admins (\ `rhpc-admin@nki.nl <mailto:rhpc-admin@nki.nl>`_\ ) after asking permission to Jonas Teuwen (send with Jonas in CC). They will then create an account for you on the server, set up a password and help you set up an authenticator app. You will need these to login to the server without an ssh connection.

Here are the commands to login to the server:

`https://wiki.rhpc.nki.nl/wiki/index.php/Login\_to\_RHPC <https://wiki.rhpc.nki.nl/wiki/index.php/Login_to_RHPC>`_

If you do this, you will be prompted to provide several passwords. First, it’s the two factors for ``rhpc.nki.nl`` `as described above <#ubuntu-nki-rhpc-ssh-passwords>`_. Then, the ``password`` is the concatenation of your set password and the 2FA code. So if your password is ``F00!b4r?`` and your 2fa code is ``123 123``\ , the password here would be ``F00!b4r?123123``.

To connect to the servers with ssh (highly recommended)
-------------------------------------------------------

If you are a windows user, install windows subsystem for linux (wsl) or ubuntu. If you are planning to use Pycharm remote deployment (very likely), then also set up the connection through Windows Powershell as explained below. Do not use the MobaXterm option, as this is more difficult to set up and offers less benefits like the direct ssh connection or connecting to the servers through Pycharm.

For this you need to create a shh key pair.

Ubuntu/WSL: Connect to rhpc
^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   Open shell

#. 
   Generate ssh key pair with: ``ssh-keygen -t rsa -b 4096 -C "USERNAME@rhpc.nki.nl"``


   #. 
      First prompt: Press enter to accept the default file location in which the public (id_rsa.pub) and private (id_rsa) key pair should be saved.

   #. 
      Second prompt: Enter a passphrase to encrypt your private key!

#. 
   Type ``ssh-add``

#. 
   Go to the .ssh folder ``cd ~/.ssh``

#. 
   Type ``cat id_rsa.pub`` → Prints the public key to the shell, so you can easily copy it Ctrl+C

#. 
   Send the public key to the rhpc-admins so they can add it: `rhpc-admin@nki.nl <mailto:rhpc-admin@nki.nl>`_

You should now be able to connect to the rhpc server via ssh without a password after the rhpc admins add your ssh key to the cluster. Test this with command: ``ssh rhpc``

Ubuntu/WSL/MacOS: To jump between rhpc servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. Create an ssh config file in ``~/.ssh/config``\ , with the following information:

.. code-block:: java

   ControlMaster auto
   ControlPath <your home folder on local machine>/.ssh/tmp/%h_%p_%r

   Host rhpc
     User <enter your RHPC username>
     HostName rhpc.nki.nl
     ServerAliveInterval 60
     ForwardAgent yes
     Compression yes
     ForwardX11 yes

   Host rhpc-wallace
     Hostname wallace
     User <enter your RHPC username>
     ProxyJump rhpc

   Host rhpc-atlas
     Hostname atlas
     User <enter your RHPC username>
     ProxyJump rhpc

   Host rhpc-ptolemaeus
     Hostname ptolemaeus
     User <enter your RHPC username>
     ProxyJump rhpc

   Host rhpc-aristarchus
     Hostname aristarchus
     User <enter your RHPC username>
     ProxyJump rhpc

   Host rhpc-eudoxus
     Hostname eudoxus
     User <enter your RHPC username>
     ProxyJump rhpc

   Host rhpc-eratosthenes
     Hostname eratosthenes
     User <enter your RHPC username>
     ProxyJump rhpc

2. chmod 600 config

3. Create a ``~/.ssh/tmp`` folder and give proper permissions (chmod 700).

4. Also give proper permissions to your RSA key with ``chmod 600 ~/.ssh/id_rsa`` and ``chmod 600 ~/.ssh/id_rsa.pub``.

You will then be able to directly jump over the `rhpc.nki.nl <http://rhpc.nki.nl>`_ host by logging in for instance with ``ssh rhpc-ptolemaeus``. Also PyCharm remote deployment should work.

Connect to rhpc with Windows Powershell (necessary for PyCharm Remote Deployment):
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   Make sure **OpenSSH Client** is in Apps & Features. If not, install it.

#. 
   Run as **administrator** the **Windows command prompt or Windows Powershell. Do not use Ubuntu or WSL.** Windows cmd and WSL need to make separate ssh connections to the server. Pycharm remote deployment on Windows is only possible through the ssh connection made by the Windows cmd to the remote server.

#. 
   Ideally, make sure that the folder **C:\Users\\ *username*\ /.ssh/** is empty to avoid any conflicts. (rm * )

#. 
   Type ``ssh-keygen`` or ``ssh-keygen -t rsa -b 4096 -C "USERNAME@rhpc.nki.nl"``

#. 
   Press enter to save your ssh private and public ssh keys in folder **C:\Users\\ *username*\ /.ssh/.**

#. 
    You’ll be asked to \ **enter a passphrase.**\  Hit \ **Enter**\  to skip this step.

#. 
   The system will generate the key pair, and display the  key fingerprint and a randomart image.

#. 
   On your local windows machine, open WSL and navigate to **/mnt/c/Users/username/.ssh** (Not !! ~/.ssh as this is a different directory in WSL).

#. 
   You should see two files. The identification is saved in the 

   **id_rsa**\  file and the public key is labeled \ **id_rsa.pub**. This is your SSH key pair.

#. 
   To add the private ssh-key to the ssh agent in Windows Powershell:


   #. 
      By default the ssh-agent service is disabled. Allow it to be manually started for the next step to work.

      # Make sure you're running as an Administrator.

      ``Get-Service ssh-agent | Set-Service -StartupType Manual``

      # Start the service

      ``Start-Service ssh-agent``

      # This should return a status of Running

      ``Get-Service ssh-agent``

      # Now load your key files into ssh-agent

      ``ssh-add C:\Users\username\.ssh\id_rsa`` (private key)

#. 
   Print your public ssh key that you generated for the connection to the server with **cat id_rsa.pub.** Then copy the key.

#. 
   Send the public key to the rhpc-admins so they can add it: `rhpc-admin@nki.nl <mailto:rhpc-admin@nki.nl>`_

You should now be able to connect to the rhpc server via ssh without a password after the rhpc admins add your ssh key to the cluster. Test this with command: ``ssh rhpc``

Windows Powershell: To jump between rhpc servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   Create a config file that contains:

   .. code-block:: java

      Host rhpc
        User <username>
        HostName rhpc.nki.nl
        ServerAliveInterval 60
        ForwardAgent yes
        Compression yes
        ForwardX11 yes

      Host rhpc-wallace
        User <username>
        HostName wallace
        ProxyCommand ssh -W %h:%p rhpc

      Host rhpc-atlas
        User <username>
        HostName atlas
        ProxyCommand ssh -W %h:%p rhpc

      Host rhpc-aristarchus
        User <username>
        HostName aristarchus
        ProxyCommand ssh -W %h:%p rhpc

      Host rhpc-ptolemaeus
        User <username>
        HostName ptolemaeus
        ProxyCommand ssh -W %h:%p rhpc

      Host rhpc-eudoxus
        Hostname eudoxus
        User <enter your RHPC username>
        ProxyCommand ssh -W %h:%p rhpc

      Host rhpc-eratosthenes
        Hostname eratosthenes
        User <enter your RHPC username>
        ProxyCommand ssh -W %h:%p rhpc

   Save as ``config`` (no extention) in the ``C:/Users/your_username/.ssh`` directory.

#. 
   You will now be able to directly jump over the `rhpc.nki.nl <http://rhpc.nki.nl>`_ host by logging in for instance with ``ssh rhpc-ptolemaeus``.Also PyCharm remote deployment should work.

Not recommended: Option to manually add ssh keys to the rhpc server:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This option does not allow jumping between servers without a password.


#. 
   Connect to the rhpc server (not one of the machines like aristarchus) with: ``ssh rhpc`` (if your wsl ssh connection is already set up) or alternatively with ``ssh username@rhpc.nki.nl`` and use your password and authenticator to login

#. 
   Go to ~/.ssh/ on rhpc server

#. 
   Create file ``authorized_keys`` with no extension (for example with **nano authorized_keys** ).

#. 
   Paste the public ssh key in the file. It appears as one line. Save the file.

#. 
   Important! Change permissions for authorised keys: **chmod 700 authorized_keys**
