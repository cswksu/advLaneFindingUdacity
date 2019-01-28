# Advanced Lane Finding
## The Project

This project calibrates a front-facing vehicle camera and identifies lane lines from 1280x720 video shot on said camera.

The goals / steps of this project are the following:
*	Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
*	Apply a distortion correction to raw images.
*	Use color transforms, gradients, etc., to create a thresholded binary image.
*	Apply a perspective transform to rectify binary image ("birds-eye view").
*	Detect lane pixels and fit to find the lane boundary.
*	Determine the curvature of the lane and vehicle position with respect to center.
*	Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## Rubric Points

I will attempt to answer all points asked by the rubric in this section.

*Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf. Here is a template writeup for this project you can use as a guide and a starting point.*

By submitting this readme file, I hope to demonstrate that I have met this objective.

*Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.*

To create the camera matrix and distortion coefficients, the code read in all 20 checkerboard images. The method cv2.findChessboardCorners() was used to check all distorted images for object points and image points to their own lists, with each list containing the object or image points for all 20 images.

The coefficients mtx and dist were there calculated, and each checkerboard image was then undistorted using the two coefficients. Viewing the output images shows that the images are now undistorted.

The below image comes from “advLaneFindingUdacity/camera_cal_undist/calibration17_undist.jpg" and more are available in the same folder.

![undistorted](https://github.com/cswksu/advLaneFindingUdacity/blob/master/camera_cal_undist/calibration17_undist.jpg)
 
*Provide an example of a distortion-corrected image.*

The below photo is available in “advLaneFindingUdacity/output_images/unDist.jpg" and shows the undistorted version of test image 2

![undistorted lane](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/unDist.jpg)

*Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.*
 
The below image shows the thresholded version of test image 2. The method process_frameNW produced this image. It is identical to process_frame(), except process_frame() also warps the perspective. The image is available in “advLaneFindingUdacity/output_images/seg2NW.jpg"

![segmented](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/seg2NW.jpg)

The basic processing is as follows. First, the image is undistorted per the coefficients found in the earlier section. Next, the image is converted to the HSL color space, and the saturation channel is further analyzed.

A variation on the central finite difference method was used to identify potential lane lines. For a given point, a value was calculated based on the difference in saturation between a given pixel and neighboring pixels.

![formula](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/formula.png)

This is similar to the second-order central finite difference, except with opposite sign, with a floor of zero, and the step size is fairly large. A large value of satDiff indicates a local maxima of saturation, or a high likelihood of being a lane pixel. A delta x value of 35 pixels was chosen, as that is wider than any lane marker seen in the test cases. A similar method was used on the periphery of the image as to avoid out of bounds errors, substituting forward and backward differences for central differences. Saturation difference was also scaled so that the maximum value was always 255.

Additionally, a scaled sobel operator was used. The x-direction was used as it is better at picking out vertical lines, such as lane lines. The thresholded image was created by combining all pixels that met a Sobel criterion and a saturation difference criteria with bitwise operators.

Finally, a mask is applied that removes much of the image to reduce noise. The masked image is below and can be found in “advLaneFindingUdacity/output_images/masked.jpg.” Note that this is shown on a full color image, as it makes the mask more obvious. In actuality, it is performed on a black and white image.

![mask](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/masked.jpg)

*Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.*

The warping process happens within process_frame(). After manipulating the provided straight line images, source and destination points were identified that make parallel lines appear parallel from a top down view. These points were then fed into cv2.getPerspectiveTransform() to yield a transformation matrix. This matrix could be used to turn the undistorted images into top-down views. An example of one of the straight line images post-warp is given below, and can be found in “advLaneFindingUdacity/output_images/warp.jpg”

![perspective transform](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/warp.jpg)

This image shows test image 2 after being thresholded and warped. It can be found at “advLaneFindingUdacity/output_images/seg2.jpg”

![segmented perspective transform](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/seg2.jpg)

*Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?*

After the process_frame() method is complete, the warped and thresholded image is given to drawMidpoint(). drawMidpoint() accepts both an image, and 2 sets of quadratic coefficients. If no prior fitted lines are available, then the coefficients are all set to zero.

The method uses a sliding window search to identify the lane marker position, centered around an initial guess. It breaks the image up into 5 horizontal bands, and starting at the bottom, identifies the region with the highest sum of thresholded values in the window. The index is retained for the window with the highest value.

A series of sanity checks is performed before accepting the sliding window result as being legitimate. The distance between the lane lines, the total number of thresholded pixels in the window, and the previous directions of the lane lines are taken into account. If these checks fail, previous values are fallen back on.

After the checks are performed, a point is created in the center of the validated or modified sliding window result. After the whole image has been sorted, a quadratic equation is fitted to the 5 left and 5 right points. If a previous fit is available, the new fit and old fit are compared. Depending on the relative difference between the two fits, the previous and new fits are given different weights and are combined to create a weighted average. Finally, the top of the parabola is cropped off, as it was found that predictions were often less accurate at the top, and due to the perspective warp, they removed very little from the final image.

drawMidpoint() outpus both the lane image and the weighted average set of coefficients that were determined.

The first image shows the parabolas fitted to test image 2. The lane lines are in white, with the grey indicating the space between the lines. The second image shows the lane line image superimposed on the threshold image. They can be found at “advLaneFindingUdacity/output_images/mp2.jpg” and “advLaneFindingUdacity/output_images/overlay.jpg” respectively.

![fitted parabolas](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/mp2.jpg)

![superimposed on thresholded lines](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/overlay.jpg)

*Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.*
 
Radius of curvature and offset are calculated in curvature() and laneOffset() respectively. Both take in coefficients for the left and right lane fit as arguments, as well as the height as which the function is being evaluated. For our purposes, that value is always 720 (the bottom of the image). Both use a scale of 12 feet to 480 pixels, as the lanes in the video appeared to be about 480 pixels wide in the top-down perspective, and a standard highway lane is 12 feet wide.

Radius of curvature is calculated based on the seconds and first derivatives of the fits. It returns the minimum radius of curvature of the two lines at the given y-position. Minimum was returned as it was the “worst-case” scenario.

Offset was calculated by finding where the fitted lines intersect the bottom of the image, and comparing the midpoints of these intersections to the middle of the image. The difference between the two values is then scaled to turn into feet. These values can be returned as text, or overlayed on the final image, as seen in the next section.

![radius and offset](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/radius%20offset%20output.png)

*Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.*

The fitted line image is inverse perspective transformed, and the grey channel is turned into a green channel that is overlaid on the undistorted image of the road (done in the method merge_im()). Radius of curvature and lane offset are superimposed for additional information. This is done in the writeOnImage() method.

The method doAll() manages the whole transformation from start to finish. The example image is given below and can be found in “advLaneFindingUdacity/output_images/doAll.jpg”

![final output](https://github.com/cswksu/advLaneFindingUdacity/blob/master/output_images/doAll.jpg)

*Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)*

The video is in/will be generated in “advLaneFindingUdacity/project_video_out.mp4”

## Discussion
