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

[image0]: ./writeup_data/chessBoard.png "Chess Board"
[image1]: ./writeup_data/undistort.png "Undistorted"
[image2]: ./test_images/test5.jpg "Road Transformed"
[image3]: ./writeup_data/unwrap.png "Warp Example"

[image4]: ./writeup_data/absSobel.png "Absolute Sobel"
[image5]: ./writeup_data/magSobel.png "Magnitude Sobel"
[image6]: ./writeup_data/dirSobel.png "Direction Sobel"
[image7]: ./writeup_data/combinedSobel.png "Combined Sobel"
[image8]: ./writeup_data/HLS_Hchannel.png "HLS H Channel"
[image9]: ./writeup_data/HLS_Lchannel.png "HLS L Channel"
[image10]: ./writeup_data/HLS_Schannel.png "HLS S Channel"
[image11]: ./writeup_data/RGB_Rchannel.png "RGB R Channel"
[image12]: ./writeup_data/RGB_Gchannel.png "RGB G Channel"
[image13]: ./writeup_data/GrayScale.png "Gray Scale"
[image14]: ./writeup_data/combined.png "Combined Thresholds"

[image15]: ./writeup_data/slidingWinPlot.png "Sliding Window Plot"
[image16]: ./writeup_data/hist.png "Distrebution Plot"
[image17]: ./writeup_data/drawAndFill.png "Draw and Fill Lines"
[image17]: ./writeup_data/curAndDist.png "Final Plot"

[video1]: ./writeup_data/videoOutput.mp4 "Video"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./Advanced-Lane-Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

Once all the corners were found using `cv2.findChessboardCorners()`, I drew the corners on the images using `cv2.drawChessboardCorners()`. In some of the images no corners were found because the not all the chessboard boxes were visible. This is what the images look like.

![image0]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

The undistortion is mostrly noticable if you look at the hood of the car.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I chose this image beacuse it has shadows and change in street colour.

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwrap()`, in the 9th code cell of the IPython notebook.  The `unwrap()` function takes as inputs an image (`image`). Then it defines the source and destination for the transform. I chose the hardcode the source and destination points in the following manner:

```python
h, w = image.shape[:2]
upper = int(0.05*w)
lower = int(0.3*w)
height = int(0.315*h)
bottomMargin = int(0.05*h)
offset = int((1/3)*w)
source = np.float32([(w//2-upper, h-height-bottomMargin),
                    (w//2+upper, h-height-bottomMargin),
                    (w//2+lower, h-bottomMargin),
                    (w//2-lower, h-bottomMargin)])
dest = np.float32([(offset, 0),
                  (w-offset, 0),
                  (w-offset, h),
                  (offset, h)])
```

This resulted in the following source and destination points, starting from the top left corner and going clock-wise:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 576, 458      | 426, 0        | 
| 704, 458      | 854, 0        |
| 1024, 684     | 854, 720      |
| 256, 684      | 426, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I found good thresholds for all different technique thought in the lectures.

a) I used absolute value of Sobel in the x direction with threshold of (29, 104). Absolute Sobel in x diection:
![alt text][image4]

b) Then used magnitude of Sobel in the x and y directoin with threshold of (40, 115) and kernal size of 31. Magnitude Sobel in x and y direction:
![image5]

c)Then used direction of Sobel in the x and y directoin with threshold of (0.72, 1.41) and kernal size of 9. Direction Sobel in x and y direction:
![image6]

d) Combining all the Sobel Plots:
![image7]

e) HLS colour space H channel with threshold of (19, 90):
![image8]

f) HLS colour space L channel with threshold of (220, 255):
![image9]

g) HLS colour space S channel with threshold of (113, 255):
![image10]

h) RGB colour space R channel with threshold of (223, 255):
![image11]

i) RGB colour space G channel with threshold of (199, 255):
![image12]

j) Gray scale with threshold of (201, 255):
![image13]

Finally combining all the thresholds from above:
![image14]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the sliding window technique though in lecture to scan through the image and find all the 'good' pixels to include in the polynomial fit. I used polynomial of degree two. 

First I found indices for the max value in the second and third quarter vertical quarter of the image (as we expect to find the line in these region). Once I has the starting point for each lane, I considered a rectangel and found all the pixels inside that rectangle. Then I took the average of their indexes to figure out I should shift the next rectangle above. During this process I saved all the pixels I found so in the end I can do a polynomial fit on them and get the line for the lanes.

Here what the image looks like with the combined filter from above, horizontal rectangles and the polynomial fit.

![alt text][image15]

This is what the histogram for the bottom 30% of the image used for the inital points of the fit:

![image16]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To find radius of curvature I first refitted the polynomial using factors to convet distances to real world space (meters). I used the following conversion factors suggested by the instructors:

```python
yMetPerPix = 30/720
xMetPerPix = 3.7/700
```

Then I used the formula for radius of curvature that involves the coefficient of the polynomial fit provided by the udacity instructor:

$R = \frac{(1+(2Ay+B)^2)^{3/2}}{2A}$

I used the same process for both lanes.

For distance from the center, I assumed that the camera is mounted on the center of the car. Then I took the base of both the left and right lane (x-intersept of the fit) and found their midpoint. Finally I took the difference between the center of the image and the center of the two lanes.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 29 of the notebook.  Here is an example of my result on a test image:

![image17]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./writeup_data/videoOutput.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The area that had the most issues was the lane detection using different filters. The hardest part was to find thresholds that would work for all street textures and shadow areas. For the areas with no shadow and dark street, just the grayscale works well enough. However, to make it works for the entire video I had to play around with the thresholds. 

My pipeline will most lickely fail when introduced to areas with shadow as different thresholds are needed.

To improve my pipeline, I introduced an averaging approach, where I averaged the fit over few previous frames including the current frame.

To improve the current implementation, a more dynamic threshold values can be assigned that change based on the frame rather than being cardcoded. Add a more robust averaging system for the fits that remove any fits that are not desired. 
