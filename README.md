# Gstreamer_KinectV2
 Gstreamer the KinectV2
 
 These are the steps I took to get the KinectV2 to run on the Jetson Nano using gstreamer.
Sorry about it being so long winded and convaluted but it was a quite a chore just to get the instructions down to this list.

At the bottom of this page are Gstreamer commands to access the Kinect V2 streams.
I also added the gestreamer commands to run the Kinect V2 with nvidia Deepstream 4.

If you dont want run the gstreamer commands with the "--gst-plugin-path=/home/nano/gst-plugins-vr/build" you can copy the contents of:/home/dlinano/gst-plugins-vr/build TO
/usr/lib/aarch64-linux-gnu/gstreamer-1.0/


Most of this stuff is needed just to compile the gstreamer plugins for the KinectV2.
I ran these commands after installing the latest nano SD card image :r32.3.1.
Here is the link to the git hub repo that I got the plugins from:

 https://github.com/lubosz/gst-plugins-vr.git



Lets get started

1.$sudo apt install python3-pip

2.$sudo pip3 install meson

3.$sudo apt-get install ninja-build

4.$sudo apt-get install libudev-dev

5.$git clone git://github.com/libusb/hidapi.git

6.$cd hidapi

7.$PKG_CONFIG_DIR=
  
8.$PKG_CONFIG_LIBDIR=$STAGING/lib/pkgconfig:$STAGING/share/pkgconfig

9.$PKG_CONFIG_SYSROOT_DIR=$STAGING 

10.$./bootstrap

11.$./configure

12.$make

13.$sudo make install

14.$cd

15.$git clone https://github.com/assimp/assimp.git

16.$cd assimp

17.$cmake CmakeLists.txt

18.$make -j4  # be patient with this one its  really works the nano

19.$sudo make install

20.$sudo apt-get install -y assimp-utils

21.$cd

22.$git clone https://github.com/OpenHMD/OpenHMD.git

23.$cd OpenHMD

24.$meson ./build

25.$ninja -C ./build

26.$cd

27.Clone this repo and move “libopenhmd-dev_0.2.0-3_arm64.deb”
to the Nano home directory

$sudo apt install ./libopenhmd-dev_0.2.0-3_arm64.deb 


28.The following 2 commands will give you file permission to copy opencv2 from the opencv4 file.

   $sudo chmod -R a+rwx /usr/local/include

   $sudo chmod -R a+rwx /usr/include

  Copy the opencv2 folder thats in the opencv4 folder
  and place it in the same folder as the opencv4 folder. 

29. $cd

30. $git clone https://github.com/OpenKinect/libfreenect2.git

31. $cd libfreenect2

32. $sudo apt-get install build-essential cmake pkg-config

33. $sudo apt-get install libusb-1.0-0-dev

34. $sudo apt-get install libturbojpeg0-dev

35. $sudo apt-get install libglfw3-dev

36. $sudo apt-get install libopenni2-dev

37. $mkdir build && cd build 

38. $cmake .. 

39. $make

40. $sudo make install

41. $sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/

42. $sudo .bin/Protonect # you should be able to test the kinect with this

43. $cd


44. $pip3 install graphene

45. $sudo apt install libgraphene-1.0-dev


46.  $git clone https://github.com/lubosz/gst-plugins-vr.git


47. $cd gst-plugins-vr

48. $sudo ./configure

49. Copy file "gst3drenderer.c" from this repo and use it to replace the one thats in the folder home/yourpath/gst-plugins-vr/gst-libs/gst/3d/

50. $sudo make

51. $cd


If all went well you should be able to run these gstreamer commands.
You will notice that gst-plugin-path needs to point to the :/home/yourpath/gst-plugins-vr/build folder.

Make sure you KinectV2 is plugged in

1.Live steam

$sudo  gst-launch-1.0 --gst-plugin-path=/home/nano/gst-plugins-vr/build freenect2src sourcetype=1 ! videoscale ! video/x-raw,width=1280,height=720,framerate=30/1 ! glimagesink

2.infared stream

$sudo gst-launch-1.0 --gst-plugin-path=/home/nano/gst-plugins-vr/build freenect2src sourcetype=2 ! glimagesink

3. depth stream

$sudo gst-launch-1.0 --gst-plugin-path=/home/nano/gst-plugins-vr/build freenect2src sourcetype=0 ! glimagesink

4. Nvidia Deepstream 4 with live stream

 gst-launch-1.0 freenect2src sourcetype=1 ! videoconvert ! 'video/x-raw,format=(string)YUY2' !  nvvidconv ! 'video/x-raw(memory:NVMM),format=(string)NV12' ! nvvidconv ! 'video/x-raw,format=(string)NV12' ! nvvideoconvert ! 'video/x-raw(memory:NVMM),format=(string)NV12' ! mux.sink_0 nvstreammux live-source=1 name=mux batch-size=1 width=1280 height=720 ! nvinfer config-file-path=/opt/nvidia/deepstream/deepstream-4.0/samples/configs/deepstream-app/config_infer_primary_nano.txt  batch-size=1 ! nvmultistreamtiler rows=1 columns=1 width=1280 height=720 ! nvvideoconvert ! nvdsosd ! nvegltransform ! nveglglessink
 
 5. Nvidia Deepstream 4 with infared stream
 
 gst-launch-1.0 freenect2src sourcetype=2 ! videoconvert ! 'video/x-raw,format=(string)YUY2' !  nvvidconv ! 'video/x-raw(memory:NVMM),format=(string)NV12' ! nvvidconv ! 'video/x-raw,format=(string)NV12' ! nvvideoconvert ! 'video/x-raw(memory:NVMM),format=(string)NV12' ! mux.sink_0 nvstreammux live-source=1 name=mux batch-size=1 width=512 height=424 ! nvinfer config-file-path=/opt/nvidia/deepstream/deepstream-4.0/samples/configs/deepstream-app/config_infer_primary_nano.txt  batch-size=1 ! nvmultistreamtiler rows=1 columns=1 width=512 height=424 ! nvvideoconvert ! nvdsosd ! nvegltransform ! nveglglessink
 
 6,Nvidia Deepstream 4 with depth stream
 
 gst-launch-1.0 freenect2src sourcetype=0 ! videoconvert ! 'video/x-raw,format=(string)YUY2' !  nvvidconv ! 'video/x-raw(memory:NVMM),format=(string)NV12' ! nvvidconv ! 'video/x-raw,format=(string)NV12' ! nvvideoconvert ! 'video/x-raw(memory:NVMM),format=(string)NV12' ! mux.sink_0 nvstreammux live-source=1 name=mux batch-size=1 width=512 height=424 ! nvinfer config-file-path=/opt/nvidia/deepstream/deepstream-4.0/samples/configs/deepstream-app/config_infer_primary_nano.txt  batch-size=1 ! nvmultistreamtiler rows=1 columns=1 width=512 height=424 ! nvvideoconvert ! nvdsosd ! nvegltransform ! nveglglessink
 
 

