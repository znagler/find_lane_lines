# Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[chess1]: ./output_images/original_chessboard.jpg "Original"
[chess2]: ./output_images/undistorted_chessboard.jpg "Undistorted"
[image1]: ./output_images/test3.jpg "Original"
[image2]: ./output_images/undistort-3.jpg "Undistorted"
[image3]: ./output_images/sobel-3.jpg "Sobel Example"
[image4]: ./output_images/hls-3.jpg "HLS Example"
[image5]: ./output_images/sobel_or_hls-3.jpg "Binary Example"
[image6]: ./output_images/warp-3.jpg "Warp Example"
[image7]: ./output_images/sliding_window-3.jpg "Fit Visual"
[image8]: ./output_images/lined_original-3.jpg "Output"
[video1]: ./output.mp4 "Video"


**Please see all the code in the [IPython Notebook](./project.ipynb)**

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In Part 1 of the [IPython Notebook here](./project.ipynb), I calculated the camera matrix and distortion coefficients with the help of a couple OpenCV functions and 16 chessboard images.  I first detected the coordinates of the chessboard corners in all of the images with `findChessboardCorners`, and added them to a list.  Using those coordinates, along with a list of incrementing grid coordinates, `calibrateCamera` is able to calculate the distortion and return the camera matrix and distortion coefficients.  The OpenCV method `undistort` uses those two values to undistort images from that camera.  Here's the before-and-after of the undistortion. 

![alt text][chess1]
![alt text][chess2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
I used the same values to undistort road images.  Here, the result is more subtle, but it's still an important step for finding lane lines accurately.

![alt text][image1]
![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Part 2 of the notebook uses a few computer vision techniques to illuminate the lane lines as much as possible.  The first is an application of a Sobel filter for edge detection in both the X and Y directions.  After the filter, I also convert the image to binary– each pixel is 'yes' or 'no' depending on the amount of edge detected and a provided threshold.  Picking good threshold values is a matter of experimentation and eyeballing.  Here's what the same image looks like with the Sobel threshold function applied.

![alt text][image3]

In the next cell of Part 2, I define a function to convert the image to the HLS color space and look specifically at the S channel.  This channel is good for ignoring differences in shadow and lighting on the road.  A similar binary threshold is taken.  Here's the result.

![alt text][image4]

To take advantage of both of those techniques, I create a new binary image that labels a given pixel 'yes' if it was 'yes' for either Sobel or HLS.

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Skipping ahead for a moment to the pipeline in Part 4, I use OpenCV's `getPerspectiveTransform` and `warpPerspective` to change the perspective of the image to a bird's eye view in order to focus directly on the lane lines.  This method takes coordinates of input and output quadrilaterals, which are shown below.  The input is trapezoidal because of the way lane lines appear to get closer together as they are more distant, and the output is rectangular to undo that effect.


| Inputs        | Ouputs   | 
|:-------------:|:-------------:| 
| 585, 460      | 280, 0        | 
| 695, 460      | 1000, 0       |
| 1120, 720     | 1000, 720     |
| 695, 460      | 280, 720      |

This is what the image looked like after the perspective transform.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The sliding window method at the start of Part 3 handles the final piece of lane line pixel identification.  The strategy involves cutting the image into 9 horizontal slices, and for each slice, trying to figure out the X value of the lane lines.  This is done by taking a histogram at each slice that counts how many white "yes" pixels there are per X value.  The peaks in the histograms should be the locations of the lane lines.  White pixels near the peaks are pushed into lists, and the process is repeated on each slice.  Using the identified pixels, two quadratic equations are fit with numpy's `polyfit`, one per lane.  Margins are applied to the quadratics to give the lanes realistic widths.  The image below shows the datapoints of the lane lines, along with the quadratics and shaded regions between the margins.

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate curvature, pixels are translated to meters using a rough unit conversion.  The left and right lane quadratics are re-fit in the new units, and the radius is calculated using those quadratics and the formula for determining radius shown [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php).  For simplicity, the mean of the left and right radius measurements is taken and displayed.

For the car position, two points are compared– the center of the image and the 'projected center' between the lane lines.  The projected center of the lane lines is calculated by seeing what X values the left and right quadratics output for the Y values at the very bottom of the image, and taking their mean.  Both of these can be found in Step 7 of the Part 4 pipeline.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The second method in Part 3 is responsible for bringing the lane lines back to the image in the original perspective.  It uses the same `warpPerspective` method, but this time with the inverse of the first warping matrix for the matrix-multipy.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

More could be done in a few places to improve the result.

1. The Sobel and HLS methods do a decent job of illuminating lane line pixels, but they could certainly be improved by tweaking the binary thresholds.  Also, while the color channel used (S in HSL) is probably the most useful, it is certainly not the only useful one, so adding others from the same or different color spaces could help.

2. The pipeline doesn't consider information from previous frames.  While it is nice for testing to have a method that treats every frame independently, it doesn't maximally use the information at hand.  This would be like a person re-evaluating his steering angle every fraction of a second instead of using his memory.  A smarter pipeline would consider previous frames and take them into account with some sort of weighted average.

3.  Quadratics cannot capture more than one curve, so this would fail on frames where the road curves both left and right.
