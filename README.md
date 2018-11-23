# thesis-master-repo
Collects different repositories belonging to my master thesis project. Because several forks were used, merging them
into one repository is not desirable.

## gym-duckietown

[repo](https://github.com/dowalder/gym-duckietown)

The duckietown gym is a simulation environment for Duckietown. Since most of the work started here, all utility 
scripts are collected here in a script folder. Additionally to what was presented in the thesis, there is code to train
a lane following agent from the simulation data in there. Multiple networks were implemented while trying to make
the trained network more stable towards disturbances. Some experiments where done to train a network to estimate 
disturbances, but this came never to a stage where it worked.

To get data from the simulator, use `generate_data.py`. This script uses configuration files to set everything up. 
`conf/datagen_domainrand.yaml` is a good example. Next to the images, a .yaml file is outputted that contains information
about all images (action, etc), but this is only important to train lane following agents, which was in the end not part
of this thesis.


## Duckietown Software

[repo](https://github.com/dowalder/duckietown_software/tree/dowalder/master) (Make sure you are on the dowalder/master
branch)

This is a fork of the main Duckietown software repository. Added from our side are two packages:
* [cnn_lanefollowing](https://github.com/dowalder/duckietown_software/tree/dowalder/master/catkin_ws/src/10-lane-control/cnn_lanefollowing):
Contains a single node that can be remotely connected to ROS running on a duckiebot. It creates ZeroMQ queues to 
communicate with scripts that do not run in ROS. It was created because a lot of our code is written in python3.
* [analyze_logs](https://github.com/dowalder/duckietown_software/tree/dowalder/master/catkin_ws/src/00-infrastructure/analyze_logs):
Provides 3 nodes that are used to extract images from rosbags. They are only ros nodes because we need access to the 
ros libraries. They do not use any duckietown utilities and can therefore be run independently of this repo, 
in contrary to cnn_lanefollowing.

### Running the pipeline

First familiarize yourself with the Duckiebot and try to run the `make demo-lanefollowing` command. At the time of writing
this, the Duckietown software was being ported to docker. The explanation here does not use docker. You can use our fork of 
duckietown/Software if you struggle to make it run on docker.

To run the pipeline, the only thing we need is to transform the image before it is sent to the lane following controller.
However, since the transformation cannot be done on the bot (Raspberry Pi and neural networks do not go well together)
we sent the image to an external computer with a gpu and then back. To eliminate problems with different python versions
(ROS uses python2.7), we implemented another layer of indirection, such that a ROS node communicates through ZeroMQ with
another script that performs the actual transformations. This makes it a somewhat complicated process.

Perform the following steps to run it (you need a working setup of the duckietown software repo on both the bot and the 
external computer):

1. On the bot: 
    1. Make sure your camera outputs 120x160 images. Larger images will introduce a too large delay when 
    sending them over the network. You can adjust that in 
    catkin_ws/src/00-infrastructure/duckietown/config/baseline/pi_camera/camera_node/default.yaml.

    2. The anti-instagram filter scales down the images. Since we already have a size of 120x160 that is unnecessary.
    Checkout the  ImageTranformerNode() in catkin_ws/src/10-lane-control/anti_instagram/src/image_transformer_node.py. 
    If it has a member `self.scale_percent`, set it to 100 (it might not have such a member, that is fine too).
    
    3. Change the input topic of the `image_transformer_node.py` to `neural_style/corrected`. Normally, this is done
    in a launch file (e.g. master.launch). However, in the version we used this had to be set within the python file
    (catkin_ws/src/10-lane-control/anti_instagram/src/image_transformer_node.py).
    
    4. Run `make demo-lanefollowing`

2. On the external computer:
    1. Run the `ros_wrapper.py --mode img_to_img` file in the gym-duckietown repo. Within the file, set the correct path
     to the model you
    want to use and choose a path to the zeromq topics. You need to set the same topics in the `neural_style.py` node to 
    make the zeromq communication work. See command line parameters for more options.
    1. Change to another terminal and go the duckietown folder
    1. Run `source set_ros_master.sh YOUR_BOTS_NAME`
    2. Run `roslaunch cnn_lanefollowing neural_style.launch veh:=YOUR_BOTS_NAME local:=true`
    1. Both scripts continuously output some stuff if it works. If neural_style.launch does output nothing, make sure 
    the lane following runs on you the bot and the connection to the bot works. If you have output on the ros node but
    not on the gym-duckietown script, the zeromq communication fails. Make sure the topics in both scripts point to the
    same location. If yes, restart the ros node. Sometimes it needs several restarts until the connection works.
    1. In yet another termial, run (in the duckietown folder) `make virjoy-YOUR_BOTS_NAME`. Now you can steer the bot
    with the keyboard and toggle between autonomous and manual mode.


## Neural Style Transfer

[repo](https://github.com/dowalder/mind_the_gap)

Here we implement the neural style transfer model. To train a model, first download the COCO dataset (we used the COCO
train 2014 set) or a similar one. Then training is straight forward: 

``python train.py --dataset path/to/COCO --style-image path/to/style_image --cuda 0``

If you want to train on multiple style images, `--style-image` needs point to a folder containing multiple images and
you need to set `--use-multiple-styles`. Set the `--batch-size` to the number of style images if there are not too many
(< 30) and you have enough memory. Otherwise set the `--random-sample-batch` flag. This will sample the style images at
every iteration from the specified folder. Checkout the command line parameters for more options.

## pix2pix

[repo](https://github.com/dowalder/pytorch-CycleGAN-and-pix2pix)

All changes were only for experiments, of which none had been fruitful. For training the networks we present in our
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

The basic structure for the unit training was not changed. We added the possibility for a content loss and coordconv
layers.

## FID

To compute the FID, we used [this repo](https://github.com/mseitzer/pytorch-fid). No changes were applied, so we did not
fork it. Use the `run_comparisons.py` script from the gym-duckietown repository to facilitate the computation of FID 
between many datasets.



