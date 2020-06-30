## Advanced Lane Finding

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

[image1]: ./output_images/binary.jpg "Binary Example"
[image2]: ./output_images/transform.png "Transform"
[image3]: ./output_images/lanepixel.png "Warp Example"
[image4]: ./output_images/pipeline.png "Output"
[video1]: ./output_videos/project_video.mp4 "Video"

### Camera Calibration

The images for camera calibration are stored in the folder called `camera_cal`. I start by finding chessboard corners and save 'objpoints' and 'imgpoints' at lists. I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function    

---

### Pipeline (single images)

#### Create a thresholded binary image

From a distortion-corrected image, I used a combination of color and gradient thresholds to generate a binary image. In order to get thresholed binary image, I converted RGB image into HLS color space and applied Sobel function and threshold. Here's an example of my output for this step.

![alt text][image1]

#### Apply a perspective transform

Next, I applied a perspective transform to rectify binary image. This step takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 50, img_size[1] / 2 + 90],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 50), img_size[1] / 2 + 90]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 450      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 690, 450      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image2]

#### Detect lane pixels and fit to find the lane boundary

By using perspective transformed image, I identified lane-line pixels and fit their position with a polynomial. I took a histogram of the bottom half of the image to detect left and right lane-line pixels. The peak of the left and right halves of the histogram is the starting point for the left and right lines. I used 'find_lane_pixels' function to identify lane-line's movement by dividing a image height into 10 sliding windows. After extracting left and right line pixel positions, I calculated a second order polynomial to both lane's fit using 'np.polyfit'. Final step is plotting the left and right polynomials on the lane lines.

![alt text][image3]

#### Determine the curvatue of the lane and vehicle position with respect to center

I used the function 'measure_curvature_real' with constant which calculating meters per pixel in x and y dimension to change the unit from pixel to real world (meter). In addition, I utilized image's horizontal half size with the difference between left and right lane-line pixels to determine the vechicle position with respect to center. If the difference is larger than zero, the vehicle is on the right from the center and the difference is less than zero when the vehicle is on the left.

![alt text][image4]

#### Example image of the result plotted back down onto the road

I implemented all the step in the class 'Line' with function 'process_image' in my code. I used this class in order to save previous image frame's fit information in order to soften the polynomials between the lane lines as well as lane detection. I initialized 'Line' class with size one for a single image. Here is an example of my result on a test image:

![alt text][image5]

---

### Pipeline (Video)

![alt text][video1]

### Discussion

Even though I used previous frames's lane-line fit information to smooth video frame, there are serveral points to improve the result. First, when car is driving on the road with dark shadow or faint lane-line marker, my implementation does not show perfect lane boundary. I might need to add up sanity check logic when pipeline doesn't detect lane-line properly.