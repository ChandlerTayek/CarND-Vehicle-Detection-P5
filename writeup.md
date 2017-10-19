**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/HOG_example.png
[image3]: ./output_images/sliding_windows.png
[image4]: ./output_images/pipeline_output_1.png
[image5]:./output_images/pipeline_output_2.png
[image6]: ./output_images/bboxes_and_heat.png
[image7]: ./output_images/output_bboxes.png
[video1]: ./test.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
Note: All of the code for the project resigns in the file `pipeline.ipyn`

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first three code cells of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Note that the image shown
below is not the pixels_per_cell used in the final model. I merely used a lower pixel per cell value
in order to increase the number of cells shown in order to get a since of how the computer visualizes
the images.

All of the feature are extracted in the cell directly underneath the title `Training`.
The features are saved under the file (`features.pkl`).

Note: you do not need to run the cell underneath training as long as you have both the
`features.pk` and `clf.pk` files.

Here is an example using the `Gray` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(2, 2)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and came to the optimal result of:
* `Color Space`: `YCrCb`
* `orient`: 9
* `pix_per_cell`: (8,8)
* `cell_per_block`: (2,2)
* `hog_channel`: `ALL`
* `spatial_size`: (32, 32)
* `hist_bins`: 32

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

In the same cell as the feature extraction, I also train the classifier. The classifier
I used is the linear support vector machine. I saved the trained classifier under the
file `clf.pk` so the user doesn't need to re-train and can just load in the file.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

All of the code for the sliding window search is located in the cell labeled `Sliding Window`,
`Box Drawing`, and `Sliding Window Example`. I decided to cut off the top and left hand side
of the image as searching the sky and other side of the road (trees) are not necessary.
I combined all three scales in order to get more positive detections as one scale wasn't
giving enough hits.
I decided to try out different scales and overlaps until I arrived at an optimal result:

* `x_start_stop`: [450, 1280]
* `y_start_stop`: [400, 656]
* `xy_window`: (128, 128)
* `xy_overlap`: (0.5, 0.5)
* `scales`: (1.0, 1.5, 2.0)


![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

In the cell labeled Example pipeline, I take in an image and push the image through all of the steps
it would take for just the feature extraction using all of the same final hog parameters. The image
below shows this:

![alt text][image4]
---

The cell labeled `Pipeline Test on test_images` shows how the entire pipeline works on a couple
test images. Shown below is the output of the images that were put through the pipeline without
any false positives removed:

![alt text][image5]


### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in  each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. The cells of code in the file are located under the labels: `Heatmap Example` and `Heatmap Threshold and Draw`.   

Here's an example result showing the heatmap from a series of frames of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image6]


### Here are the six frames that have the `scipy.ndimage.measurements.label()` applied and false positives removed:
![alt text][image7]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The hardest part of the project and something that is still broken is smoothing out all of the
jittering and drawing the bounding boxes on the frames where the bounding boxes disappear.
The logic of when I need to draw a car at its average location when it doesn't show up in the
frame originally needs to be changed (any tips would be great).
