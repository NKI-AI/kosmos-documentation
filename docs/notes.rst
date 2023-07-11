=============
Cluster Notes
=============


.. note::

    **GALILEO NODE IN PARTITION A6000**

    The `galileo` node in partition `a6000` is currently experiencing issues with multiprocessing functionality.
    We apologize for the inconvenience caused.

    **Status**: Currently not working for multiprocessing jobs (gpus > 1).

    **Expected Fix**: The issue is expected to be resolved in the upcoming patch on Monday.

    **Workaround**

    A quick workaround is available to address this issue. You can modify your command by disabling the peer-to-peer transport in the NCCL backend.
    Please follow the steps below:

    1. Modify your command by adding the environment variable `NCCL_P2P_DISABLE=1` before your actual command.
       For example:

       ::

            export NCCL_P2P_DISABLE=1
            export CUDA_VISIBLE_DEVICES=0,1
            python3 pytorch_file.py

    2. Run your command with the modified configuration.

    This workaround will help you bypass the multiprocessing issue on the `galileo` node until the fix is applied.

    Link to suggested workaround: `Suggested Workaround <https://discuss.pytorch.org/t/torch-distributed-init-process-group-hangs-with-4-gpus-with-backend-nccl-but-not-gloo/149061/2>`_.

    Please feel free to reach out to the system administrators for further assistance.
