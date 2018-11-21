# thesis-master-repo
Collects different repositories belonging to my master thesis project. Because several forks were used, merging them
into one repository is not desirable.

## gym-duckietown

[repo](https://github.com/dowalder/gym-duckietown)

The duckietown gym is a simulation environment for Duckietown.
The following functionality was added:
* Pytorch code to train a lane following agent from the simulation data.
There is code for data extraction, training and testing. Multiple networks are implemented, experimenting with making
the trained network more stable towards disturbances. Some experiments where done to train a network to estimate 
disturbances, but this came never to a stage where it worked.
* Several utility scripts in ./scripts. The descriptions in the files themselves provide more information.

## Duckietown Software

[repo](https://github.com/dowalder/duckietown_software/tree/dowalder/master) (Make sure you are on the dowalder/master
branch)

This is the main software repository. Added from our side are two packages:
* [cnn_lanefollowing](https://github.com/dowalder/duckietown_software/tree/dowalder/master/catkin_ws/src/10-lane-control/cnn_lanefollowing):
Contains a single node that can be remotely connected to ROS running on a duckiebot. It creates ZeroMQ queues to 
communicate with scripts that do not run in ROS. It was created because a lot of our code is written in python3.
* [analyze_logs](https://github.com/dowalder/duckietown_software/tree/dowalder/master/catkin_ws/src/00-infrastructure/analyze_logs):
Provides 3 nodes that are used to extract images from rosbags. They are only ros nodes because we need access to the 
ros libraries. They do not use any duckietown utilities and can therefore be run independently of this repo, 
in contrary to cnn_lanefollowing.

## pix2pix

[repo](https://github.com/dowalder/pytorch-CycleGAN-and-pix2pix)

All changes that were only for experiments, of which none had been fruitful. For training the networks we present in our
paper, the original framework was used. The procedure to train a network is as follows:

* To create the training data, we used the above mentioned duckietown gym. All scripts can be found in the gym repo. 
You will find two scripts there that are
useful: `generate_data.py` to extract images from the simulation. An example for a working conf file is the 
`datagen_domainrand.yaml`. And then `creator.py` which creates the augmented dataset. in addition to the simulation data
you will need a directory containing background images. These can be basically anything, we used a few hundred indoor
photographs.
* To bring the data in the correct form for pix2pix use the scripts provided there (e.g. `datasets/make_dataset_aligned.py`)
* To train the network use the following command: 
``python train.py --dataroot path/to/data --name your_name_here  --model pix2pix``
* If you can, use multiple gpus and a high batchsize (e.g. `--batchSize 32 --gpu_ids 0,1,2,3`). Then training only takes
a few hours.

## UNIT

[repo](https://github.com/dowalder/UNIT)

Only minor changes were applied here. We added the possibility to use coordconv layers instead of normal conv, but could
not see an improvement for our case.

