# NxR's Baxter Package

This package contains most of the work that has been done by the Neuroscience and Robotics Laboratory with our Baxter Research Robot (referred to internally as RJ). 

RJ has been installed in a public space at Northwestern University and is currently running a demo that allows a user to pick one of two games. The first game is the "mime" game, and the second is the "crane" game. They are described below.

Since RJ is in a public setting, his software is designed to run 24/7. There are some problems with this, primarily with the use of an Asus Xtion to interact with users. Over time, the performance of the Asus can degrade and it will often not start up properly. Since much of the code for this device is closed source and proprietary, we have developed a few work arounds which we describe below.

The only command needed to launch the games is:
```
roslaunch nxr_baxter startup.launch
```

## Mime Game
To start this game, the user raises their right arm. After doing so, they match their arms with RJ's by raising them to their sides. Once the user has done this, the robot will mimic the user's arm motions.

In reality, the robot is only mimicking the users shoulder and elbow angles. 

The `mimic.py` file contains the description for the `Mime` class. This class provides an interface for playing the mime game described above. The only function that should be used is `Mime().desired_joint_vals(left_shoulder, left_elbow, left_hand, right_shoulder, right_elbow, right_hand)` which takes in tfs to the skeleton data for the human, and returns the joint values for the robot to achieve to mimic the user. None of the other functions besides the constructor should be used.

## Crane Game
To start this game, the user raises their left arm. After doing so, they match their left arm with RJ's right by raising it to their sides. Once the user has done this, the robot will mimic the user's left arm motions. By bending their right arm at the elbow, they can control RJ's left gripper. This allows for picking and placing objects. The goal is to pick up and move around various blocks on the table placed in front of RJ.

Since the robot is only mimicking the shoulder and elbow angles, it can be difficult to actually pick up objects. We are currently working on implementing a cartesian based planner instead of a configuration based planner.

The `crane.py` file contains the description for the `Crane` class. This class provides an interface for playing the game. The class has two functions that are primarily accessed once an instance has been created. The two are `desired_joint_vals(left_shoulder, left_elbow, lefthand, right_shoulder, right_elbow, right_hand)` and `desired_pos_vals()` with the same input arguments. The inputs in this case are the TFs for the person at each of those joints as given by the skeleton tracker. Both functions check if the right arm's elbow has been bent and will close the gripper if that is the case. The `desired_joint_vals` function will return the desired baxter arm configuration to match the human's arm, and `desired_pose_vals` will return the desired end effector pose. This latter function is the configuration based planner described above, and is not currently working. The rest of the functions beyond the constructor should not be needed.


## Workarounds for Asus Xtion
After being on for a long time, the Asus will start to slow down the frequency at which it publishes skeletons.

### USB Restart
Sometimes, restarting the usb interface (which is equivalent to disconnecting and reconnecting) will solve the problem. In order for this to work, the drivers also need to be shut down and restarted. This is taken care of in the `tracker_heartbeat.py` script. By using a modififed version of the skeletontracker_nu which publishes an empty message every time it publishes a skeleton, the tracker_heartbeat class can monitor the frequency of the Asus. When the frequency drops below some threshold (~24Hz, normally 30Hz) it will run the `restart_usb.sh` script. Note that for this script to work, the permisions for binding and unbinding the usb drivers needs to be set properly. 

The `tracker_heartbeat.py` script also handles starting up and shutting down the drivers and skeleton tracker. Fairly often, these will not start up properly. Sometimes running `restart_usb.sh` will fix that, and often it will not.

### Computer Restart
When the `restart_usb.sh` script has been called 5 times, `tracker_heartbeat.py` will restart the computer to try to solve the problems. This is handled in another workaround fashion. There is a file that is being monitored by a background script running on the computer every second. Whenever that script sees a 1 in the file instead of a 0, it will restart the computer. The reason is that the script needs to be run as root at startup in order to force the system to go down.

One problem with this is that sometimes the computer gets stuck on BIOS which is a huge pain. Someone then needs to go in and start it manually, I think the problem is with the keyboard.

##Images
Note that images are not uploaded as they were included in the .gitignore. However, `image_files.txt` is quite important, and indicates the locations of all the images. The `ImageSwitcher` class as defined in `image_switcher.py` reads in that file at the start, with the syntax for that file defined in the classes comments for its constructor. The file should be a list of modes of execution followed by a list of filenames that the ImageSwitcher should cycle between when in that mode. 


#Other Scripts
- `camera_arm.py` is an example of how to use the hand and head cameras. Some of its functions may have been deprecated

- `clap.py` provides a class for "clapping" where RJ brings his arms together and then apart

- `cv_sandbox.py` was used as a testbed for learning opencv alongside RJ

- `grab.py` asks a user to give RJ an object and then passes it from one hand to the other. Doesn't look like it was ever finished

- `launcher.py` was the old way of starting up the demo. Now we are using launch files, see above the top of the readme

- `meta_mode_controller.py` contains the `MetaMode_Controller` class that handles the service calls to request a mode change as well as the topic to publish about mode changes. The idea is one node sends a service call to request the mode to change, if its possible, then the class publishes to a latched topic the new mode and all nodes listening have a callback to handle changes in mode. This allows all nodes to have a synchronized mode without requiring polling the parameter server.

- `restart_usb.sh` is the script that unbinds and rebinds the usb device to effectively unplug and replug the Asus

- `skeleton_filter.py` is the filter that Jarvis wrote with Jon to filter out noise from the Asus. This comes from a white paper that Microsoft published.

- `test_moveit.py` is a script I (Adam) wrote to try and get MoveIt working with Cartesian space stuff. I used it to post questions about how to do this on the brr-users group as well as the moveit-users group. It now contains testing for doing Inverse Kinematics to get around using moveit cartesian space planning.

- `test_skel_distance.py` is a script that prints someone's x, y, and z locations according to the Asus to try and clip the space that the Asus sees.

- `the_robot.py` provides `The_Robot` class which does the "robot" dance.

- `trajectory.py` contains the `Trajectory` class which allows an easier interface to Trajectory objects. I dont think we actually ended up using this.

- `vector_operations.py` contains a number of useful functions for dealing with 3 dimensional quantities


