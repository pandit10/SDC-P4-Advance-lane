## Advanced Lane Finding Project

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

[image1]: ./md_images/undist_chess.png "Undistorted"
[image2]: ./test_images/undistorted/test2.jpg "Road Transformed"
[image3]: ./test_images/binary/test2.jpg "Binary Example"
[image4]: ./md_images/perspective_test2.png "Warp Example"
[image7]: ./md_images/histogram.png "histogram"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/test2.jpg "Output"
[video1]: ./project_video_lanes.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in lines 6 through 36 of the file called `P4.py`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 60 through 100 in `P4.py`).  Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is included in the function 'perspective_transform()' in the lines 106 through 130 in 'P4.py'.  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. If only an image was passed to the function, default 'src' and 'dst' values are calculated as a function of image sized in the following manner:

```python
if M is None:
    if src_in is None:
        src = np.array([[575. / 1280. * img_size[1], 460. / 720. * img_size[0]],
                        [705. / 1280. * img_size[1], 460. / 720. * img_size[0]],
                        [1127. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                        [203. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)
    else:
        src = src_in
    if dst_in is None:
        dst = np.array([[320. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                        [960. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                        [960. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                        [320. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)
    else:
        dst = dst_in
```

The result in a 720x1280 image is the following source and destination points:

| Source        |-------->| Destination   | 
|---------------|-|---------------| 
| 575, 460      |.| 320, 0        | 
| 203, 720      |.| 320, 720      |
| 1127, 720     |.| 960, 720      |
| 705, 460      |.| 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After undistorting the image, extracting binary data, and applying perspective transform, the next required step is to extract pixels that are associated to lane lines. This was done using the sliding window search, with the start point taken as the histogram peak of the lower imaege half shown in the figure below.
![alt text][image7]

Then I fit my detected lane pixels with a 2nd order polynomial as show in the below figure:

![alt text][image5]

This is done in the function `find_lanes()` in the code lines 197 through 271 in `P4.py`.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I also did this in the function `find_lanes()` in the code lines 279 through 289 in my code in `P4.py` by defining conversions in x and y from pixels space to meters, fitting new polynomials in the world space and calculating their new radii of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 160 through 194 in my code in `P4.py` in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_lanes.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### Challenges
* The first major challenge I've faced is the binary data extraction. First, I used Sobel X operator combined with a thresholded S-channel from the HSV colour space. I implemented my first prototype with this extraction method. However, it has failed in 2 frames on the project_video file (approx. 21 and 41 sec.). Accordingly, I migrated to another method of extraction, taking into account 3 masks; white, yellow and gradient: 
  - The white mask was extracted from and RGB colour space image within the range `(200,200,200)` to `(255,255,255)`, which perfectly detected white lane lines.
  - The yellow mask was extracted from the HSV colour space image within the range `(0,100,100)`to `(80,255,255)`, which perfectly detected yellow lane lines.
  - The gradient mask was extracted using the sobel operator in the X direction with thresholds `30` to `150`, which perfectly detected all vertical lines missed by the yellow and white masks due to shadows
, which perfectly detected white lane lines. and different light conditions.
* The second challenge was finding a way to implement the lane update function instead of executing a blind search for every frame. The `Line()` was really helpful for storing data that is accessible by every function in the pipeline.

##### Further improvements 
* The binary data extraction method used is very limited to near-perfect light condition. This is the main reason of the system failure when testing using the challenge video. A further tuned binary extraction function would definitely lead to a more robust perception.
* Further sanity checks could help improve the tracking quality by raising a flag indicating a corrupt detected lane, initiating a new blind search.
