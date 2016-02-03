# Run s-gupta's rgbd repository

This is a note on how to run the code of "Perceptual Organization and Recognition of Indoor Scenes from RGB-D Images" (CVPR 2013) by S. Gupta. The code works on Ubuntu evironment.

Check out the **Problems Found** section below if you encounter any problems while following the instructions.

Check out the **Remarks** section below for (naive) evaluation of the system's performance.

## Instructions
1. **rbgd** repository
 * Clone the **rgbd** repository.
 ```sh
 $ git clone https://github.com/s-gupta/rgbd.git
 $ cd rgbd
 ```
 * Switch to branch **dev**.
 ```sh
 $ git checkout dev
 ```
2. Pretrained models
 * Download the **Pretrained models**: http://www.cs.berkeley.edu/~sgupta/cvpr13/model.tgz.
 * Extract the compressed models to folder **rgbd**.
 ```sh
 $ tar -xvf /path/to/model.tar
 ```
3. Dependencies

  All dependencies are stored in folder **external**.
    
  These libraries are:

    a. gPb-UCM from http://www.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/BSR/BSR_full.tgz.
   
    b. VLFeat from http://www.vlfeat.org/.
    * The VLFEAT that already exists in the **rgbd** repository does not seem to be working. You should delete the folder vlfeat. Then, you download the library using the link above, extract it, rename it to vlfeat and copy it to folder external.
    * Copy folder vlfeat/toolbox/mex/mexa64 to vlfeat/.
   
   c. liblinear from http://www.csie.ntu.edu.tw/~cjlin/liblinear/.
    * The liblinear that already exists in the **rgbd** repository does not seem to be working. You should delete the folder liblinear-1.94. Then, you download the library using the link above, extract it, rename it to liblinear-1.94 and copy it to folder external.
    * In the Makefile of external/liblinear-1.94/matlab, you need to modify the MATLABDIR to points to where you install MATLAB, for example:
    ```sh
    MATLABDIR ?= /usr/local/MATLAB/R2015a
    ```
   
   d. liblinear dense from http://ttic.uchicago.edu/~smaji/projects/digits/.
    * The liblinear dense that already exists in the **rgbd** repository does not seem to be working. You should delete the folder liblinear-1.5.-dense. Then, you download the library using the link above, extract it, rename it to liblinear-1.5-dense and copy it to folder external.
    * In the Makefile of external/liblinear-1.5-dense/matlab, you need to modify the MATLABDIR to points to where you install MATLAB, for example:
    ```sh
    MATLABDIR ?= /usr/local/MATLAB/R2015a
    ```
   
    e. SIFT color desprictors from http://www.colordescriptors.com, v2.1.
   
   f. Image Stack Library https://code.google.com/p/imagestack/.
    * The Image Stack Lib that already exists in the **rgbd** repository does not seem to be working. You should delete the folder imagestack-src. Then, you download the library using the link above, extract it, rename it to imagestack-src and copy it to folder external.
    * Create the following folders:
    ```sh
    $ mkdir external/imagestack-src/bin
    $ mkdir external/imagestack-src/bin/build
    ```
    
  In order to build these libraries, you must add matlab entry to the launcher.
  ```sh
  $ sudo apt-get install matlab-support
  ```
  The script external/build_external.sh help you build all these dependencies. However, you need to change line 3, 10, 23 in this script from "make" to "make -f Makefile". After that, run the script as follows:
  ```sh
  $ cd external
  $ sh build_external.sh
  ```
  
  After building all the dependencies:
  * Set executable permission for colorDescriptor.
  ```sh
  $ chmod +x colorDescriptor
  ```
  * Set executable permission for ImageStack.
  ```sh
  $ chmod +x ImageStack
  $ cd ..
  ```
   
4. Run the matlab code
 * Open matlab, navigate to folder **rgbd**, run **startup.m**.
 * Setup the directories for storing results:
 ```sh
 $ mkdir cachedir/release/cache
 $ mkdir cachedir/release/cache/sceneOuts
 $ mkdir cachedir/release/cache/ucmFeatures
 $ mkdir cachedir/release/cache/ucmFeatures/features
 $ mkdir cachedir/release/cache/visOut
 $ mkdir cachedir/release/cache/visOut/cc
 $ mkdir cachedir/release/cache/visOut/ss
 $ mkdir cachedir/release/cache/visOut/ucm
 $ mkdir cachedir/release/output/ucm
 $ mkdir cachedir/release/output/segmentation
 $ mkdir cachedir/release/output/amodal
 ```
 * You can run the system on a new pair of RGB-D image by using the **runAll.m** function.
   * Provide the following parameters for **runAll.m**:
     * imNum: id for the output, it could be any integer starting from 1.
      * rgbImage: the RGB image.
       * depthImage: the depth image.
        * cameraMatrix: the parameters of the Kinect camera. It is used to project the depth image into the point cloud. You can find the Kinect parameters in the toolbox of the NYU v2 Dataset (http://cs.nyu.edu/~silberman/code/toolbox_nyu_depth_v2.zip).
    * Example:
    ```sh
    matlab > rgbImage = imread(rgbName)
    matlab > depthImage = imread(depthName)
    matlab > cameraMatrix = load(cameraName) 
    matlab > runAll(1, rgbImage, depthImage, cameraMatrix)
    ```
 * We also provide a sample pair of RGB-D image for testing the system
   * Download two files **img_5001.png** and **img_5001.mat** in this repository and put it in folder **rgbd**.
    * Since **img_5001.mat** contains the precomputed point cloud, you need to modify the code in **runAll.m** from:
    ```sh
    33. [x3 y3 z3] = getPointCloudFromZ(double(depthImage*100), cameraMatrix, 1);
    34. save(fullfile(paths.pcDir, [imName '.mat']), 'x3', 'y3', 'z3');
    ```
     * into:
     ```sh
     33. %[x3 y3 z3] = getPointCloudFromZ(double(depthImage*100), cameraMatrix, 1);
     34. x3 = depthImage.x3;
     35. y3 = depthImage.y3;
     36. z3 = depthImage.z3;
     37. save(fullfile(paths.pcDir, [imName '.mat']), 'x3', 'y3', 'z3');
     ```
      * Example:
      ```sh
      matlab > rgbImage = imread('img_5001.png')
      matlab > depthImage = load('img_5001.mat')
      matlab > runAll(1, rgbImage, depthImage, [])
      ```

## Problems Found
1. Performance
 * It takes a fair amount of time to run a pair of RGB-D image. You can speed up the performace by replacing **parfor** statement with **for** statement in the matlab code.
2. Missing libraries when building the dependencies in folder **external** 
 * Fix by:
 ```sh
 $ sudo apt-get install libname
 ```
3. Cannot set executable permission for **colorDescriptor** and **ImageStack**
 * If you store your repository in a NTFS/FAT hard drive, you cannot set executable permission.
 * Fix by: moving colorDescriptor and ImageStack to Ubuntu hard drive
 ```sh
 $ mv colorDescriptor /home/path/to/dir
 $ mv ImageStack /home/path/to/dir 
 ```
 * then set the permission as instructed above
 ```sh
 $ chmod +x /home/path/to/dir/colorDescriptor
 $ chmod +x /home/path/to/dir/ImageStack
 ```
 * modify the paths to these dependencies in COM/getPaths.m from:
 ```sh
 46. pathstr = fileparts(mfilename('fullpath'));
 47. paths.siftLib = fullfile(pathstr, '..', 'external', 'colorDescriptor');
 48. paths.imageStackLib = fullfile(pathstr, '..', 'external', 'ImageStack');
 ```
 * into:
 ```sh
 46. pathstr = fileparts(mfilename('fullpath'));
 47. paths.siftLib = fullfile('/home/path/to/dir', 'colorDescriptor');
 48. paths.imageStackLib = fullfile('/home/path/to/dir', 'ImageStack');
 ```
 * Make sure that the ImageStack library is executable. To ensure that your computer will not crash/hang when running the system, put a break points at line 19 in the file segmentation/jointBilateral.m. Stop running immediately if the system cannot call ImageStack library. This is because jointBilateral.m contains a recursive call that will be executed if ImageStack doesn't work.

##Remarks
* The system runs smoothly on a 4GB RAM, i7 GPU.
* It takes quite a long time to perform recognition on a pair of RGB-D image (about 3-5 mins).
* Computing UCM features takes the longest time. Fortunately, the system stores UCM results. So you just have to run it once.
* Computing colorDescriptor also takes a long time (due to external library).
* Recognition result on the sample pair of RGB-D image (img_5001.png and img_5001.mat) is good, but there are still errors.
* Sample result:

![alt tag](https://github.com/tbhuu/gupta-rgbd/blob/master/result.png)
