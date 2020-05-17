## Writeup
---
**Highway Driving**


[//]: # (Image References)

[image1]: ./write_up_images/1_chessboard_undistorted.jpg "Undistorted"
[image2]: ./write_up_images/2_straight_lines1_undistorted.jpg "Road Transformed"
[image3]: ./write_up_images/binary_exploration.jpg "Binary exploration"
[image4]: ./write_up_images/3_binary.jpg "Binary Final"
[image5]: ./write_up_images/source_locations.jpg "Source locations"
[image6]: ./write_up_images/4_perspective.jpg "Perspective Transform"
[image7]: ./write_up_images/5_final.jpg "Final output"
[video1]: ./output_images/test_output.mp4 "Video"

---
References:
Udacity starter code provided in the tutorials was used throughout and the Q and A https://www.youtube.com/watch?v=vWY8YUayf9Q&feature=youtu.be was used to help fine tune the approach. 


### Camera Calibration

Found in section 'SECTION 1 - Camera Calibration' of the IPython notebook: Advanced_Lane_Finding

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `object_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `image_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `object_points` and `image_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()`

![alt text][image1]


### Pipeline (single images)

#### 1.Distortion-corrected image.

Using the distortion coefficients and matrixes that were calculated with the chessboard the following could be used to remove any distortion from the road. 

```python 
undistorted = cv2.undistort(image, mtx, dist, None, mtx)
```

Here is an example of a distortion corrected image of the road. There isn't a big difference between the original image and the undistorted image. This isn't surprising as the camera is a good camera and is not wide angled or fish eye. Although it isn't easy to see the difference, the sign on the right hand side of the picture is closer to the edge. This helps to give a feel for how the edges of the picture are being slightly changed by the distortion. 

![alt text][image2]

#### 2. Thresholded binary image

Found in section 'SECTION 2 - Thresholded binary image (color transforms and gradients)' of the IPython notebook: Advanced_Lane_Finding

I explored a number of different ways to generate a binary image. These included x gradient thresholds, y gradient threshold, magnitude gradient thresholds, gradient direction thresholds, R color threshold, V color threshold and S color threshold. I'll explain each one more detail below. 

The starter code for these sections was taken from the earlier sections of the Udacity tutorial.

##### x and y Gradient Thresholds
This approach looks at applying the sobel kernel over either the x or y direction and applying a threshold. In finding locations of large gradient these will help to identify the change from road to line.

This is how the sobel kernel is applied for the x direction.

```python
sobel = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize = sobel_kernel)
```

##### Magnitude and Direction Thresholds
These approaches look at the overall gradient by combining the x and y gradients to find the overall gradient magnitude and direction. Through experimentation I found the direction approach to be very noisy and neither of these were used in the final implementation.  

##### R,V,S Thresholds
Converting the image between RGB, HLS and HSV allowed me to target different channels. I focused on Red, Saturation and Value. Allowing different thresholds to be applied. 

I used a combination of color and gradient thresholds to generate a binary image 

Here is an example of what the different methods contributed and how I viewed each one during experiementation to find settings. 

![alt text][image3]

Having tried different options and taken setting ideas from https://www.youtube.com/watch?v=vWY8YUayf9Q&feature=youtu.be, the following shows the final output of the binary stage. 

![alt text][image4]

#### 3. Perspective transform

Found in section '# SECTION 3 - Perspective Transform' of the IPython notebook: Advanced_Lane_Finding

For the perspective transform I decided to hard code in the location of the source points and then check these against one of the test images. I then assumed that this would be acceptable for most of the occurances. After having attempted the harder video challenges I chose to limit the distance that the car looks ahead to help manage harder scenarios where there are sharp corners. 

```python 
corner_locations1x = 520.0
corner_locations2x = 770
corner_locations3x = 1050
corner_locations4x = 250

corner_locations1y = 500.0
corner_locations2y = 500
corner_locations3y = 690
corner_locations4y = 690
```

The following image shows where these points would fall on the road. I then projected this onto a standard rectangle and having tried different offsets stuck with 200.

![alt text][image5]


Example of perspective transform on the binary image. 

![alt text][image6]

#### 4. Lane Line Detection

Found in section 'SECTION 4 - Sliding Window to find Lanes.' of the IPython notebook: Advanced_Lane_Finding.

Here I used the approach of applying a histogram to pixel values in y axis. This allowed me to find where the peak of 'white' pixels in the binary output where and therefore where the line was most likely to be. 

The image was split in two, to find the peak for the left and peak for the right hand lane. 

Breaking the image up vertically and tracing up the image with windows allowed the model to follow where the left and right lines went. 

A polynomial was then fit on top of these locations. 



#### 5. Radius of Curvature

This was calculatated within 'SECTION 4 - Sliding Window to find Lanes' and the function 'measure_curvature_real()' was used to predict the actual curvature in real space.

(I calculated the radius of curvature in both pixel and real space using the radius of curvature calculation)

```python
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
```

The location of the vehicle is also calculated. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

These lines and region between was then plotted back on to the original image. To do this the inverse matrix was required to perform the transform. 

![alt text][image7]

---

### Pipeline (video)

#### 1. Project video result

Here's a [link to my video result][video1]

---

### Discussion

The lane finder now performs well in the project video where there is a slight bend. 

All stages of removing camera distort, converting to a binary image of the key features, perspective transform, and line finding were all implemented. 

In the challenge and harder challenge video the process does not perform so well. 

There are a few limitations to my line finding implementation. I don't retain the past locations of the lines in previous images and use that to smooth out any challenges sections.By using the polynomial from the previous image you could have a better chance of finding the line.

I also didn't implement a method to deal with the windows going off the edge of the image in the case of very tight corners which also causes issues in the harder challenge. These two areas would be the next areas in address in improving the performace. 






