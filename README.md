# **Vehicle Detection Project**

The goals/steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier.
* Apply a color transform and append binned color features, as well as histograms of color, to the HOG feature vector. 
* Implement a sliding-window technique and use the trained classifier to search for vehicles in images.
* Run the pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/output_8_0.png
[image2]: ./examples/output_8_1.png
[image3]: ./examples/output_8_2.png
[image4]: ./examples/output_22_1.png
[image5]: ./examples/output_28_0.png
[image6]: ./examples/output_28_1.png
[image7]: ./examples/output_28_2.png
[image8]: ./examples/output_28_3.png
[image9]: ./examples/output_28_4.png
[image10]: ./examples/output_28_5.png
[image11]: ./examples/1.png
[image12]: ./examples/10.png
[image13]: ./examples/image24.png
[image14]: ./examples/image25.png
[video1]: ./project_video_processed.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the second code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

Vehicle

![alt text][image11]  ![alt text][image12]

Non-Vehicle

![alt text][image13]  ![alt text][image14]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YUV` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image1]

![alt text][image2]

![alt text][image3]


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and color spaces. Visualizing the hog features gave me a good idea about the information in each channel. I trained the classifier with hog features derived from several different color spaces and comparing them I found YUV to have the best accuracy. 

Even in YUV, I observed that U and V channels didn't actually have a lot of useful information so I used only the Y channel from YUV. The pixels per cell parameter is set to 8 and cells per block is set to 2. I left these parameters the same as used in audacity exercise since changing any of them either reduced the accuracy of the model or didn't have any significant effect.    

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Initially, I trained a LinearSVM classifier which was quick to train and predict but only gave an accuracy of 97%. Now since each frame has 304 windows a 97% accuracy would mean about 9 wrong predictions per frame. I could use thresholding and average over past frames to get rid of false positives.

I tried `Sklearn.svm.SVC` with `rbf` kernel and default `C`. This classifier had an accuracy of 99.5 % which although is much better than LinearSVM but takes 3 times the time to train and is about 100 times slower in prediction. If I was to work on a live video feed I'd use LinearSVM for its speed but for detecting vehicles, in a video file, I prefer accuracy over speed. So, I stuck with SVC.       

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

For sliding window, I chose to ignore the top half of the frames which is mostly sky and tree. I also removed the left half from the frame because I was interested in detecting the cars on the same road which is on the right.

I decided to use scales from 1 - 3 because this range is able to capture the farthest and closest vehicles in the video. 

Overlap is set to 2 cells. This strikes the perfect balance between getting too many windows (which slows down the processing) and getting to so few that it's difficult to filter out noise. 

Here is a visualization with all the windows displayed.


![alt text][image4]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on three scales (1,2,3) using Y channel HOG features ( from YUV color space ) plus spatially binned color and histograms of color in the feature vector, which provided a nice result.

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_processed.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections, I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected provided the blob is wider than 20px to remove any noise that might have slipped through the threshold.  

Here's a visualization of the initial detection, Heatmap and the final output of the pipeline when running on test images.


![alt text][image5]

![alt text][image6]

![alt text][image7]

![alt text][image8]

![alt text][image9]

![alt text][image10]

---

### Discussion

#### 1. Briefly discuss any problems/issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* I  have used SVC with 'rbf' kernel for classifying each window which is very slow and would not be suitable for a real-time system. The pipeline only processes every 8th frame to make it faster but that also makes the system less responsive to sudden changes in vehicle positions.

* One way to improve the project would be to use a dynamic thresholding heatmap. I have used a threshold of 3 which is good for most frames but a dynamic threshold which changes according to the number of windows detected would be a big improvement.

* I chose to ignore the left part of the frame which again is very specific to the project video and would need to be removed if I want to use the pipeline for other videos.
