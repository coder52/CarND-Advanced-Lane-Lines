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
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![undistorted test1][./output_images/undistorted/test1.jpg]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one: I have written a function for making undistortion. In this function by cv2.calibrateCamera() function I took matrix (mtx) and distortion coefficients (dist) for undistortion. Then I used them in cv2.undistort() function and get undistorted image.This function is in 2nd cell in the IPython notebook.You can see many examples in 3rd, 5th, and 7th cells and also in folder "./output_images/undistorted/"
          
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # 8 # and in `thresholds.py`).  In this function I used sobel absolute mask in x dimension and HLS color mask in S channel. You can see an example picture in the 10th cell and many examples in folder ".\output_images\undistorted-thresholded". We use this function to make the stripe lines look the best in all conditions.
Here's an example of my output for this step.

![./output_images/undistorted-thresholded/test5.jpg][./output_images/undistorted-thresholded/test5.jpg]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `p_transfer()`, which appears in the 12th code cell of the IPython notebook.  The `p_transfer()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[[679, 447],        # top right (x,y)
                   [1090,700],        # bottom right (x,y)
                   [225, 700],        # bottom left (x,y)
                   [600, 447]]])      # top left (x,y)

dst = np.float32([[[850, 0],          # top right (x,y)
                   [850, 720],        # bottom right (x,y)
                   [250, 720],        # bottom left (x,y)
                   [250, 0]]])        # top left (x,y)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 679, 447      | 850, 0        | 
| 203, 700      | 850, 720      |
| 225, 700     | 250, 720      |
| 600, 460      | 250, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. You can see a good example at 14th cell and also many examples in folder '.\output_images\undistorted-transformed'.

![alt text][./output_images/undistorted-transformed-thresholded/test2.jpg]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this: I wrote a line finder  cell at 17th cell for testing lane lines. The '`find_lane_pixels ()`' function in cell 18 is used to find the pixels of the path lines.Here we first determine the x coordinates of the lane lines by summing the pixel values along the column.Then, as in `line_finder ()`, we can see if the pixels actually belong to the lines from their height on the y-axis.We divide the screen from top to bottom into equal windows and take the x coordinates of the lines we find with the help of the histogram as the starting point.Then, if the number of white pixels in the window is more than a certain threshold value, we move upwards by accepting the average of the x coordinates of these pixels as the center of the next window. And so, we are trying to get all pixel coordinates of highway lines on the picture. In cell 19, the function named `fit_polynomial ()` obtains the coordinates from the `find_lane_pixels ()` function and the constants of the second-order equation of a curve using the function `np.polyfit ()`Road lanes show continuity along the y-axis in the pictures.Therefore, if we put all the coordinates of the y-axis in the polynomial equation, we find the x coordinates of the curves representing the lane-lines. We draw this curve on the screen with the `plt.plot ()` function. We can also adjust the color of the pixels as desired using their coordinates. The `search_around_poly ()` function in cell 25th likewise allows us to color a certain range of pixels along the curve drawn by the polynomial.You can see examples for that between 21-28 cells and '.\output_images\undistorted-transformed-thresholded-pipe' , '.\output_images\undistorted-transformed-thresholded-windowed' folders.

![alt text][./output_images\undistorted-transformed-thresholded-pipe/test3.jpg]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In 31st cell, I implemented 'radii_offset()' function. Bu işlev ayrıca 'find_lane_pixels ()' işlevinden elde edilen koordinatları alır ve bu koordinatları her iki dizenin yarıçapını bulmak için dairenin yarıçap formülünde kullanır.To do this, he finds polynomial constants representing the lane-lines with the help of 'np.polyfit ()'.Using the polynomial function, the coordinates of the lane-lines are found for the start of x (for y = max), which can be used to calculate the position of the car on the road.The function stores the healthy values in the 'class ()' structure created in cell 29 and uses it to obtain more healthy and stable values on the video.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 34 and 36 in my code in the function `join_together()`.  You can see the example in the 38th cell.This function adjusts the pixels between the two lane-lines to the desired color using the coordinates of the lane-line pixels and places them on the original image with a reverse perspective transform (Minv).

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

For example, these codes will not work on roads where lines do not appear due to bad weather conditions. In such cases, Deep Learning models can be considered to predict the location of the lane lines.

The region of interest should be able to change according to the speed of the car. Speed ​​can be estimated by tracking the movements of lane lines.

The codes I wrote in this project can be made a bit more organized. For example, Line related functions can be added to the `Line()` class. And other functions can be defined in a separate class by dividing it into smaller functional parts. 

The windows found in the `find_lane_pixels ()` function are a bit more improvable. For example, when small windows enter an indeterminate field (as in test1 test2 and test3 pictures), the deviation can be reduced by using the information of the previous windows or with the information of the other strip. 

The lane line finder function (by using Canny Edge Detection and Hough Lines ) that we used in the previous project and The `line_finder ()` function in this project can be combined to create a healthier lane finding function.

The noise in the numbers displayed on the screen can be reduced by adjusting the parameters in the function `radii_offset()` written to find the radius and by freezing a constant value when the path is flat (ie when the radius is too large).



