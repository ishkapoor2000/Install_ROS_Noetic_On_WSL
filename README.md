# Install_ROS_Noetic_On_WSL
### Install ROS Noetic using WSL

## 1. Update Windows 10
A recent version of **Windows 10** needs to be installed for **ROS** to work. To check which version is installed go to

`Settings -> System -> About`.

Check the version and Update Windows 10 (Version 20H2 or later is recommended for optimal performance).

## 2. Install WSL
Install WSL on your system via [MicrosoftStore](https://www.microsoft.com/en-in/p/ubuntu/9nblggh4msv6?activetab=pivot:overviewtab) or via [this](https://docs.microsoft.com/en-us/windows/wsl/wsl2-kernel) (follow *Step 4 - Download the Linux kernel update package*).

## 3. Set WSL 2 as your default version
This is important for you to install **ROS via WSL**. You need an *updated WSL*.

Open PowerShell and paste the following command:
```bash
wsl --set-default-version 2
```
Now, paste the following command:
```bash
wsl -l -v
```

There are two possibilities of the outcome of output. If the output after `wsl -l -v` is:
```bash
 NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         2
```

Then, **congratulations**. It was a successful step.
But if the output is something like:
```bash
 NAME            STATE           VERSION
* Ubuntu-20.04    Stopped         1
```

Then you have **not** updated accurately as the version is still *1*. You should then download WSL via [this](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) link.

## 4. Installing ROS on WSL
Have you tried installing ROS on WSL 1? If yes, then, you might have to **purge** those files and *reinstall ROS on WSL 2*. But if you did not, then, let's continue ahead.

For **Ubuntu 20.04**, you might have to install *ROS Noetic*.
I simply followed the guide [here](https://janbernloehr.de/2021/02/28/ros-windows-update) by [*Jan Bernlöhr*](https://janbernloehr.de/).

Let us follow few steps:

Paste the following commands in WSL 2:
```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
curl -sSL 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0xC1CF6E31E6BADE8868B172B4F42ED6FBAB17C654' | sudo apt-key add -
sudo apt update
sudo apt install -y ros-noetic-desktop python3-rosdep
sudo rosdep init
rosdep update
```

***Warning***: *If you encountered any error, just simply close the current WSL and open a new one. Then paste these commands one by one at a time.*

To *source ROS Noetic automatically* forever bash session, paste the following commands in WSL.
```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

To check if the above step was successful, open .bashrc file with **`nano`** command. Paste this command `nano ~/.bashrc`. This command will open the file inside WSL. Go at the bottom. If you find *```source /opt/ros/noetic/setup.bash```* there. **Congratulations**! These steps were successful.

If not, then, type it manually there and save it `ctrl+x -> y -> Enter`.

## 5. Testing Installation of ROS
Open *three new WSL bash prompts*.
* In the first WSL bash, run the following commands:
```bash
cd
roscore
```

* In the second WSL bash, here, we will create a new python file with the name **`publish.py`** with "touch" and edit it using the **`nano`** command.
Paste the following commands:
```bash
touch publish.py
nano publish.py
```

When the file opens in editor mode, paste the following Python Code:
```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import String
   
pub = rospy.Publisher('chatter', String, queue_size=10)
rospy.init_node('talker', anonymous=True)
rate = rospy.Rate(10) # 10hz
while not rospy.is_shutdown():
   hello_str = "hello world %s" % rospy.get_time()
   rospy.loginfo(hello_str)
   pub.publish(hello_str)
   rate.sleep()
```

Save the file `ctrl+x -> y -> Enter`. Now, run the file using **`python3 publish.py`**. If it ran successfully, then **congratulations**!

* In the third WSL bash, we will create another python file with the name **`subscribe.py`** with **`touch`** and edit it using the **`nano`** command.
Paste the following commands:
```bash
touch subscribe.py
nano subscribe.py
```
* When the file opens in editor mode, paste the following Python Code:
```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import String

def callback(data):
    rospy.loginfo(rospy.get_caller_id() + "I heard %s", data.data)

def listener():
    rospy.init_node('listener', anonymous=True)
    rospy.Subscriber("chatter", String, callback)
    # spin() simply keeps python from exiting until this node is stopped
    rospy.spin()

if __name__ == '__main__':
    listener()
```
Save the file `ctrl+x -> y -> Enter`. Now, run the file using **`python3 subscribe.py`**. If it ran successfully, then **congratulations**!

This step ensures that we have *successfully installed **ROS using WSL***.

## 6. Setting up ROS for Graphical Output (GUI)
To run applications with graphical output, you need to install an **X Server** on Windows. To me, [**VcXsrv**](https://sourceforge.net/projects/vcxsrv/) (*recommended*) works best.

After you have installed VcXsrv, you also need to *configure WSL* to use it. To do so modify you *.bashrc* as follows:
```bash
export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
source ~/.bashrc
```
**OR**

You can simply add `DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0` inside *.bashrc* file (as mentioned in step-4).

Finally, launch VcXsrv from the start menu. You need to change the following two settings:
* Native OpenGL needs to be **unchecked**. Otherwise, applications such as *rviz* do not run as expected.
* Disable access control needs to be **checked**. Otherwise, applications within WSL cannot access the x server.
* When prompted by the Windows Firewall make sure that you enable access for *both private and public networks* - unless you are 100% sure that you only want to use the **x server** when connected to private networks.

***Note***: These settings might look different depending on when in the future you have downloaded VcXsrv.

## 7. Running Applications with Graphical Output (GUI)

The popular **turtle_sim** tutorial works fine WSL as well.

Make sure you have an *X Server* (eg: VcXsrv) installed, configured (checked and unchecked), and running as described above👆.

Start a new bash prompt and run the following command:
```bash
cd
roslaunch turtle_tf turtle_tf_demo.launch
```
You can *control the turtle* by using the **arrow keys** by going back to the second prompt.

**RVIZ**: Start a new WSL bash and run the following command:
```bash
cd
rosrun rviz rviz -d `rospack find turtle_tf`/rviz/turtle_rviz.rviz
```
If everything ran successfully, then **BOOM**. You've *finally done it*!

## 8. Useful tools
*Note*: The below-mentioned reference is from [QSTP_Robot-Automation-using-ROS_2021](https://github.com/ERC-BPGC/QSTP_Robot-Automation-using-ROS_2021/blob/main/WEEK%201/Preparing%20your%20Development%20Environment.md).

* **Git**: Fundamental tool in open source software development. Used for version control and sharing of code. To install, run the following command:
```bash
sudo apt install git
```
* **Terminator**: Terminal Emulator useful for having multiple terminals in a window. To install, run the following command:
```bash
sudo apt install terminator
```
* **Code Editors**: A good editor can go a long way in boosting productivity. We recommend **VSCode** which has plugins for Python and ROS.

* **catkin_tools**: A better alternative for catkin_make.

## 9. Creating Workspace
**Note**: *For Ros Noetic and above, you will have to build the Turtlebot package from the source*.

Try the following command to see, if the workspace is available or not:
```bash
cd ~/catkin_ws/src
```
If you have not yet set up your catkin workspace, you can do that by:
```bash
cd
mkdir -p catkin_ws/src
```
Here clone the following repositories using 'Git'.
```bash
git clone https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
```
## 10. Building Workspace

Paste the following command:
```bash
cd ..
catkin_make
```

Now, Add the following commands in *.bashrc* manually or using **`echo`** command (as mentioned in step-4):
* ```export TURTLEBOT3_MODEL=burger```
* ```source ~/catkin_ws/devel/setup.bash```

Test your **Turtlebot** and **Gazebo** setup by launching a sample launch file using the following command:
```bash
roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch
```
#### If you see the Turtlebot in the gazebo environment, then my friend, the installation was successful! Take a break and enjoy a cup of coffee.

## License
[MIT](https://choosealicense.com/licenses/mit/)
