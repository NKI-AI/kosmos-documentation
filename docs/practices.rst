.. _practices:

=============================================
Training/ Inference on Cluster Best Practices
=============================================

Introduction
------------

When training machine learning models on a node in the cluster, it's essential to follow best practices to
ensure efficiency and to avoid unnecessary network congestion.
This page outlines recommended procedures for handling data and setting up your training jobs on the cluster.

Each machine in the cluster is equipped with fast SSD storage in RAID-6, mounted in `/processing/`. The `/processing/`
directory is the scratch disk, which is local to the machine.
When you log in, `/processing/<username>` is created with permissions set to 700, meaning that no other
regular users can read your data. You can reference this directory using the environment variable `$SCRATCH`,
eliminating the need to hardcode paths.

However, please note that the `/processing/` disk may be periodically wiped (oldest files first) if it starts to get full.

1. Data Preparation
--------------------

Before you start your training job, it's crucial to copy your training data to the local scratch (SSD)
disks on the GPU node you'll be using. This ensures that data is readily accessible to the GPU,
reducing data transfer overhead and enhancing training performance.

    - **Step 1**: Copy your training data to the scratch disk using `rsync`:

      ```
      rsync -avv --info=progress2 --delete <SOURCE> $SCRATCH
      ```

      or

      ```
      rsync -avv --info=progress2 --delete <SOURCE> /processing/<username>/
      ```


2. Training Process
---------------------

When setting up your training job, it's recommended to "train from scratch" using the local data on the scratch disk, rather than "training from network." Training from the network can lead to slower training times and increased network traffic, which can affect other users on the cluster.

    - **Training from Scratch**: Use the data available on the local scratch disk to train your machine learning models. This approach is faster and more efficient.

      Example command for training from scratch:

      ```
      python train.py --data $SCRATCH/training_data
      ```

      or

      ```
      python train.py --data /processing/<username>/training_data
      ```

    - **Training from Network**: **Avoid** this method if possible, as it can be slow and may negatively impact cluster performance.

      Example command for training from network:

      ```
      python train.py --data /path/to/your/network/data
      ```

We urge everybody to ensure they have near-optimal GPU utilization (near 100%) on the A6000 and A100 GPUs. If you cannot
get your code optimized to achieve this, please use an RTX node instead. Note that if you want to debug with 48GB of
VRAM, you should preferably use roentgen (slower 48GB gpus than A6000).

3. Inference
-------------

When it comes to running inference, it's generally acceptable to perform inference from network storage.
Inference typically involves smaller data transfer and is less resource-intensive compared to training.
This means that you can retrieve your model and input data directly from network storage when performing inference tasks.

If you run inference with less than 11GB VRAM needed, please use a 2080ti for inference.

4. Best Practice
-----------------

Following these best practices ensures efficient use of cluster resources and a better training experience for you and other users. Be mindful of managing your data and job setup to maximize the capabilities of the GPU nodes while minimizing network traffic.

Remember to clean up any temporary data or files when your job is finished to keep the scratch disks available for others. Avoid storing important training artifacts on the `/processing/` drive, as these can be deleted without warning. Use project data folders for persistent storage.

The processing folders of the GPU nodes are mounted on kosmos/gaia in `/mnt/processing/<hostname>`, so you can inspect the folders without needing to schedule a job on a compute node.

Conclusion
-----------

By moving your data to the local scratch (SSD) disks on the GPU node and training from scratch, you can maximize training performance and maintain the integrity of the cluster environment. These best practices contribute to a smoother training experience and faster results.

Feel free to contact the cluster administrators if you have any questions or need further assistance in optimizing your training jobs on the cluster.