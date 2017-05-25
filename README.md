
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

[image1]: ./examples/undist0.png "Undistorted"
[image2]: ./examples/undist1.png "Undistorted"
[image3]: ./examples/thresholdonly.png "Threshold Only"
[image4]: ./examples/gradientmag.png "Gradient Magnitude"
[image5]: ./examples/gradientdir.png "Gradient Direction"
[image6]: ./examples/combinedgradient.png "Combined Gradient"
[image7]: ./examples/HLS.png "HLS Color Space"
[image8]: ./examples/finallane.png "Final lane detection"
[image9]: ./examples/perstrans.png "Bird eye view"
[image10]: ./examples/binarizewarp.png "ROI warped"
[image11]: ./examples/laneploy.png "Fit lane line"
[image12]: ./examples/finaltestimage.png "Lane line projection"
[image13]: ./examples/advanceLaneLines.gif "Lane line video"
[image14]: ./examples/challengeLaneLines.gif "Lane line video"










## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Once the camera matrix and distortion coefficients are learned. I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image, see section **Define gradient and color transforms to help detect lane lines** in the python notebook.  Here's an example of my output for this step. note: this is from test images 5 which contains curvy lane lines as well as shades in the lane; S color space helped finding the yellow lane lines and L channel threshold removed the impact of shades; with the help of directional Sobel gradient the lane line detection is clear in the final combination.

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image8]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

See **perspective transform** section in the IPython notebook. The straighlines1 test image has been picked because it has straight lane lines which correponds to a rectangle in the birdeye view. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[592,450],[688,450],[1125,720],[190,720]])
offset = 250 # offset for dst points
dst = np.float32([[src[0,0] - offset, 0], [src[1,0] + offset, 0], \
                      [src[1,0] + offset, src[2,1]], [src[0,0] - offset, src[3,1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 592, 450      | 342, 0        | 
| 688, 450      | 938, 0        |
| 1125, 720     | 938, 720      |
| 190, 720      | 342, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image9]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

See the section **Binarize the image and ploynomial fit the lane line**; the input are undistored and binarized; I also applied region of interest before change perspective to top down view. Then I fit my lane lines with a 2nd order polynomial.


![alt text][image10]
![alt text][image11]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in section **Calculate the curve radius and offset to centor of road in unit of meters**; basically x and y vectors are scaled from pixel space to meter space then fit again.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of my result on a test image:

![alt text][image12]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

A video processing pipeline has been developed to search the lane line within a margin of previous detection. Some sanity checks are introduced to prevent catastrophic failures. Basically I used previous 5 detections as well as some other threshold from empirical study to prevent failed detection from misleading the following iterations. The final pipeline is robust enough to handle different potential failures. The curvature radius and offset stats are also reported correctly most of the time. More details please see section **Video Pipeline and Sanity check of detected lane lines**.

![alt text][image13]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

* Shades in the road: Introduce L channel in HLS color space. it worked very well in removing shades which are low values in the L channel.
* Long duration of shady area in the road: Sanity checks of the detected lane lines can help. Basically if the detected lane lines vary a lot from previous ones, then we have to use the previous N good detection to infer the next one instead of following the wrong detection.
* Wobbly lines: this can also be avoided effectively by the smoothing from the last N datapoints.
* Challenge and harder video: they are closer to real life scenarios. the carpool lane sign on the ground, different color inside the lane, ascending landscape, slopes, mirror of the dashboard on wind shield, sharp turns, other vehicles; these are all distractions;

in order to get a better result, I will need to redo the camera calibration and perspective transformation and come up with a better approach for lane line detection as well as the video pipeline sanity check. One thing worth trying would be create a set of profile of lane detector parameters and attempt a different profile when the pipeline failed, we need a better classification based on the cause of failure like the current pipeline prints the reason it failed the sanity check; there might be a mapping that profile A is helpful in solving failure signature B. Given this is a mapping from X->Y problem. I think deep neural network might shine here.

![alt text][image14]

