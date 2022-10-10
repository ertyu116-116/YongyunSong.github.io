---
layout: page
title: turtlebot_3 with SLAM
description: Localization, mapping, and path-planning
img: assets/img/turtlebot/turtlebot_3.jpg
importance: 4
category: work
---

* ## turtlebot_3 with SLAM

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/turtlebot/turtlebot_3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1. Mapping
</div>     


<iframe width="461" height="820" src="https://www.youtube.com/embed/vw9Mafc4hDM" title="turtlebot3" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<div class="caption">
    Path-planning
</div>     


* ## autonomus driving with move_base

```python
#!/usr/bin/env python

import rospy
import actionlib
from smach import State,StateMachine
from move_base_msgs.msg import MoveBaseAction, MoveBaseGoal

waypoints = [
    ['one', (0, 0), (0.0, 0.0,0, 1)],
    ['two', (5, 0), (0.0, 0.0,-1, 1)],
    ['three',(5,-2.5),(0.0, 0.0,-6, 1)],
    ['four', (5, 0), (0.0, 0.0,-6, 1)]
]

#['three', (-0.7217897828952179, 0.3019791814614233), (0.0, 0.0,-0.036968278678352166, 0.9993164395583412)],
#['four', (-0.6991844290024956, 0.40205015910919495), (0.0, 0.0,-0.014366698469828125, 0.9998967936617644)]


class Waypoint(State):
    def __init__(self, position, orientation):
        State.__init__(self, outcomes=['success'])

        # Get an action client
        self.client = actionlib.SimpleActionClient('move_base', MoveBaseAction)
        self.client.wait_for_server()

        # Define the goal
        self.goal = MoveBaseGoal()
        self.goal.target_pose.header.frame_id = 'map'
        self.goal.target_pose.pose.position.x = position[0]
        self.goal.target_pose.pose.position.y = position[1]
        self.goal.target_pose.pose.position.z = 0.0
        self.goal.target_pose.pose.orientation.x = orientation[0]
        self.goal.target_pose.pose.orientation.y = orientation[1]
        self.goal.target_pose.pose.orientation.z = orientation[2]
        self.goal.target_pose.pose.orientation.w = orientation[3]

    def execute(self, userdata):
        self.client.send_goal(self.goal)
        self.client.wait_for_result()
        return 'success'


if __name__ == '__main__':
    rospy.init_node('patrol')

    patrol = StateMachine('success')
    with patrol:
        for i,w in enumerate(waypoints):
            StateMachine.add(w[0],
                             Waypoint(w[1], w[2]),
                             transitions={'success':waypoints[(i + 1) % \
                             len(waypoints)][0]})

    patrol.execute()
    
```

* ## fall detection with turtlebot3 using jetson tx2

<iframe width="457" height="813" src="https://www.youtube.com/embed/Yx25mwMAHZ8" title="fall detection with turtlebot3 using jetson tx2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<div class="caption">
    fall detection
</div>     

* ## pub.cpp (for publish real time images with opencv)

```python
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <iostream>
#include <vector>
#include <ros/ros.h>
#include <opencv2/highgui/highgui.hpp>
#include <sensor_msgs/Image.h>

int main(int argc, char** argv)
{
    ros::init(argc, argv, "opencv_pub");
    
    ros::NodeHandle nh;
    ros::Publisher pub = nh.advertise<sensor_msgs::Image>("camera/image/true", 1);

    cv::VideoCapture cap(0);
    cv::Mat frame;

    
    while(nh.ok())
    {
        cap >> frame;

        if(!frame.empty())
        {

            cv_bridge::CvImage img;

            img.encoding = sensor_msgs::image_encodings::BGR8;
            img.image = frame;
            img.header.frame_id = "camera_link";
            img.header.stamp = ros::Time::now();

            pub.publish(img.toImageMsg());

            
            cv::waitKey(1);
          
        }

        ros::spinOnce();
    }

    return 0;
    
}
```

* ## sub.cpp (subscribe for receiving the real time images with opencv)

```python
#include <ros/ros.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <opencv2/highgui/highgui.hpp>
#include <cv_bridge/cv_bridge.h>
#include <vector>
#include <std_msgs/UInt8MultiArray.h>

void imageCallback(const std_msgs::UInt8MultiArray::ConstPtr& array)
{
  try
  {
    cv::Mat frame = cv::imdecode(array->data, 1);
    cv::imshow("view", frame);
    cv::waitKey(1);
  }
  catch (cv_bridge::Exception& e)
  {
    ROS_ERROR("cannot decode image");
  }
}

int main(int argc, char **argv)
{
  ros::init(argc, argv, "opencv_sub");

  cv::namedWindow("view");
  cv::startWindowThread();

  ros::NodeHandle nh;
  ros::Subscriber sub = nh.subscribe("camera/image", 5, imageCallback);

  ros::spin();
  cv::destroyWindow("view");
}
```