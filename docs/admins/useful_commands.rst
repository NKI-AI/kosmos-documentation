===============
Useful commands
===============

Switch to user <username>:

.. code-block:: bash
   
   sudo -i -u <username>

Change project folder permissions to <group>:

.. code-block:: bash
   
   getent group <group>
   chgrp <group> /projects/<project_folder>
   chmod -R g+rw /projects/<project_folder>
