**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the Jupyter notebook named `"vehicle_detection.ipynb"` under the `"Extracting HOG features"` heading.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes and the extracted hog features:
[image1]: ./output_images/hogexample.png

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

The above image is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and tried to find a set that would achieve 99% accuracy or better on the test data..
With every parameter, I tried several options high and low of the original options and stuck with the one that provided a highest accuracy.

I decided to use all channels to extract the HOG features because I felt it would help the features better represent the information in image.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

This code is found under the `" Extracting HOG features and Training"` heading.
I trained a linear SVM using hog features, a color histogram, and spatial features.  

I stacked all those features in a single feature vector on line 40.
I removed the mean and scaled to unit variance using the StandardScaler on lines 42-44.

The data was randomized and split up into a test and training set on lines 50-52

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The code for this implementation in the find_cars method in vehicle.py.
I constrained the area of search to a horizontal section including the road.  I did that by limiting the y values when generating the windows.
The algorithm for finding the next window to classify is as follows.
1. Step across the image horizontally at a pixel interval defined by pix_per_cell and cells_per_step.
2. Extract a patch of the image to send to the classifier for prediction.
3. When reaching the horizontal edge of the image, step across the image vertically using the same increment.
4. Once you reach the vertical edge of the search space defined by ystop, stop.


I decided on a scale of 1.25 for the sliding window search.
I found that lower numbers provided a higher resolution, 
but higher numbers helped limit the size and compute time.
I settled on 1.25 by trying a variety of values and choosing the one that gave the least number of misclassifications on the test image.

The cells_per_step parameter determines the amount of overlap that each window has.  I wanted to have a lot of overlap to increase the number of positive predictions on the cars, so I set this value to 1.
A higher value would have meant less overlap.

#####  All Sliding Windows
[fullslide]: ./output_images/full_sliding_windows.png

![alt text][fullslide]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

To optimize the performace of the classifier, I kept the sliding windows large enough to extract images that it could correctly predict most of the time.  I found a good balance through trial and error.  I found that the classifier could make good predictions even if it only saw a portion of the vehicle at a time.

Ultimately I searched on a scale of 1.25 using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

[positive_detections]: ./output_images/positive_detections.png

![alt text][positive_detections]

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

[video1]: ./result_video.mp4
Here's a [link to my video result](./result_video.mp4)

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

The pipeline kept a recent history of the heatmaps from the last 5 frames. I summed the binary heatmaps and then thresholded that summed map to negate some false positives.  If an area of pixels didn't contain a positive prediction for any of the frames in recent history, that area would be filtered out in heatmap of the final frame.

Here's an example result showing the heatmap from a series of frames of video.

### Here are five frames and their corresponding heatmaps:
[image5]: ./output_images/heat_maps.png

![alt text][image5]

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had a lot of trouble with picking the right parameters for the sliding window implementation.
A more sophisticated implemenation would really improve the performance of the pipeline.  Tracking a vehicle and remember features of a vehicle (color, relative location in the image, size, etc) once identified would help generate much more accurate predictions in subsequent frames.

I also had a problem with false positives in on the road.
I could fix this by:

1. Getting more training data and improving the accuracy of the classifier.
2. Using a more intelligent search method to ignore areas can't have cars.
3. Adding redundant checks, like seeing if the a different classifier detected a car in the same place.
