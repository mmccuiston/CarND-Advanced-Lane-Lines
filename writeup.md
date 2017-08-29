[//]: # (Image References)

[image1]: ./writeup/distorted-chess.png "Distorted"
[image2]: ./writeup/undistorted-chess.png "Undistorted"
[image3]: ./writeup/undistorted-view.png "Undistorted Example"
[image4]: ./writeup/perspective.png "Warped Perspective"
[image5]: ./writeup/threshold.png "Thresholded"
[image6]: ./writeup/overlay.png "Overlay"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration
The code can be found in the first five cells of the advanced-lane.ipynb file.  

First I read in the supplied calibration images using cv2.imread.  Then I convert to grayscale and use the cv2.findChessboardCorners function to find the locations of the black/white squares in the calibration images.  I then use the cv2.drawChessboardCorners function to draw lines between the detected corners and plot them to visually verify that the corners are detected correctly.  I also save a list of "object points" which are the expected location of the chessboard corners in the real world.  These will be used later.

Finally, I use the a object points above and the locations of the detected chessboard corners to calculate a calibration matrix and distortion matrix using the cv2.calibrateCamera function.

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. First I undistort the image
Here is an example of a captured image where the camera distortion is corrected.  This image was undistorted by using the cv2.undistort function along with the previously computed distortion matrix and calibration matrix.
![alt text][image3]


#### 2. Then I warp the image using a perspective transform.
I first calculate the M and Minv matrices used in the perspective transform.  These matrices will be reused for all image transformations going forward.  The code for this is in cells 13-15 in the advanced-lane.ipynb.  I used the same image from the previous section where the lane lines appear perfectly straight.  I then manually selected 4 points in the image (2 near the vanishing point of the lanes, and two where the lanes intersect the bottom of the image).  I then selected 4 other points to tranform to (same points at the bottom of the image, and two points intersecting the top of the image directly above the other  two points).  I then used the cv2 getPerspectiveTransform to calculate the perspective transform matrix M.  I then used the same points, but reversed the parameter order to cv2.getPerspectiveTransform to calculate the inverse matrix of M, Minv.  To verify that the M was correct I applied the perspective transform using cv2.warpPerspective to the above image.  You can see that the resulting image below has parallel lane lines.

![alt text][image4]

The pipeline utilizes the function def perspective_transform(img, M) to perform this transform going forward.


#### 3. Thresholding to find lane lines.

I decided to convert the images to the HLS color space.  Once converted I applied two different thresholds to find white and yellow lane lines, then combine the separate thresholded images to show all lane lines.  For white lines I found that using a threshold on Luminosity above 200 worked well.  For yellow, I found that a combination of saturation and luminosity both being above 100 worked well.

See the above image after thresholding.
![alt text][image5]


#### 4. Fit polynomial to detected lanes
Next I attempt to fit a second degree polynomial to the pixels that are set after thresholding above.  The polynomial fitting algorithms takes the following steps.  
1. Split the image into left and right halves.
2. For each half compute a histogram of all the x values for pixels that are 'on'.
3. Set the 'start' pixel for both halves to be the peak of the respective histograms from previous step.
4. For both halves create a window that is 200 pixels wide, 1/9th the image in height, and centered on the 'start' pixel from above.
5. Save the x,y coords of 'on' pixels within the window to be used for the fitting algorithm later.
6. If there are at least 50 pixels enabled in the window, then recenter the x value for the window to be the mean value x of the pixels in the window.
7. Repeat steps 4-7 until 9 sliding window computations have been completed, thus covering the entire image.
8. Use the coords saved from step 5 to fit a second degree polynomial to the coordinates.  We use numpy.polyfit for this.


#### 5. Curvature and offset calculations
We then calculate the curvature in meters of the lane.  We do this by converting all of the pixels saved from the previous step from pixel coordinates to real-world coordinates.  We use the following conversion factors.

ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension

Once in real-world coordinates we fit a second degree polynomial to them.  Once we have the polynomial fit, we use the R-curve function.

(((1 + (dx/dy)^2) ^ 3/2) / (d2x/dy2)

This will give us the radius in meters for both the left and right lanes separately.  We then average them to get one value for the lane radius.

The offset is calculated by calculating the intersection of both the right and left lane polynomials with the bottom of the image.  Then I take the middle of these two points and assume this is the center of the lane.  I use the center of the image as the position of the vehicle.  Calculating the offset is done by taking the difference of these two values.

#### 6. Display lane overlay on single image
I fit a polygon to the left and right points in the top-down perspective image.  Then I use open cv2.fillPoly to draw a polygon between the lines.  I overlay this polygon onto the original image using the cv2.addWeighted function.  Then I warp the image back to it's original perspective by using the Minv matrix calculated earlier and the cv2.warpPerspective function.
TODO

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion
I think my detection algorithm does a pretty good job on the project video.  There is some jitter around the furthest parts of the image.  I believe this could be made better by improving my thresholding algorithm and perhaps smoothing detections between images.
