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

TODO

#### 5. Curvature and offset calculations
TODO

#### 6. Display lane overlay on single image

TODO

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion
TODO
