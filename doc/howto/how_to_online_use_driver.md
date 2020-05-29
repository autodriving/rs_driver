# How to get point cloud from lidar (online)



### Introduction

​	This document will show you how to use the interface to get point cloud from lidar when online connecting a lidar.

### Steps

1， define a point type

2， define a point cloud callback funtion

3， define a exception callback function

4， instanciate the driver object

5， define the parameter, config the parameter

6， call the driver initialize fuction

7， register the point cloud callback and exception callback

8， start the driver



Please follow the above steps to do advanced development, details are as follow:



#### Step1

Define a point type. Now the driver only support **x,y,z,intensity**, so users must make sure their self-defined point type contains variable **x,y,z,intensity**. Otherwise compile error may occur. (or user can use pcl::PointXYZI directly)

```c++
struct PointXYZI ///< user defined point type
{
    double x;
    double y;
    double z;
    double intensity;
};
```

#### Step2

Define the point cloud callback function. The PointXYZI in template is the point type we defined in step1. When point cloud message is ready, this function will be called by driver. **Note! Please dont add and time-consuming operations in this function!** User can make a copy of the message and process it in another thread.  Or user can add some quick operations such like ros publish in the callback function.

```c++
void pointCloudCallback(const PointcloudMsg<PointXYZI> &msg)
{
    std::cout << "msg: " << msg.seq << " pointcloud size: " << msg.pointcloud_ptr->size() << std::endl;
}
```

#### Step3

Define the exception callback function. When driver want to send out infos or error codes, this function will be called. Same as the previous callback function, please **do not add any time-consuming operations in this callback function!**

```c++
void exceptionCallback(const Error &code)
{
    /* Note: Please do not put time-consuming operations in the callback function! */
    /* Make a copy of the error message and process it in another thread is recommended*/
    std::cout << "Error code : " << code.toString() << std::endl;
}
```

#### Step4

Instanciate the driver object

```c++
 LidarDriverInterface<PointXYZI> driver;          ///< Declare the driver object
```



#### Step5

Define the parameter and config it. The msop port and difop port number of lidar can be got from **wireshark(a network socket capture software)**. The default value is 6699 and 7788. User also need to make sure the **lidar type** is set correctly.

```c++
 RSLiDAR_Driver_Param param;                      ///< Creat a parameter object
 param.input_param.msop_port = 6688;              ///< Set the lidar msop port number the default 6699
 param.input_param.difop_port = 7799;             ///< Set the lidar difop port number the default 7788
 param.lidar_type = LiDAR_TYPE::RS16;             ///< Set the lidar type. Make sure this type is correct!
```



#### Step6

Call the initialize function and pass the parameter into the driver.

```c++
 driver.init(param);                              ///< Call the init funtion and pass the parameter
```



#### Step7

Register the callback functions we defined in step2 and step3.

```c++
 driver.regPointRecvCallback(pointCloudCallback); ///< Register the point cloud callback funtion into the driver
 driver.regExceptionCallback(exceptionCallback);  ///<Register the exception callback funtion into the driver
```



#### Step8

Call the start function to start the driver

```c++
 driver.start();                                  ///< Call the start funtion. The driver thread will start.
```



### *Congratulations! You have finished the first simple demo of Robosense lidar driver! You can find the whole demo code in the demo folder under the project. Feel free to connect with us if you have and question about the driver.*