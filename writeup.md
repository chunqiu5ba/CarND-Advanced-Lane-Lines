## Project Advanced Lane Lines

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

[image0]: ./output_images/chessboard.png "Undistorted"
[image1]: ./output_images/Undist1.png "Undistorted"
[image2]: ./output_images/Undist2.png "Udistorted Road"
[image3]: ./output_images/process_binary.png "Binary Example"
[image4]: ./output_images/Unwarp.png "Warp Example"
[image5]: ./output_images/process_bird.png "Bird Eye"
[image6]: ./output_images/sliding_windows.png "Sliding Windows"
[image7]: ./output_images/histogram.png "Histogram"
[image8]: ./output_images/final_output.png "Final Result"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (9, 6, 3) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (9, 6) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Here is the chessboard corners visualized:

![alt_text][image0]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradient X/Y thresholds (cell 6),Color channel HLS S, HSV V (cell 9), and white color mask(cell 10) to generate a binary image (summarized in code cell 13, function `binary()`). Other mothod like Magnitude and Direction Threshold were also explored and finally dropped out from the process as those mothod will invite much more noise.
Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears in the 5th code cell of the IPython notebook.  The `unwarp()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```python
h,w = img.shape[:2]
offset = 300
src = np.float32([(0.46*w,0.62*h), (0.54*w, 0.62*h), (0.88*w, 0.95*h), (0.12*w, 0.95*h)])
dst = np.float32([(offset, 0), (w-offset, 0), (w-offset, h), (offset, h)])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 589, 446      | 300, 0        | 
| 691, 446      | 980, 0        |
| 1126, 684     | 980, 720      |
| 154, 684      | 300, 720      |

I verified that my perspective transform was working as expected that the lines appear parallel in the warped image.

![alt text][image4]

Here is the visulization of warped image on binary images:

![alt text][image5]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Line finding on a single image or the first frame of the video is done via 9 sliding windows, which position is located by historgram peak then slided through the vertical line, with a margin 100. Line fitted by a second order polynomial show in yellow.


![alt text][image6] 


![alt text][image7] 

Once line is found on first frame, silding window is not used any more. I just update lines based on the previous fit position.
This is implemented in code cell 13.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Continued in cell 13, raduis curvature is calculated by math described here: http://www.intmath.com/applications-differentiation/8-radius-curvature.php

I take the average of right radius and left radius as the radius displyed on the final image output. Vehicle position is calculated by the differenct between image center and mid-point of two lines. 

Pixel to meter ratio is used:

ym_per_pix = 30/720 # meters per pixel in y dimension

xm_per_pix = 3.7/700 # meters per pixel in x dimension

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 15, in the function `draw_lines()`.  Here is an example of my result on the test images:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

[Link to project video result](./project_video_output.mp4)

[Test result on Challenge Video](./challenge_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I used HLS S channel to capture the Yellow line, HSV V channel and white color mask to captuer the White line, helped with gradient X and Y threshold to filter out some noises. It worked very well on project videos.

However the pipeline worked badly on challenge video. It could be imporved by:
1, make fit hight area smaller and add a fixed interest region mask to cut off outsider pixels which will cause problem
2, implemnt outlier rejection
3, averge dawelines over 20 or more frames to make the line movement smooth.
