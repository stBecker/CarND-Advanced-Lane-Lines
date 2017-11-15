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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/road_undistorted.png "Undistorted"
[image3]: ./output_images/binary_combo_example.png "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image4b]: ./output_images/warped_tresh.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image5b]: ./output_images/color_fit_lines2.png "Fit Visual"
[image6]: ./output_images/overlay.png "Output"
[image7]: ./output_images/overlay2.png "Output"
[video1]: ./output_images/project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3. code cell of the IPython notebook located in "advanced_lane_finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, 
such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it 
every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image 
plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  
I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The calculated calibration and distortion coefficients can now be used to undistort the camera images mounted inside the car.

    img = cv2.imread("test_images/test1.jpg")
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    dst = cv2.undistort(img, mtx, dist, None, mtx)


Which results in the following undistorted image:

![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

First I convert the image to HLS color space. I use a Sobel filter in the x direction on the lightness channel and
apply a threshhold to the result; I also apply a threshhold to the saturation channel. The combined pixel masks are
the output of the treshholding function (cell 6 in the notebook).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used an interactive matplotlib graph of the straight_lines1 test image to identify a rectangular shape within the image
(i.e. a shape that would be rectangular when viewed from above, in the image itself it is a trapezoidal shape).
I found the following source and destination shapes to work resonably well in transforming the perspective:

    src = np.float32([
            [223, 692],
            [1074, 692],
            [590, 452],
            [688, 452],
        ])
    dst = np.float32([
        [img_size_x/2 - offset, img_size_y],
        [img_size_x/2 + offset, img_size_y],
        [img_size_x/2 - offset, 0],
        [img_size_x/2 + offset, 0],
    ])

The complete function is given in cell 10.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a 
test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

Using both threshholding and perspective transforms gives the following result:

![alt text][image4b]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code to fit my lane lines with a 2nd order polynomial is given in cell 13 and 14; in the former cell the sliding
window method is used, in the latter one the search window is around a given polynomial.
The results of both methods are visualized below:

![alt text][image5]

![alt text][image5b]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell 15 - 17 in the notebook.
The radius of curvature can be calculate by evaluating the formula with the fitted polynomial at the
maximum y-coordinate (the bottom of the image):

    left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])

The curve radius in meters can be determined by first converting the y and x pixel values to meters and
then fitting the polynomial.
The lane center is the midpoint between the left and right fitted lanes; the distance of the camera
from the lane center can simply be found by calculating how far the lane center is located from the
middle of the image:

    lane_center = (left_fit[0]*y_eval**2+left_fit[1]*y_eval+left_fit[2]+right_fit[0]*y_eval**2+right_fit[1]*y_eval+right_fit[2])/2
    distance_from_center = lane_center - binary_warped.shape[1]/2
    print("Distance in m:", distance_from_center*xm_per_pix)
    
 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 18.  Here is an example of my result on a test image:

![alt text][image6]

Additionally I added information about the detected curve radius and distance from the center to the
video output:

![alt text][image7]


The complete image processing pipeline starts at cell 19.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

During my initial attempt at the project video, the lane detection briefly failed during difficult
images and detected part of the breakdown lane; to make detection more robust to failures I added
some sanity checks to the detection (e.g. how far the detected lanes may be apart) and reused the 
detections from previous frames if the sanity check failed. I also averaged over the most reccent detections
to make the output smoother.
The pipeline seems to work reasonably well for the project video, but fails to accurately detect the lane
in the challenge videos.
