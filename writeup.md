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

[image1]: ./output_images/calibration.png "Undistorted"
[image2]: ./output_images/calibration2.png "Road Transformed"
[image3_1]: ./output_images/HLSThreshold.png "HLS Transformed"
[image3_2]: ./output_images/HSVThreshold.png "HSV Transformed"
[image3_3]: ./output_images/LABThreshold.png "LAB Transformed"
[image3_4]: ./output_images/LUVThreshold.png "LUV Transformed"
[image3_5]: ./output_images/SobelThreshold.png "Sobel Transformed"
[image3]: ./output_images/CombineThreshold5.png "Binary Example"
[image4]: ./output_images/PerspectiveTest.png "Warp Example"
[image5]: ./output_images/PolynomialFit6.png "Fit Visual"
[image6]: ./output_images/Drawdata6.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Project_Advanced_Lane_Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I compared several color spaces and gradients whether depict yellow and white lane lines well. 
![alt text][image3_1]
![alt text][image3_2]
![alt text][image3_3]
![alt text][image3_4]
![alt text][image3_5]


From these result, I choose HSL - S & L, LAB - B, and Sobel x-orient gradient channels to filter the image and threholds to generate a binary image (thresholding steps at fourth code cell of the `Project_Advanced_Lane_Lines.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

And I masked the images to erase unnecessary boundaries of images with given polygon:
```python
vertices = np.array([[(20,imshape[0]),
                    (int(imshape[1]*0.47), int(imshape[0]*0.6)), 
                    (int(imshape[1]*0.53), int(imshape[0]*0.6)), 
                    (imshape[1]-20,imshape[0])]])   
```

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 5th code cell of `Project_Advanced_Lane_Lines.ipynb`).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
# take source points as the trapazoidal vertices
src = np.float32([[(img_size[0] / 2) - 64, img_size[1] / 2 + 100],
                  [((img_size[0] / 6) + 25), img_size[1] - 40],
                  [(img_size[0] * 5 / 6) - 25, img_size[1] - 40],
                  [(img_size[0] / 2 + 64), img_size[1] / 2 + 100]]) 

# take source points as the rectangular vertices
dst = np.float32([[(img_size[0] / 4), 0],[(img_size[0] / 4), img_size[1]],
                  [(img_size[0] * 3 / 4), img_size[1]],[(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 576, 460      | 320, 0        | 
| 238, 720      | 320, 720      |
| 1041, 720     | 960, 720      |
| 704, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this (the 8th code cell of `Project_Advanced_Lane_Lines.ipynb`):

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 10th code cell of `Project_Advanced_Lane_Lines.ipynb`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 12th code cell of `Project_Advanced_Lane_Lines.ipynb`.  Here is an example of my result on a test image with curvature of load, distance from center of load, coefficients of fit:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

FIrst of the problem I encounted in this project were to find appropriate thresholing filters to mask lane lines in the images. Due to shadows, light conditions, the colors of lines are unstable and it is hard to define the range of thresholing. To overcome this problem, I utilize several color space, LAB and HSV,  to filter yellow and white colored lines. Also, the Sobel x-orient gradient is useful to identify lines far from the viewer. This pipline was quite working well to solve `project_video.mp4`, but, in `challenge_video.mp4` and `harder_challenge_video.mp4`, it was worse because of harsher conditions. This issue would be more serious with snow and rain.

The second problem is to build algorithm to define `best_fit` of lines when we did not detect it with our given filter. In the code, when I don't have the fitting coefficient, I throw out the oldest fitting in `current_fit` and try to calculate `best_fit` with remained fittings in `current_fit`. (the 11th code cell of `Project_Advanced_Lane_Lines.ipynb`) If I know the best way to update `best_fit` without appropriate estimation, it will improve the detection results even in the terrible situations. 

In the future, the dynamic thresholding would be helpful to detect lanes with abrupt environments. And, I hope to have the powerful strategies to gain `best_fit` with limited detections.

