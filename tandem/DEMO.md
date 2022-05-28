# TANDEM - DEMO
# TANDEM-DEMO
This README describes the setup of the TANDEM live demo using a Realsense D455, which has global shutter. TANDEM only uses the mono RGB image and ignores the depth and IMU data. It should be straightforward to adapt the code to another monocular, global shutter RGB camera. Make sure that the camera is connected to a USB3 port, trying different ports can help a lot. Also having a physically good USB3 connection is important.
这个README描述了使用RealsenseD455的TANDEM在线demo的安装，D455是全局快门相机。TANDEM仅使用单目RGB图像，忽略深度图和IMU数据。项目应该可以直接适用于其他单目全局快门RGB相机。请确保相机使用USB3端口，尝试不同的接口位置或许会有帮助，有良好的物理USB3连接也非常重要。

**Please consider the section "General Notes for Good Results" from the main README**
**请参考README中的"General Notes for Good Results"章节

## Build & Run
## 构建及运行

* The Realsense camera requires `librealsense2` for interfacing and we used version `2.49.0-0~realsense0.5306`. We used the method based on `librealsense2-dkms` (see their [github](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md#installing-the-packages)) for getting the kernel patches on the kernel `5.3.0-46-generic` with Ubuntu `18.04.5 LTS`.
* Realsense相机要求`librealsense2`库用作界面，本项目使用`2.49.0-0~realsense0.5306`版本。本文基于`librealsense2-dkms`(查看其[github](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md#installing-the-packages))的方法以获取Ubuntu18.04.5 LTS内核版本`5.3.0-46-generic`的内核补丁。

* Follow the instructions in the README.md to run `cmake` in the build folder an then run `make tandem_demo -j` to build the executable.
* 跟随README.md中的指令在build文件夹中运行`cmake`，然后运行`make tandem_demo -j`以构建可执行文件。

* Calibrate your camera as explained below. **Warning**: The conversion mechanism for the Realsense parameters is a very coarse approximation and performance will be much degraded. This will be used if you omit `calib=X` from the command line. It is only there so that one can quickly check if the interface works.
* 按照如下所述标定你的相机.**警告**：Realsense参数的转换机制是非常粗糙的近似，性能会因此恶化。如果你在命令行中省略`calib=X`，则会使用改机制，这知识为了快速检查接口是否工作。


* Run the demo from the base folder (**for good results please provide your calibration with `calib=X`**)
* 从目录中运行demo(**为了得到良好的结果请使用`calib=X`提供你的标定结果**)
```
build/bin/tandem_demo \
    preset=demo \
    result_folder=results/demo \
    mvsnet_folder=exported/tandem_512x320 \
    mode=1 \
    demo_secs=60
```


## Calibration
## 标定
A good geometric calibration is paramount to achieve good results. We use the excellent calibration software from [basalt](https://gitlab.com/VladyslavUsenko/basalt/-/blob/master/doc/Calibration.md).
一个良好的几何标定对获取好的结果非常重要。我们使用超棒的标定软件[basalt](https://gitlab.com/VladyslavUsenko/basalt/-/blob/master/doc/Calibration.md).

* Build the recorder with `make realsense_calib_recorder -j`
* 使用`make realsense_calib_recorder -j`构建recorder

* Make a directory for results `export TANDEM_CALIB_DIR=path/to/calibration dir && mkdir -p $TANDEM_CALIB_DIR`
* 创建目录以保存结果并`export TANDEM_CALIB_DIR=path/to/calibration dir && mkdir -p $TANDEM_CALIB_DIR`

* Record images: `build/bin/realsense_calib_recorder num_images=600 output_dir=$TANDEM_CALIB_DIR frame_skip=5`. If you have never recorded images for calibration please consider the [basalt documentation](https://gitlab.com/VladyslavUsenko/basalt/-/blob/master/doc/Calibration.md) or [kalibr's wiki](https://github.com/ethz-asl/kalibr/wiki). Images have to be **at least**: well light, taken from many angles, cover all portions of the image, taken under slow motion and thus have minimal rolling shutter. It is often good to place the calibration pattern, **which must be perfectly flat**, on the floor or a wall and move the camera.
* 记录图像: `build/bin/realsense_calib_recorder num_images=600 output_dir=$TANDEM_CALIB_DIR frame_skip=5`。如果你从未为标定记录过图像，请参考[basalt documentation](https://gitlab.com/VladyslavUsenko/basalt/-/blob/master/doc/Calibration.md)或者[kalibr's wiki](https://github.com/ethz-asl/kalibr/wiki)。图像必须要：1.良好光照，2.多角度拍摄，3.覆盖图像的各部分，4.缓慢移动，5.具有最小的卷帘快门。通常将标定板(需要绝对平整)置于地板或者墙上然后移动相机。


* Setup a python environment with ROS packages to use the `scripts/calib_convert_to_rosbag.py` script. We used e.g. `pip install --extra-index-url https://rospypi.github.io/simple/ sensor_msgs` to install pure python version of ROS packages without setting up a full ROS environment, see their [github](https://github.com/rospypi/simple). [RoboStack](https://github.com/RoboStack) seems to be another alternative that we haven't tried.
* 配置具有ROS包的python环境以使用`scripts/calib_convert_to_rosbag.py`脚本。我们使用`pip install --extra-index-url https://rospypi.github.io/simple/ sensor_msgs`以安装ROS包的纯python版本并避免安装全ROS环境。查看rospy的[github](https://github.com/rospypi/simple).[RoboStack](https://github.com/RoboStack) 似乎也可以使用，但我们没有尝试.


* Convert the images to a rosbag used by basalt: `python scripts/calib_convert_to_rosbag.py $TANDEM_CALIB_DIR`
* 使用basalt将图像转换为rosbag`python scripts/calib_convert_to_rosbag.py $TANDEM_CALIB_DIR`

* Install `basalt_calibrate`: we used the `apt` [installation](https://gitlab.com/VladyslavUsenko/basalt#apt-installation-for-ubuntu-2004-and-1804-fast)
* 安装`basalt_calibrate`:我们使用`apt`安装[installation](https://gitlab.com/VladyslavUsenko/basalt#apt-installation-for-ubuntu-2004-and-1804-fast)


* Run the calibration: `basalt_calibrate --dataset-path $TANDEM_CALIB_DIR/calib.bag --dataset-type bag --aprilgrid /usr/etc/basalt/aprilgrid_6x6.json --result-path $TANDEM_CALIB_DIR --cam-types kb4` and follow the instructions [in the docs](https://gitlab.com/VladyslavUsenko/basalt/-/blob/master/doc/Calibration.md). We are done after the intrinsics have been optimized and saved (`save_calib` button).
* 运行标定:`basalt_calibrate --dataset-path $TANDEM_CALIB_DIR/calib.bag --dataset-type bag --aprilgrid /usr/etc/basalt/aprilgrid_6x6.json --result-path $TANDEM_CALIB_DIR --cam-types kb4`并跟随指令文档执行[in the docs](https://gitlab.com/VladyslavUsenko/basalt/-/blob/master/doc/Calibration.md)。当内参被优化和保存后，标定就结束(`save_calib`按钮)


* Convert to the `camera.txt` format needed by TANDEM: `python scripts/calib_convert_to_txt.py $TANDEM_CALIB_DIR`. This script also considers the down-scaling from `1280x800` to `512x300`. The demo is run in resolution `512x300` for better performance. Doing the calibration in resolution `1280x800` gives more accurate results.
