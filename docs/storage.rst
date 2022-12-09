.. role:: raw-html-m2r(raw)
   :format: html


There are several storage systems available, for data, document and code storage.


* `Data storage <#Storage-Datastorage>`_

  * `Requesting access for I:\group Teuwen: <#Storage-RequestingaccessforI:\groupTeuwen:>`_
  * `Add I:\group Teuwen to network: <#Storage-AddI:\groupTeuwentonetwork:>`_
  * `Mounting I:\group Teuwen to a linux machine (connected to the NKI network): <#Storage-MountingI:\groupTeuwentoalinuxmachine(connectedtotheNKInetwork>`_\ :)

* `Sharing large files with externals <#Storage-Sharinglargefileswithexternals>`_
* `Transferring files from surf filesender using the command line <#Storage-Transferringfilesfromsurffilesenderusingthecommandline>`_
* `Document storage <#Storage-Documentstorage>`_
* `Code storage <#Storage-Codestorage>`_

Data storage
============


.. list-table::
   :widths: 20 10 10 10 10 10
   :header-rows: 1

   * - Location
     - Space
     - Request access
     - Backup
     - Accessibility
     - Remarks
   * - \\image-storage\group Teuwen
     - 50TB
     - Servicedesk (webform)
     - Daily incremental 
     - Only within NKI network
     - Cannot be used to directly load data for training models   
   * - \mnt\archive\data
     - ±300TB
     - rhpc-admin@nki.nl
     - No
     - Only within RHPC network (rhpc.nki.nl)
     - Contains the home folders on RHPC                          
   * - Surf Lisa storage
     - 50TB
     - rhpc-admin@nki.nl
     - No
     - Only within Surf’s network
     - This is shared with several groups - ACL possible
      
Requesting access for ``I:\group Teuwen``\ :
----------------------------------------------


#. 
   Log in to the NKI intranet:


   #. 
      By logging in to the NKI virtual desktop (VM environment)

   #. 
      Or a physical device (Work PC) linked to the NKI network

#. 
   Access the Service Portal at `https://topdesk.nki.nl/tas/public/xfg/cat4toegang <https://topdesk.nki.nl/tas/public/xfg/cat4toegang>`_

#. 
   Choose option 10 from the service list: “\ *10. toegang netwerkschijf map*\ ” (access network disk folder)

#. 
   Fill in the form details as follows:


   #. 
      e.g.:


      .. image:: attachments/requesting-access.png
         :target: _images/requesting-access.png
         :alt: 


   #. Ask permission from your supervisor to request for access

#. 
   Submit form


   #. 
      You will receive an email confirmation for submitting the form

   #. 
      And another email for the service desk action (after they contact the drive owner, e.g.: Jonas)

#. 
   In case you need to ask for access rights to another account (not in the lists of coworkers, e.g.: svs-slidescore for displaying WSIs on slidescore, managed by Jan Hudecek)


   #. 
      After submitting the form, click on the link in the email confirmation to view the request

   #. 
      Add a reply/comment with your request such as: “\ *please give access to the account svs-slidescore*\ “

Add ``I:\group Teuwen`` to network:
---------------------------------------


#. 
   Log in to the NKI VM:

#. 
   Go to ‘This PC’

#. 
   Click ‘Computer’ on the top left

#. 
   Click ‘Map network drive’, select ‘map network drive’

#. 
   Enter ``I:\group Teuwen`` as folder

Mounting ``I:\group Teuwen`` to a linux machine (connected to the NKI network):
-----------------------------------------------------------------------------------

.. code-block:: bash

   sudo mount -t cifs -o username=<NKI_username>,vers=2.0,uid=$(id -u),gid=$(id -g),file_mode=0777,dir_mode=0777 //172.20.3.112/"Group Teuwen" <mount_dir>

where **<NKI_username>** is the username used for NKI login e.g.: ``a.panteli`` for `a.panteli@nki.nl <mailto:a.panteli@nki.nl>`_\ , and **<mount_dir>** is a directory path which will mount the contents of the image-storage drive (preferably an empty folder).

From this point navigating to the <mount_dir> folder will also allow for viewing the contents of the image-storage drive and can add/remove contents to subfolders like the “Group Teuwen/“ folder simply by placing them at <mount_dir>/”Group Teuwen”/.

Copying data from another server to the image-storage drive can be as simple as:

.. code-block:: bash

   rsync --progress -cav <user>@<server_url>:<dir_to_share> <mount_dir>/<project_folder>/

where **\ :raw-html-m2r:`<user>`\ ** is the user name of the server and **<server_url>** is the is something like `\ *login-gpu.lisa.surfsara.nl* <http://login-gpu.lisa.surfsara.nl>`_\ _, **<dir_to_share>** is the directory to be shared and **<project_folder>** is a subdirectory for the project in the image-storage drive under the Group Teuwen folder.

If transferring from using tunneled ssh connection (e.g.: ssh to rhpc and then to wallace on rhpc) then this command works:

.. code-block:: bash

   rsync -azv -e 'ssh -A -J <user>@rhpc.nki.nl' --info=progress2 \
    <user>@rhpc-wallace:<dir_to_share> <mount_dir>/<project_folder>

Note that if you want to transfer from e.g. Lisa or your local machine TO the NKI server, you need to switch the directories in the above statement, like

.. code-block:: bash

   rsync -azv -e 'ssh -A -J <user>@rhpc.nki.nl' --info=progress2 \
    /path/to/local/<dir_to_share> <user>@rhpc-wallace:<mount_dir>/<project_folder>

**Important note**\ : All files to be rsync-ed by a user need to have permissions rights for user **at least read (for files) and executable (for folders and subdirectories)** permissions\ **.** Check `here <https://www.linode.com/docs/guides/modify-file-permissions-with-chmod/>`_ for more information on permission rights.

Sharing large files with externals
==================================

A secure way to share large files is provided by Surf: `https://www.surf.nl/en/surfdrive-store-and-share-your-files-securely-in-the-cloud <https://www.surf.nl/en/surfdrive-store-and-share-your-files-securely-in-the-cloud>`_

Transferring files from surf filesender using the command line
==============================================================

The ``curl`` command can be used, but it requires two properties for the url (specific for surf filesender):


* 
  Tag ``download.php`` should be included in the target domain (and not “s=download”)

* 
  The file ID(s) should be included for download

In the download page, get link address for downloading one or multiple files as zip or tar, and use the curl command as in the example below:

.. code-block:: bash

   curl -o data.tar 'https://filesender.surf.nl/download.php?token=7f9aad80-b9ce-43af-b7cc-863c14a8b8cd&files_ids=5610281%5610282'

Document storage
================

Make sure to save your important documents in a backed-up location. The NKI provides you with a OneDrive account.

Code storage
============

We use GitHub: `https://github.com/NKI-AI <https://github.com/NKI-AI>`_ access can be obtained through Jonas Teuwen or Yoni Schirris
