## Writeup

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test4.jpg "Road Transformed"
[image3]: ./out_examples/binary_combo_example.jpg "Binary Example"
[image4]: ./out_examples/warped_lines.jpg "Warp Example"
[image5]: ./out_examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./out_examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in 2. code cell(define a function 'def get_mtx_dist()' for calibration) and 4. code cell(run the function) in `lane_lines_fineder.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function, in order that i could use 'cv2.undistort()' function to get undistorted camera images.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color(RGB & HLS) and gradient thresholds(x, direction) to generate a binary image (thresholding steps in 5. code cell and thresholding function in 2. code cell(line number #31-113) in `lane_lines_fineder.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

It was improved when i apply the thresholds for videos. Because of some extreme situation i need to tune more thresholds and parameters, e.g. for yellow lines detecion i added a RGB threshold.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform_and_warp(img, src)`, which appears in 2. code cell at lines 115 through 124 in the file `lane_lines_fineder.ipynb`.  The `perspective_transform_and_warp(img, src)` function takes as inputs an image (`img`), as well as source (`src`). `dst` would be defined in the funtion, because in this project we use same size images. I chose the hardcode the source and destination points in the following manner:

```python
src_ini = np.float32([[540,480],[180,690],[1170,690],[780,480]])
shape = img.shape[::-1]
w = shape[0]
h = shape[1]
dst = np.float32([[0,0],[0,h],[w,h],[w,0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 540, 480      | 0, 0        | 
| 180, 690      | 0, 720      |
| 1170, 690     | 1280, 720      |
| 780, 480      | 1280, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

The code for line pixels finding located in 2. code cell at lines #126-230. The input of the function is warped image. At first, I used the histogram to get peaks in warped image. Then I apply the sliding windows to find the pixels and fit their postions with a polynomial. The outputs of the function are the fitted polynomial functions of left and right lines and the x-position of left and right lanes. Some condition statements were added when i used the function to process videos.

The code for processing an example appears in 10. code cell.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this to an example in 11. code cell. The referenced function is in 2. code cell at lines #316-328.  The function took img, x-positions of left and right lines as input. At first, i used img to get the shape in order to divide it in y-achse into evey pixel unit. Then i transformed the image pixels into real world distance using the known relative functions. after that, i fit the transformed x-y with a new polynomial and got the x postion when y is maximum. With the funtion we have already in course seen, i finally got the curvature of the driving lane in real world.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 11. code cell with the function(draw_lane()) described in 2. code cell at lines #330-355.  Here is an example of my result on a test image:

![alt text][image6]

With x positions of left and right lines, the inverse matrix of perspective transform i could finally draw a green region in the original undistorted image. It fits the lane pretty well! 

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my project video result](./output_videos/project_video.mp4)
Here's a [link to my challenge video result](./output_videos/challenge_video.mp4)


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The most difficult point of this project is finding a robust lane among frames of videos. When processing 1. video and challenge videos, i met many problems. The lane lines were so instable, before i used search_around_poly() function and added a bunch of condition statements. In my code, i have to check if the left line or right line is found, if the found lines were good enough to fit the lane(with running average restriction and x-intercept restriction). I saved 15 frames lines to smooth the final behavior of lane finding. When illumination changing, the lane lines detection could also be instable. I tuned my parameters of thresholds and added some new thresholds to solve this. It took a lot of time to test and tune. 

But with harder challenge video i failed to find lines. It was full with extreme situation. I think i should change the whole codes...(change/reduce the region of perspective transform, because in this video there is so much curve driving; Re-tune the thresholds and paramters to find lane lines easier; Restrict the condition statements to make the lane finding more robust). But at the end it worked not well. 