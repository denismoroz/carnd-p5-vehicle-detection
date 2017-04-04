
**Vehicle Detection Project**
---
The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[car]: ./test_images/5959.png
[not_car]: ./test_images/image3710.png
[hog_car]: ./output_images/hog_features.jpg
[hog_not_car]: ./output_images/hog_features_not_car.jpg
[normalized_features_1]: ./output_images/normalized_features_1.jpg
[normalized_features_2]: ./output_images/normalized_features_2.jpg
[normalized_features_3]: ./output_images/normalized_features_3.jpg
[normalized_features_4]: ./output_images/normalized_features_4.jpg
[found_car]: ./output_images/found_cars.jpg
[found_car_all]: ./output_images/found_cars_all_tests.jpg
[video1]: ./project_video_output.mp4
[region_of_interest]: ./output_images/found_cars_region_of_interest.jpg


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the third code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][car]
![alt text][not_car]

Using `skimage.hog()` I extracted different features set for car and not car image.

![alt text][hog_car]
![alt text][hog_not_car]

Then I extracted a binned color features and compute color histogram features for images and normalize them.

Here is an example using the `HLS` color space and HOG parameters of `orientations=16`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:
![alt text][normalized_features_1]
![alt text][normalized_features_2]
![alt text][normalized_features_3]
![alt text][normalized_features_4]

Resulting feature vector length is 21750.

####2. Explain how you settled on your final choice of HOG parameters.

To get best results I first build working pipeline that allows me to detect vehicles and the then I modified different parameters.
I tried less orientations for hog but it results in more false positives. 16 orientations was played good job from my point of view.

From different color spaces I selected HLS because it helped me to detect vehicle on sunny parts of the video. There were issues when algorithm failed to detect white car on places when it drives from shadow part of the road to sunny and back. But at the same time black car was detected perfectly. Switching to HLS helped to detect white care more accurate.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Code for this step can be found in third cell of the IPython notebook.
 
There are different numbers of images in provided training dataset for car and non_cars. So on the first step I sliced max sized set to fit minimum one result: cars samples:  8792 and not cars samples:  8792.
 
Input for linear SVM was set 17584 samples. 20% of samples was taken for testing data using `sklearn` `train_test_split` funciton.
On my notebook it took 42.26 seconds to train SVC with resulting test accuracy 0.9923.
 
###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used hog sub-sampling window search to optimize sliding window search. Hog features was calculated once for a frame, and then subsampled to get rest of overlaying windows.
Code for this step can be found at 7th cell of my notebook `find_car` function.

![alt text][found_car]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on 1.5 scales using HLC 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][found_car_all]

Optimization: scale to 1.5. take on part of frame for vehicle detection: ystart = 380 ystop = 656.
That helped to recude number of windows that should be checked.

---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Here's a [link to my video result](./project_video_output.mp4) and [Youtube version](https://youtu.be/T7QveuB8CT4)

Detected car shown with blue box. Red box - region of interest. Green boxes - results of detection by linear SVM.

####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Algorithm working it two states:
 - full windows search mode.
 - region of interest search mode.

Start mode is full windows search mode. It tries to detect vehicles using classifier on the whole image.
Using heat map false positives are detected and cleaned up. Resulting labels set are used on the next frame to predict area where car should be searched for.
Algorithm do a full search each 25 frames or when it is failed to detect any vehicle in region of interest.

On the next set of pictures seen these steps. Blue boxes are detected vehicles and red box is area of interest for the next frame.

![alt_text][region_of_interest]

Also a different treshold for heatmap during full search and region of interest search.
Full search heatmap treshold = 3 and region of interest search treshold = 2.

Using this algorithm allows to speedup processing for project_video_output.mp4 from 8mins to 2m 55sec.
  
---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most difficult thing was to find right combination for colorspace, hog orientation and number of histogram bins. It took me a lot of time to find them.
This implementation will fail on bad weather conditions, on mountain roads. And the main issue is that is VERY slow. For 50 seconds of video almost 3 minutes are needed.
I expect that the next step is to use C++ and more advanced technics like kalman filters to speed up pipeline.