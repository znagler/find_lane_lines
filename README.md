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

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In Part 1 of the [IPython Notebook here](./project.ipynb), I calculated the camera matrix and distortion coefficients with the help of a couple OpenCV functions and 16 chessboard images.  I first detected the coordinates of the chessboard corners in all of the images with `findChessboardCorners`, and added them to a list.  Using those coordinates, along with a list of incrementing grid coordinates, `calibrateCamera` is able to calculate the distortion and return the camera matrix and distortion coefficients.  The OpenCV method `undistort` uses those two values to undistort images.  Here's the before-and-after of the undistortion. 

![alt text][chess1]
![alt text][chess2]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
I used the same values to undistort road images.  Here, the result is more subtle, but it's still an important step for finding lane lines accurately.

![alt text][image1]
![alt text][image2]


####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Part 2 of the notebook uses a few computer vision techniques to illuminate the lane lines as much as possible,  The first is an application of a Sobel filter for edge detection in both the X and Y directions.  After the filter, I also convert the image to binary– each pixel is 'yes' or 'no' depending on the amount of edge detected and a provided threshold.  Picking good threshold values is a matter of experimentation and eyeballing.  Here's what the same image looks like with the Sobel threshold function applied.

![alt text][image3]

In the next cell of Part 2, I define a function to convert the image to the HLS color space and look specifically at the S channel.  This channel is good for ignoring differences in shadow and lighting on the road.  A similar binary threshold is taken.  Here's the result.

![alt text][image4]

To take advantage of both of those techniques, I create a new binary image that labels a given pixel 'yes' if it was 'yes' for either Sobel or HLS.

![alt text][image5]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
