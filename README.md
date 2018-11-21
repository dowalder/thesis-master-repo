# thesis-master-repo
Collects different repositories belonging to my master thesis project. All subrepos are described here, as some of them
are forks and the original README is not changed.

## gym-duckietown

The [duckietown gym](https://github.com/duckietown/gym-duckietown) is a simulation environment for Duckietown.
The following functionality was added:
* Pytorch code to train a lane following agent from the simulation data.
There is code for data extraction, training and testing. Multiple networks are implemented, experimenting with making
the trained network more stable towards disturbances. Some experiments where done to train a network to estimate 
disturbances, but this came never to a stage where it worked.
* Several utility scripts in ./scripts. The descriptions in the files themselves provide more information.

## 
