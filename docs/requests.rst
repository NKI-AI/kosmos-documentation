===============
Kosmos requests
===============

.. contents::

Project folders
---------------

You can request a project folder in ``/projects`` with a default storage quota of 100G. If you expect that your project will require more than 100G of storage, you can apply for larger quota, in which case you will need to provide a short explanation. Typically, this should include a back-of-the-envelope calculation demonstrating why you expect that 100G is insufficient. Non-specific reasons like "to store checkpoints" are not enough: explain how large you expect individual checkpoints to be and why you need to store x amount of checkpoints. You probably need fewer than you think.

Please use the following template and post it in the ``#tech-kosmos-requests`` channel. 

    **Project name:** <project name>

    **Quota:** <desired quota (default 100G)>

    **Does it require backups:** <yes/no (default no)>

    **Username:** <your username>

    Optional:

    **Reason above 100G:** <reason>

Choose a recognizable and sufficiently unique project name. Examples of good project names would be ``dino_duct_segmentation`` and ``cbct_noise2inverse``, while examples of bad project names would be ``john2022`` or ``unet``.

Note that the backups mentioned in the request template are *snapshots* that are stored on the same filesystem: you can use them to recover files that you have accidentally deleted/overwritten (see the :ref:`data management<data_management>` page for instructions), but the files will still be gone if our hardware fails. We are working on providing tape backups to guard against such failures.


Extra resource allocations
--------------------------

If you have an approaching deadline and you fear that you cannot complete all your computational work in time, you can **temporarily** request more compute resources. For example, you may get access to four a6000 GPUs (instead of just two) for two weeks if that helps you complete your work in time. Do keep in mind that this is a shared cluster, and these elevated rights should be seen as a privilege. Accordingly, such rights will only be granted if you provide a **very** good reason, and if the request is reasonable with respect to that reason.

Please use the following template and post it in the ``#tech-kosmos-requests`` channel. 

    **Desired extra resources:** <e.g. two extra a6000 GPUs>

    **Timeframe:** <e.g. "15 July 2023 - 30 July 2023", or "starting ASAP until ...">

    **Username:** <username>

    **Reason:** <be precise and provide relevant information, like an abstract submission date, and what tasks you need to complete before that time>
