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

The code for this step is contained in the first code cell of the IPython notebook located in "./Advnced_lane_lines.ipynb".  

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

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
