## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

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

[image1]: ./writeup_images/undistort.png "image1"
[image2]: ./writeup_images/binary_transform.png "image2"
[image3]: ./writeup_images/warp.png "image3"
[image4]: ./writeup_images/window_fitting.png "image4"
[image5]: ./writeup_images/histogram.png "image5"
[image6]: ./writeup_images/curve_fitting.png "image6"
[image7]: ./writeup_images/drawing_lane_lines.png "image7"
[image8]: ./writeup_images/writing_info.png "image8"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in [Advnced_lane_lines.ipynb](https://github.com/R-EM/CarND-Advanced-Lane-Lines/blob/master/Advanced_Lane_Lines.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function. The result of the calibration can be seen below when undistorting the test image.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
Here, I begin with loading all the calibration images, using them for calibration, the result of the calibration can be seen below on the test image:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Cell 5 shows the binary transform function, using a combination of color and gradient thresholds to generate a binary image. Here's an example of my output for this step.

![alt text][image2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
The warp function in cell 7 takes an image as input.  I chose the hardcode the source and destination points in the following manner:

```python
img_size = (img.shape[1], img.shape[0])  # Image size
bot_width = .76  # percent of bottom trapizoid height
mid_width = .1  # percent of middle trapizoid height
height_pct = .62  # percent for trapizoid height
bottom_trim = 0.935 # percent from top to bottom to avoid car hood

src = np.float32([[img_size[0]*(.5-mid_width/2), img_size[1]*height_pct],
                 [img_size[0]*(.5+mid_width/2), img_size[1]*height_pct],
                 [img_size[0]*(.5+bot_width/2), img_size[1]*bottom_trim],
                 [img_size[0]*(.5-bot_width/2), img_size[1]*bottom_trim]])

offset = img_size[0]*.15
dst = np.float32([[offset,0],
                  [img_size[0]-offset,0],
                  [img_size[0]-offset,img_size[1]],
                  [offset,img_size[1]]])
```


I verified that my perspective transform was working as expected by using it on a test image, as can be seen below.

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I begin by taking the histogram of the bottom half of the binary warped image (Cell 9) as can be seen below

![alt text][image5]

I then proceed to use the highest value of the left and right side of the histogram as the starting point, and use a sliding window search algorithm to locate the lane lines (Cell 10), as can be seen below

![alt text][image4]

Finally, in cell 12 the function for fitting a second order polynomial can be seen, and the result can be seen below

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Cell 14 contains two functions, one for getting the radius of the lane lines, and one for measuring the distance from the middle. Note that only the left lane line was used for this calculation.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Cell 15 contains a function for drawing the lines, and warping the image back to the way it was before warping it in the first place. The result can be seen below

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Initially, my pipeline work quite well, up untill the lighting conditions became brighter and the lane lines were somewhat more difficult to find. The code for my pipeline can be found in cell 18, and here I've done a few things to help improve its performance.

First, I began by checking for large spikes in curvature differences, between the current and the previous value. Here, and for all modifications, if the change is undesirable, the previous curvature is used. The change in curvature was considered undesirable if the absolute value was greater than the value of the variable called "limit", located in cells 19 and 20. The current value for this limit is 5000.

Next, I checked the sign of the radius curve, and the difference between the previous value and the current one. If this value is large, and the direction has changed, then this is clearly an unwanted change since roads don't usually change direction that quick. 

Finally, I check for sudden changes in x coordinates. If the changes were too large, the previous x coordinates were used instead. The varables pxDist and pxDistTol are used here, where pxDist is the distance between the lane lines, and pxDistTol is the amount of pixels that the lane lines can move. So the distance between the left and right lane lines should have a pixel distance of pxDist, and move plus minus pxDistTol. The values used for pxDist is 600 and pxDistTol is 120. This means that the maximum distance between the lane lines should be 720 pixels, and the minimum distance should be 480. If this distance is greater than the maximum, or lower than the minimum, the previous x coordinates were used. This is done for every x coordinate 
