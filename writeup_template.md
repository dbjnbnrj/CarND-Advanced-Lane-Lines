## Writeup

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

All code for this project is in file [pipeline.ipynb](https://github.com/dbjnbnrj/CarND-Advanced-Lane-Lines/blob/master/pipeline.ipynb)

The video is available on [project_video_output.mp4] (https://github.com/dbjnbnrj/CarND-Advanced-Lane-Lines/blob/master/project_video_output.mp4)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

#### Ans:

* I used the chessboard images present in camera_cal to obtain an object to image point mapping.

* Object Points correspond to points in the real world and are defined using a (x,y,z) coordinate system. We assumed the chessboard size to 9x6 for the project so we had to create points using a 3D matrix with for 9 rows and 6 columns. For object points we assume that z is 0.


* As we are mapping object points to image points we need to obtain image points from our chessboard images using the `cv2.findChessboardCorners` function.

![Chessboard Corners](./output_images/chessboard_corners.png "Find Chessboard Corners")

* Finally, I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![Distortion Correction](./output_images/original_to_undistorted_chessboard.png "Original To Undistorted")


Code Link: The code is in pipeline.ipynb section **Calibration Matrix**

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.


Ans:
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:


![Distortion Correction](./output_images/original_to_undistort_car.png "Original To Undistorted Car")


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.


####  Ans:
I experimented with a number of color space and gradient transform techniques to create the threshold binary image. This was one of the most challenging parts of the videos.



####  Playing with Gradients

I used Sobel operators of different kinds with my image and obtained the following results


1.Absolute Sobel operator in X-direction

![Absolute Sobel operator in X-direction](./output_images/gradient_abs_sobel_x.png "Absolute Sobel operator in X-direction")


2.Absolute Sobel operator in Y-direction

![Absolute Sobel operator in Y-direction](./output_images/gradient_abs_sobel_y.png "Absolute Sobel operator in Y-direction")


3.Sobel with direction threshold

![Sobel with direction threshold](./output_images/gradient_to_dir_sobel.png "Sobel with direction threshold")

4.Sobel with magnitude threshold

![Sobel with magnitude threshold](./output_images/gradient_to_mag_sobel.png "Sobel with magnitude threshold")

5.Combined Result of Absolute Threshold in X and Direction/Magnitude Threshold

![Combined Sobel](./output_images/gradient_combined_sobel.png "Combined Sobel")

####  Playing with Colorspace


1. RGB Colorspace

**Red Channel**

![Red Colorspace with threshold](./output_images/colorspace_rgb_r.png "R_transform _to_R_transform_w_threshold")

**Blue Channel**

![Blue Colorspace with threshold](./output_images/colorspace_rgb_b.png "B_transform _to_B_transform_w_threshold.png")


**Green Channel**

![Green Colorspace with threshold](./output_images/colorspace_rgb_g.png "G_transform _to_G_transform_w_threshold.png")


> From this we observe that the Red channel is very good at detecting the yellow lane line


2. HLS Colorspace

a. H Channel

![H Colorspace with threshold](./output_images/colorspace_hls_h.png "H_transform")

b. L Channel

![L Colorspace with threshold](./output_images/colorspace_hls_l.png "L_transform")


c. S Channel

![S Colorspace with threshold](./output_images/colorspace_hls_s.png "S transform")

> The S Channel can detect lane lines fairly accurately


3. LAB Colorspace

a. L Channel

![L Colorspace with threshold](./output_images/colorspace_lab_l.png "L_transform")

b. A Channel

![A Colorspace with threshold](./output_images/colorspace_lab_a.png "A_transform")


c. B Channel

![B Colorspace with threshold](./output_images/colorspace_lab_b.png "B transform")

> As we obtained fairly accurate results by isolating the R channel from RGB & S from HLS we use these two color channels to obtain the binary image in our pipeline


#### Final Binary Image

> I obtained the final binary image by applying perspective transform followed by combining the images from the R colorspace of RGB and the S colorspace of HLS.

![original_to_pipeline_transform_test1.png](./output_images/original_to_pipeline_transform_test1.png")

![original_to_pipeline_transform_test2.png](./output_images/original_to_pipeline_transform_test2.png")

![original_to_pipeline_transform_test3.png](./output_images/original_to_pipeline_transform_test3.png")

![original_to_pipeline_transform_test4.png](./output_images/original_to_pipeline_transform_test4.png")


![original_to_pipeline_transform_test5.png](./output_images/original_to_pipeline_transform_test5.png")



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.


####  Ans:
Code for my perspective transform is available in the Perspective Transform section of the ipynb notebook.


I used the function `get_perspective_transform_points` to get the source and destination points for my image.

This resulted in the following source and destination points:
| Source        | Destination   |
|:-------------:|:-------------:|
| 600,450     | 400,0        |
| 750,454      | 900,0      |
| 250,680     | 400, 720      |
| 1100,680     | 960, 720       |

I verified that my perspective transform was working as expected:

![get_perspective_transform_points.png](./output_images/get_perspective_transform_points.png "Get_perspective_transform_points.png")


After perspective transform (which is done by function `perspective_transform`) the result is as follows -

![original_to_perspective_transform.png](./output_images/original_to_perspective_transform.png "original_to_perspective_transform.png")




#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifying lane line fit is available in the [Calculating Lane Lines](https://github.com/dbjnbnrj/CarND-Advanced-Lane-Lines/blob/master/pipeline.ipynb#Calculating Lane Lines) section of the ipython notebook in the functions `find_lane_lines_with_window` and `find_lane_lines_without_window`.

The result of the two functions is-


1. Find lane lines with window
![find_lane_lines_with_window.png](./output_images/find_lane_lines_with_window.png "find_lane_lines_with_window.png")

2. Find lane lines without window
![find_lane_lines_without_window.png](./output_images/find_lane_lines_without_window.png "find_lane_lines_without_window.png")


For both these functions I used the example code provided in the course to observe the lane lines and plot windows and polynomial lines.


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for calculating Radius of Curvature is in a section of my ipython notebook marked [Radius of Curvature](http://localhost:8888/notebooks/pipeline.ipynb#Radius-of-Curvature). The name of the function that calculates the value is `radius_of_curvature_calculation`. The result of radius of curvature calculation is as follows -

![radius_of_curvature.png](./output_images/radius_of_curvature.png "radius_of_curvature.png")


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.


The `draw_lane` function plots the lane back to the original image.

![radius_of_curvature.png](./output_images/radius_of_curvature.png "radius_of_curvature.png")

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Core issues I faced involved the following

- Tree shadows:
Getting an accurate binary image using the given colorspace and gradients proved difficult, particularly for road images where shadows were present.

- Handling changing lane curves:
Calculating steep curves is difficult as the radius of curvature is less accurate. Often the video pipeline has to revert back to the previous image and then correct itself for the curved data.
