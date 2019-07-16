



# <center>ROS程序包编写</center>

<big>[官方教程](http://wiki.ros.org/cn/ROS/Tutorials)</big>


## 此处我们以一个实例阐述ROS程序包的编写

### 目标
* 创建一个消息发布器，向话题“Brake”发布“control_node_msgs/BrakeCmd”类型的自定义消息，自定义消息格式为：

    > Header header
    > float32 brake_cmd

* 创建一个消息订阅器，订阅话题“Brake”的消息，并输出到屏幕。
消息发布器和消息订阅器均使用C++实现，并使用 roslaunch启动

### 演示环境：
* 
    * Ubuntu 16.04.6 LTS (Xenial) amd64
    * ROS Kinetic
    * catkin

### 操作步骤

#### 1.创建程序包
- 在当前用户的HOME目录下创建catkin工作目录

    ```
    $ mkdir ~/catkin_ws
    ```

- 进入catkin工作目录，创建并进入src源码文件夹

    ```
    $ cd ~/catkin_ws
    $ mkdir src
    $ cd src
    ```

- 现在使用catkin_create_pkg命令来创建一个名为'control_node_msgs'的新程序包(根据我们的目标进行命名)，这个程序包依赖于std_msgs、roscpp和rospy：

    ```
    $ catkin_create_pkg control_node_msgs std_msgs rospy roscpp
    ```
    
   > 这将会在 ~/catkin_ws/src 目录下创建一个名为control_node_msgs 的文件夹，这个文件夹里面包含一个package.xml文件和一个CMakeLists.txt文件，这两个文件都已经自动包含了部分你在执行catkin_create_pkg命令时提供的信息。

#### 2.创建自定义消息
- 创建并进入msg文件夹

    ```
    $ mkdir ~/catkin_ws/src/control_node_msgs/msg
    $ cd ~/catkin_ws/src/control_node_msgs/msg
    ```
- 创建 .msg 的自定义消息文件

    ```
    $ touch BrakeCmd.msg
    ```

- 在 BrakeCmd.msg 文件中根据目标写入如下信息

    ```
    Header header
    float32 brake_cmd
    ```
    
    >Header类型详细参考
    [std_msgs/Header Message](http://docs.ros.org/api/std_msgs/html/msg/Header.html)
    此处简略说明，即Header包含3个成员
    uint32 seq
    time stamp
    string frame_id
    其中stamp包含 stamp.sec 和 stamp.nsec，这二者均为integer，前者为秒，后者为毫秒
    floa32类型即为浮点型

- 查看package.xml, 确保它包含下面两条语句，如果不存在，请自行添加

```
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>
```

- 修改CMakeLists.txt文件
    * 打开CMakeLists.txt，找到find_package()代码块，最初应该显示如下

        ```
    	find_package(catkin REQUIRED COMPONENTS
		  roscpp
		  rospy
		  std_msgs
		)
		```

	* 在加上一项message_generation，即将其变为

		```
		find_package(catkin REQUIRED COMPONENTS
		  roscpp
		  rospy
		  std_msgs
		  message_generation
		)
		```

	* 找到catkin_package()代码块，开始时应该显示如下

		```
		catkin_package(
		#  INCLUDE_DIRS include
		#  LIBRARIES control_node_msgs
		#  CATKIN_DEPENDS message_generation message_runtime roscpp rospy
		#  DEPENDS system_lib
		)
		```

	* 加上 CATKIN_DEPENDS message_runtime ,使其变为如下

		```
		catkin_package(
		#  INCLUDE_DIRS include
		#  LIBRARIES control_node_msgs
		#  CATKIN_DEPENDS message_generation message_runtime roscpp rospy
		#  DEPENDS system_lib
		  CATKIN_DEPENDS message_runtime
		)
		```

	* 找到如下代码块

		```
		# add_message_files(
		#   FILES
		#   Message1.msg
		#   Message2.msg
		# )
		```
	
	* 将其改为如下代码，其中.msg为刚才创建的.msg文件
	
		```
		 add_message_files(
		   FILES
		   BrakeCmd.msg
		 )
		```

	* 再找到如下代码块

		```
		# generate_messages(
		#   DEPENDENCIES
		# #  std_msgs  # Or other packages containing msgs
		# )
		```
	
	* 去掉注释，修改为如下

		```
		 generate_messages(
		   DEPENDENCIES
		   std_msgs  # Or other packages containing msgs
		 )
		```
- 此时就可以编译一下，查看自定义消息是否成功，编译及检查自定义消息步骤如下
	* 回到catkin的工作目录
	
		```
		$ cd ~/catkin_ws
		```
	
	* 编译

		```
		$ catkin_make
		```
	
	* 查看自定义消息，即检查

		```
		$ rosmsg show control_node_msgs/BrakeCmd
		```
	
	* 如果一切顺利，应该会显示如下信息

		```
		std_msgs/Header header
		  uint32 seq
		  time stamp
		  string frame_id
		float32 brake_cmd
		```
	
	* 如果提示找不到这个消息类型，则需要执行如下操作

		```
		$ source ～/catkin_ws/devel/setup.bash
		```

	* 上述操作适用于临时使用，想一劳永逸的话，请使用如下操作

		```
		$ echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
		$ source ~/.bashrc
		```

	* 此时再用上面提到的命令就能查看到自定义消息类型了，如果创建自定义消息成功，

#### 3.创建消息发布器和消息订阅器

- 进入程序包目录

	```
	$ cd ~/catkin_ws/src/control_node_msgs
	```

- 创建并进入 src 目录

	```
	$ mkdir src
	$ cd src
	```

- 分别创建 talker.cpp 和 listener.cpp 文件，对应 消息发布器 和 消息订阅器

	```
	$ touch talker.cpp
	$ touch listener.cpp
	```
- 下面给出 talker.cpp 和 listener.cpp 文件的源代码，代码说明请参考 [编写简单的消息发布器和订阅器 (C++)](http://wiki.ros.org/cn/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29)
	* talker.cpp

		```
		#include "ros/ros.h"
		#include "std_msgs/String.h"
		#include "control_node_msgs/BrakeCmd.h"
		#include <sstream>
		#include <string.h>
		int main(int argc, char **argv)
		{
		    ros::init(argc, argv, "talker");
		    ros::NodeHandle n;
		    ros::Publisher chatter_pub = n.advertise<control_node_msgs::BrakeCmd>("Brake", 1000);
		    ros::Rate loop_rate(10);
		    int count = 0;
		    float brake_cmd = 6.66;
		
		    while (ros::ok())
		    {
		        control_node_msgs::BrakeCmd msg;
		        msg.header.seq = count;
		        msg.header.stamp = ros::Time::now();
		        msg.header.frame_id = "BrakeCmd";
		        msg.brake_cmd = brake_cmd;
		        ROS_INFO("talker/\n");
		        ROS_INFO("  Header/\n");
		        ROS_INFO("    seq [%d]\n", msg.header.seq);
		        ROS_INFO("    stamp [%d]\n", msg.header.stamp.sec);
		        ROS_INFO("    frame_id [%s]\n", msg.header.frame_id.c_str());
		        ROS_INFO("  brake_cmd [%f]\n", msg.brake_cmd );
		        chatter_pub.publish(msg);
		        ros::spinOnce();
		        loop_rate.sleep();
		        ++count;
		    }
		    return 0;
		}
		
		```
		
	* listener.cpp

		```
		#include "ros/ros.h"
		#include "std_msgs/String.h"
		#include "std_msgs/String.h"
		#include "control_node_msgs/BrakeCmd.h"
		
		void chatterCallback(const control_node_msgs::BrakeCmd& msg)
		{
		    ROS_INFO("listener/\n");
		    ROS_INFO("  Header/\n");
		    ROS_INFO("    seq [%d]\n", msg.header.seq);
		    ROS_INFO("    stamp [%d]\n", msg.header.stamp.sec);
		    ROS_INFO("    frame_id [%s]\n", msg.header.frame_id.c_str() );
		    ROS_INFO("  brake_cmd [%f]\n", msg.brake_cmd );
		}
		int main(int argc, char **argv)
		{
		    ros::init(argc, argv, "listener");
		    ros::NodeHandle n;
		    ros::Subscriber sub = n.subscribe("Brake", 1000, chatterCallback);
		    ros::spin();
		
		    return 0;
		}
		
		```

- 编译前需修改 CMakeLists.txt  文件，在其最后加上如下代码 ，不同的程序包记得修改后6行

	```
	include_directories(include ${catkin_INCLUDE_DIRS})
	
	add_executable(talker src/talker.cpp)
	target_link_libraries(talker ${catkin_LIBRARIES})
	add_dependencies(talker control_node_msgs_generate_messages_cpp)
	
	add_executable(listener src/listener.cpp)
	target_link_libraries(listener ${catkin_LIBRARIES})
	add_dependencies(listener control_node_msgs_generate_messages_cpp)
	```

-  保存 CMakeLists.txt  文件，即可开始编译

	```
	$ cd ~/catkin_ws
	$ catkin_make
	```

- 如果一切顺利，将会得到如下显示

	```
	Base path: /home/muyangren907/catkin_ws
	Source space: /home/muyangren907/catkin_ws/src
	Build space: /home/muyangren907/catkin_ws/build
	Devel space: /home/muyangren907/catkin_ws/devel
	Install space: /home/muyangren907/catkin_ws/install
	####
	#### Running command: "make cmake_check_build_system" in "/home/muyangren907/catkin_ws/build"
	####
	####
	#### Running command: "make -j8 -l8" in "/home/muyangren907/catkin_ws/build"
	####
	[  0%] Built target std_msgs_generate_messages_py
	[  0%] Built target std_msgs_generate_messages_eus
	[  0%] Built target std_msgs_generate_messages_nodejs
	[  0%] Built target std_msgs_generate_messages_lisp
	[  0%] Built target std_msgs_generate_messages_cpp
	[  0%] Built target _control_node_msgs_generate_messages_check_deps_BrakeCmd
	[  9%] Built target control_node_msgs_generate_messages_lisp
	[ 18%] Built target control_node_msgs_generate_messages_nodejs
	[ 27%] Built target control_node_msgs_generate_messages_cpp
	[ 45%] Built target control_node_msgs_generate_messages_py
	[ 63%] Built target control_node_msgs_generate_messages_eus
	Scanning dependencies of target listener
	Scanning dependencies of target talker
	[ 63%] Built target control_node_msgs_generate_messages
	[ 72%] Building CXX object control_node_msgs/CMakeFiles/listener.dir/src/listener.cpp.o
	[ 81%] Building CXX object control_node_msgs/CMakeFiles/talker.dir/src/talker.cpp.o
	[100%] Linking CXX executable /home/muyangren907/catkin_ws/devel/lib/control_node_msgs/talker
	[100%] Linking CXX executable /home/muyangren907/catkin_ws/devel/lib/control_node_msgs/listener
	[100%] Built target talker
	[100%] Built target listener
	```

#### 4.使用roslaunch启动多个节点
- 进入程序包目录

	```
	$ cd ~/catkin_ws/src/control_node_msgs
	```

- 创建并进入 src 目录

	```
	$ mkdir launch
	$ cd launch
	```

- 创建 BrakeCmd.launch 文件

	```
	$ touch BrakeCmd.launch
	```

- 在 BrakeCmd.launch 文件中填入如下内容

	```
	<launch>
	  <node pkg="control_node_msgs" type="talker" name="talker" output = "screen"/>
	  <node pkg="control_node_msgs" type="listener" name="listener" output = "screen"/>
	</launch>
	```

	其中根元素采用 `<launch>` 标签定义，节点采用`<node>`标签定义，启动一个节点需要三个属性：pkg、type和name。其中pkg定义节点所在的功能包名称，type定义节点的可执行文件名称，这两个属性等同于在终端中使用rosrun命令执行节点时的输入参数。name属性用来定义节点运行的名称，将覆盖节点中init()赋予节点的名称。其余可选属性如下
	*  output = "screen"：将节点的标准输出打印到终端屏幕，默认输出为日志文档；
	*  respawn = "true"：复位属性，该节点停止时，会自动重启，默认为false；
	*  required = "true"：必要节点，当该节点终止时，launch文件中的其他节点也被终止；
	*  ns = "namespace"：命名空间，为节点内的相对名称添加命名空间前缀；
	*  args = "arguments"：节点需要的输入参数。

- 最后，一切顺利的话就能运行了

	```
	$ roslaunch control_node_msgs BrakeCmd.launch
	```
 - 结果如下图
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716233052666.gif)
