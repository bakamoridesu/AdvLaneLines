
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

---

#### Note: this writeup briefly covers every step. More detailed comments and pictured examples of every substep can be found inside the notebook.

### 1. Camera Calibration

The first step is to read in calibration images of a chessboard.

![Gray image example](/pictures/camera_cal.png)

Each chessboard here has 9x6 corners to detect.
For camera calibration I used the `opencv`'s `calibrateCamera` function which requires two arrays as input values: 
- 3D points in real world space (image points)
- 2D points in image plane(object points)
So, for it calibration image I had to create these two arrays.
The object points are all the same. Just the known object coordinates of the chessboard corners for an 9 by 6 board. These points are 3D coordinates x, y and z from the left top corner (0,0,0) to the bottom right corner (8,5,0). The z-coordinate is 0 for every point, since the board is on a flat image plane. And x and y are all the coordinates of the corners.
Next, to create the image points, I used `opencv`'s `findChessBoardCorners` on the distorted image. To draw found corners I used the `drawChessBoardFunction`, and here is the result of finding corners:

![Gray image example](/pictures/detected_corners.png)

Now that I have image points and object points for every image where the corners were found, I can get the calibration coefficients and the camera matrix that I need to transform 3D object points to 2D image points. The `opencv`'s `undistort` function performs this step, and here are the results of undistortion.

![Gray image example](/pictures/undistortion_chess.png)

![Gray image example](/pictures/undistortion_road.png)

### 2. Gradient and color selecting.
#### 2.1 Gradient threshold
Next step is about finding the edges in an image which are most likely being lane lines.
But first I want to collect some images for tests. Namely I need images containing:

- straight lines
- curved lines
- dotted, sparsed lines

Here are some examples that I found useful for tests:

![Gray image example](/pictures/test_images_str.png)

![Gray image example](/pictures/test_images_cur.png)

Firstly I used the `opencv`'s `Sobel` function for finding edges in a picture. I calculated the vertical and horizontal gradient, and also gradient's magnitude and direction:

![Gray image example](/pictures/h_v_gradient.png)

![Gray image example](/pictures/mag_dir_gradient.png)

Then I tried different combinations of those techniques. Eventually i found out that the best combination is: `(vertical_gradient AND horizontal_gradient) OR (gradient_magnitude AND gradient_direction)`.

![Gray image example](/pictures/combo_grad.png)

#### 2.2. Color threshold.

For the color thresholding there were not any scientific approaches, I just tried out every single color space to overcome partial shadows and different line colors. The result was just OK, but if I had all the time in Universe, I could probably do something better. 

![Gray image example](/pictures/color_tr.png)

#### 2.3. Combining thresholds.

Here is the result of combining color and gradient thresholds:

![Gray image example](/pictures/combbo_tr.png)

### 3. Perspective transformation.

For performing the perspective transformation I just picked an image with straight lines. These lines are supposed to be strict and parallel to each other. So slicing them with another two parallel lines which are perpendicular to them will result a rectangle. I found the four points of the larger rectangle that, after perspective transform, made the lines look straight and vertical from a bird's eye view perspective.

![Gray image example](/pictures/pers_trans.png)

### 4. Finding lane boundary.
First I needed the starting point of the lines. For this I took the very bottom of the image since there were still some large noize in curved images.
Here are the examples of an image and it's very bottom.

![Gray image example](/pictures/boundary1.png)

The pixel histogram of the image's bottom looks nice. There is no noise, just two solid peaks. 

![Gray image example](/pictures/hist.png)

But anyway to avoid noize I start from the center of the image and search for the closest peaks with a small convolutional window. First peak to the left means the left line and first to the right means right line (and of course I use a threshold for peaks). So, the starting points for lines are found. Next, I move convolutional windows up to find the rest of the line pixels.

![Gray image example](/pictures/boundary2.png)

Finally, I fit a second order polynomial to each line.

![Gray image example](/pictures/boundary3.png)

After that I draw the lane between the lines using `opencv`'s `fillPoly` function. And here are the results on my test images:

![Gray image example](/pictures/result_images.png)

### 5. Measuring Curvature


### 5. Pipeline (video)

For the first run 





### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
