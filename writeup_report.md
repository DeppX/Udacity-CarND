

```python
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

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code file named as [camera_calibration.py](./camera_calibration.py) or [camera_calibration.ipynb](./camera_calibration.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

One example plot is showing here as the result of original chessboard image with corners highlighted on it:

<img src="./output_images/Chessboard/img1_chessboard_corners.jpg" width="500">

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

The original chessboard image:
    
<img src="./output_images/Chessboard/img2_original_chessboard.jpg" width="500">

The distortion-corrected chessboard image:
    
<img src="./output_images/Chessboard/img3_corrected_chessboard.jpg" width="500">

Then I saved calibration results of "mtx, dist, rvecs, and tvecs" in the dictionary "calibration_result.p" which can be found in the submitted folder. 

For the preprocessing in the later steps, these calibration parameters can be retrieved quickly from the dictionary without redoing the calibration every time a new image is showing up.




### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, a separated snippet code named [correct_test_images.py](./correct_test_images.py) or [correct_test_images.ipynb](./correct_test_images.ipynb) is created in the same folder.
In it, all files in "./test_images" folder are loaded and processed with cv2.undistort, by applying the mtx, dist parameters calculated in "camera_calibration.py".

For a better comparison, I've also downloaded an example image from one quiz in the lectures, as shown here:

<img src="./test_images/img_quiz.png" width="500">

It has rectangler-shaped highway road signs which are more obvious when undistorted, as shown below:

<img src="./output_images/Undistorted/img_quiz_undistorted.jpg" width="500">

All other test images are also processed with cv2.undistort and saved in the "./output_images/Undistorted/" folder, named with their original name, together with a suffix "_undistorted".

One example is shown here as original test1.jpg:
    
<img src="./test_images/test1.jpg" width="500">


The corrected test1_undistorted.jpg:
    
<img src="./output_images/Undistorted/test1_undistorted.jpg" width="500"> 



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

As suggested from Q&A session, I used a combination of color and gradient thresholds to generate a binary image. 
Functions of thresholds filtering are summarized in [color_gradient_thresh.py](./color_gradient_thresh.py) or [color_gradient_thresh.ipynb](./color_gradient_thresh.ipynb).

For color thresholds setup, both HLS and HSV coordinates are selected. Saturation threshold is set as (100,255). Value threshold is set as (50,255).
Images with color thresholds filtered only are stored in the folder "./output_images/Thresholded/Color_Thresh/".

For gradient thresholds setup, both x and y directions are concerned. Gradx threshold is set as (12,255). Grady threshold is set as (25,255).
Images with gradient thresholds filtered only are stored in the folder "./output_images/Thresholded/Gradient_Thresh/".

An "OR" operator is applied to color and gradient thresholds. Images with these combined filter are stored in the folder "./output_images/Thresholded/Combined_Thresh/".


An example of unfiltered undistorted quiz image is shown here:
    
<img src="./output_images/Undistorted/img_quiz_undistorted.jpg" width="500">


Color filtered quiz image is shown here:
    
<img src="./output_images/Thresholded/Color_Thresh/img_quiz_color.jpg" width="500">

Gradient filtered quiz image is shown here:

<img src="./output_images/Thresholded/Gradient_Thresh/img_quiz_gradient.jpg" width="500">

A comprehensively filtered quiz image is shown here:
    
<img src="./output_images/Thresholded/Combined_Thresh/img_quiz_thresholded.jpg" width="500">

Example of test1.jpg is also demonstrated here:
    
Undistorted test1.jpg without filtering:
    
<img src="./test_images/test1.jpg" width="500">

Color filtered test1.jpg is shown here:
    
<img src="./output_images/Thresholded/Color_Thresh/test1_color.jpg" width="500">

Gradient filtered test1.jpg is shown here:

<img src="./output_images/Thresholded/Gradient_Thresh/test1_gradient.jpg" width="500">

A comprehensively filtered test1.jpg is shown here:
    
<img src="./output_images/Thresholded/Combined_Thresh/test1_thresholded.jpg" width="500">

From examples above, it can be found that the color filtering helps removes noises edges around lane lines and highlights the core part of lane lines, although it might lose some lane line information, especially on dashed lines. Meanwhile it also helps eliminate influence of different colors on the detection.
On the other side, the gradient method helps reserve lane line information at distance from the image.
The combination of the two helps highlight the lane line and preserve as much of its outline information as possible.


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

A separate file of [perspective_transform.py](./perspective_transform.py) or [perspective_transform.ipynb](./perspective_transform.ipynb) is created here to perform a bird-view transformation of test_images.
As recommended from Q&A session, parameters for src/dst points selection are defined as below:
```
    bot_width = 0.76
    mid_width = 0.08
    height_pct = 0.62
    bottom_trim = 0.935
    
    src = np.float32([[img.shape[1]*(0.5-mid_width/2), img.shape[0]*height_pct],
                      [img.shape[1]*(0.5+mid_width/2), img.shape[0]*height_pct],
                      [img.shape[1]*(0.5+bot_width/2), img.shape[0]*bottom_trim],
                      [img.shape[1]*(0.5-bot_width/2), img.shape[0]*bottom_trim]])
    
    offset = img_size[0]*0.25
    
    dst = np.float32([[offset,0], 
                      [img_size[0] - offset, 0],
                      [img_size[0] - offset, img_size[1]],
                      [offset, img_size[1]]])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 588.8, 446.4  | 320, 0        | 
| 691.2, 446.4  | 960, 0        |
| 1126.4, 673.2 | 960, 720      |
| 153.6, 673.2  | 320, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
Filtered images from Step 3 are re-plotted with src points (red color). Bird-view images transformed from those filtered images are also plotted with blu dst points on them. 
Images can be found in the folder "./output_images/Warped/".

Example quiz image with src points are here:

<img src="./output_images/Warped/img_quiz_thresholded_dot.jpg" width="500">

Perspective transformed quiz image with dst points are here:

<img src="./output_images/Warped/img_quiz_thresholded_warp.jpg" width="500">

Example test1 image with src points are here:

<img src="./output_images/Warped/test1_thresholded_dot.jpg" width="500">

Perspective transformed test1 image with dst points are here:

<img src="./output_images/Warped/test1_thresholded_warp.jpg" width="500">




#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To idnetify lane-line pixels and fit their positions, [lane_line_fitting.py](./lane_line_fitting.py) or [lane_line_fitting.ipynb](./lane_line_fitting.ipynb) is created.
The perspective transformed image is divided into several horizontal strips (steps) of equal height. 
Each strip is then divided by half to identify left and right lane lines.
For each half, np.sum is used to count all the non-zero pixels within the step height, along the x-axis, like a histogram.
np.convolve is used to get an accurate result. Each convolve calculation only include a small window of the half strip.
In it, the sum array of the pixels and a ones array are convoluted. The x-value at which maximum np.convolve is achieved is recorded as the most possible position for lane line in that half stripe (either left or right).
This x-value is treated as the centroid of the lane line in that strip.
The recent 15 strips are averaged to smoothen the detection.
These are wrapped up in a class definition named tracker() as in the first cell of [lane_line_fitting.ipynb](./lane_line_fitting.ipynb). 
To find the lane line position reliably, the first left and right strip centroid is selected utilizing the convolution of bottom quarters of the image height (instead of only the height of a window) and whole half image width.
For all the other strips above the first one, only a window height is used, and only around (with a margin) the previous position of lane line in the previous strip.

Fit positions of lane-line pixels with a polynomial np.polyfit.

All test images after being warped have been loaded to [lane_line_fitting.ipynb](./lane_line_fitting.ipynb) for lane line finding and curvefitting.
Processed images with lane line marked as green boxes can be found in the folder "./output_images/Fitting/".


Example quiz image with small windows marked lane lines is here:

<img src="./output_images/Fitting/img_quiz_thresholded_bv_windows.jpg" width="500">

Example quiz image with curvefit lane lines is here:

<img src="./output_images/Fitting/img_quiz_thresholded_bv_fit.jpg" width="500">

Inner lane is highlighted as here:
    
<img src="./output_images/Fitting/img_quiz_thresholded_bv_highlight.jpg" width="500">


Example test1 image with small windows marked lane lines is here:

<img src="./output_images/Fitting/test1_thresholded_bv_windows.jpg" width="500">

Example test1 image with curvefit lane lines is here:

<img src="./output_images/Fitting/test1_thresholded_bv_fit.jpg" width="500">

Inner lane is highlighted as here:
    
<img src="./output_images/Fitting/test1_thresholded_bv_highlight.jpg" width="500">



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature and vehicle position are calculated in Cell 4 of [lane_line_fitting.py](./lane_line_fitting.py) or [lane_line_fitting.ipynb](./lane_line_fitting.ipynb).

Radius of curvature is calculated with the y-value at the bottom of the image, following with the equation: 
    
    R = (1 + (2 x A x y + B)^2)^(3/2) / abs(2xA)
    curvefitting =  A x y^2 + B x y + C
    
Meter per pixel on y axis is 10/720, and on x axis is 4/384.

To calculate the vehicle position referring to the center of the road, the mid point of the two lane lines detected and the mid point of the camera image are compared.
The difference is rescaled with the meter per pixel on x axis.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I created a [image_process.py](./image_process.py) or [image_process.ipynb](./image_process.ipynb) to put the above steps all together to process all the test images.

Based on Step 4, I already have curvefitted lane line and inner lane highlighted on bird-view images. In this Step 6, I warped them back to normal cameral view by utilizing the Minv created during perspective transform.
The code is in Cell 6 of "image_process.py" as here:
```
    road_warped = cv2.warpPerspective(road,Minv,img_size,flags=cv2.INTER_LINEAR)
```
I also put curvature radius and vehicle offset in text on the undistorted image, using the following codes:
```
    cv2.putText(result, 'Radius of Curvature = ' + str(round(curverad,3))+'(m)',(50,50), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,255),2)
    cv2.putText(result, 'Vehicle is ' + str(abs(round(center_diff,3)))+'(m)'+side_pos+' of center',(50,100), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,255),2)
```

The final look of quiz image is as below:
    
<img src="./output_images/Process/img_quiz_process.jpg" width="500">

The final look of test1 image is as below:
    
<img src="./output_images/Process/test1_process.jpg" width="500">

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The video is generated in the [video_process.py](./video_process.py) or [video_process.ipynb](./video_process.ipynb).
Here's a [link to my video result](./output1_tracked.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Road shoulder and other noises might interfere with lane line detections, resulting wrong curvature and inner lane.
An example is test4.jpg as shown here:

<img src="./test_images/test4.jpg" width="500">

The left-hand-side road shoulder still looks highlighted after color and gradient filtering, as shown here:
    
<img src="./output_images/Thresholded/Combined_Thresh/test4_thresholded.jpg" width="500">

And after perspective transform, the shoulder still have relatively high dense of non-zero pixelslooks, similar with left lane line.

<img src="./output_images/Warped/warped_images/test4_thresholded_bv.jpg" width="500">

Increasing the gradx threshold might help. The following image is test4 with gradx min = 40

<img src="./output_images/Try/Combined_Thresh/test4_thresholded.jpg" width="500">

The warped test4 looks also better with gradx min = 40.

<img src="./output_images/Try/warped_images/test4_thresholded_bv.jpg" width="500">


```


```python
### Improvement Discussion (After First Submission)

#### Comments from viewer:

The radius of curvature seems greatly underestimated in the resulting video, as values between 400 and 1500 meters are expected in the curved stretches. This lesson provides a visual representation for this.

Suggestion:

I recommend reviewing the meter per pixel ratios, using the warped images as a reference.

To prevent a slight wobbling of the lane causing a great deviation here one alternative is calculating the curvature of both lanes and showing the average.

For further information and an example, check this lesson.

#### Modifications been made:

Here I have double checked the warped image. A single piece of dashed-line in detected lane lines usually takes 1/10 of y-axis pixels.
Since the length of one dashed-line on road is 3 meters. The y axis meter per pixel can be calculated as 30/720.

The lane width normally takes 500 pixels on the x axis. Thus the x axis meter per pixel is set as 3.7/500.

To avoid wobbling, I have averaged both left and right lane line curvature radius as below:
```
    curve_fit_left = np.polyfit(np.array(res_yvals,np.float32)*ym_per_pix, np.array(leftx,np.float32)*xm_per_pix, 2)
    curve_fit_right = np.polyfit(np.array(res_yvals,np.float32)*ym_per_pix, np.array(rightx,np.float32)*xm_per_pix, 2)
    curverad_left = ((1+(2*curve_fit_left[0]*yvals[-1]*ym_per_pix + curve_fit_left[1])**2)**1.5)/np.absolute(2*curve_fit_left[0])
    curverad_right = ((1+(2*curve_fit_right[0]*yvals[-1]*ym_per_pix + curve_fit_right[1])**2)**1.5)/np.absolute(2*curve_fit_right[0])
    curverad = (curverad_left+curverad_right)/2
```

The snippet of code can be found in both [image_process.ipynb](./image_process.ipynb) and [video_process.ipynb](./video_process.ipynb)

A new video based on the above adjustment is generated as [link to my video result](./output2_tracked.mp4)

It can be found that in the new video, the curvature radius of most frames have been corrected back to the normal range of 400 to 1500 meters.
There are still couple frames when the lane line is very straight and the radius reachs over 5000 meters. 
For future improvement, I think efforts can be spent on averaging the radius among neighbouring frames and filtering out outlining radius results. 
```
