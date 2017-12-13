
## Writeup 

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

[image1]: ./image1.png  "chess board un-distortion"
[image2]: ./image2.png  "road camera un-distortion"
[image3]: ./image3.png  "line detection"
[image4]: ./image4.png  "Warp the image"
[image5]: ./image5.png  "Fit Visual"
[image6]: ./image6.png  "Output"
[video1]: ./project_video_lanes.mp4 "Video"


### STEP1. Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook in the github main folder, under the name of "main.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### STEP2. Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I did three things.

First, I set up the sobel x to detect the gradient. Second, I tuned the saturation threshold. Third, I used the "OR" function to combine the two so that I got both lines far away (saturation is more reliable) and nearby (sobel x performs better).

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`. The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src=np.float32([[275,668],[1024,668],[577,462],[703,462]])
dst = np.float32([[200,700],[1280-200,700],[200,20],[1280-200,20]])
```

Below is the source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 275, 668      | 200, 700      | 
| 1024, 668     | 1080, 700     |
| 577, 462      | 200, 20       |
| 703, 462      | 1080, 20      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The lane-line pixels were identified within the bird-view image, or the so-called warped image.

Then I identified the lane-line pixles using a moving window sliding to determine the max signal for left lane and right lane. To avoid  minpix pixels, I recentered next window on their mean position if there is an unstability.

```python
        if len(good_left_inds) > minpix:
            leftx_current = np.int(np.mean(nonzerox[good_left_inds]))
        if len(good_right_inds) > minpix:        
            rightx_current = np.int(np.mean(nonzerox[good_right_inds]))

```

![alt text][image5]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

For both side lines, it reports the curvature to be: 595.148713873 m and 575.111358463 m, respectively. I need to do two things then. 

First, I solve for the curvature by applying the fitting method as below.
```python
   left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
 
```
Second, I try to extract the car position relative to lane, by compare the position of camera to position of the lane center.
```python
    camera_position = image.shape[0]/2*xm_per_pix
    left = leftx*xm_per_pix
    right = rightx*xm_per_pix
    lane_center = (right[700] + left[700])/2
    center_offset_pixels = (camera_position - lane_center)/2
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 6 within the jupyter notebook "Video_generator". It is composed of two steps, un-warp the identified area, and projected onto top of the original image.
```python
    newwarp = cv2.warpPerspective(color_warp, Minv, (image.shape[1], image.shape[0])) 
    image_5 = cv2.addWeighted(image, 1, newwarp, 0.3, 0)
 
```
Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I tested a few pictures and they worked pretty well. I don't have the code to run the video.

I don't know how to do it in video form, but my work should prove that I am competent in the computer vision ideas.

Please see video clip of the work. 

To make it work, I re-organized the program into Video_generator.ipynb. The algorithm is not changed. But the funciton is packaged into the function:

```python
def MyGame(img0):
...
return image_5
```

Then I use the video package to do the frames one by one, as below.

```python
clip = input_vidoe.fl_image(MyGame) 
```

![alt text][video1]

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

First, I realized the combination of sobel x and saturation works pretty well. Below is the code.
```python
    combined[((sxbinary == 1) | (s_binary == 1))] = 1
```

Second, if I were to pursuing this project further, I'd spend more time in dealing with shadows, which is challenging to real drivers as well. The shadow with low saturation should be able to be distinguished, assuming we deal with sobel smartly. I'd like to run different combinations in the first point.

