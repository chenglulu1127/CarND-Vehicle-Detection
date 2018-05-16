## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/image_plot.png
[image2]: ./output_images/hog_features_vis.png
[image3]: ./output_images/color_hist_features_vis.png
[image4]: ./output_images/bin_spatial_features_vis.png
[image5]: ./output_images/pre_heatmap.png
[image6]: ./output_images/apply_heatmap.png
[image7]: ./output_images/label_test_img.png
[video1]: ./project_video_out.mp4
[video2]: ./test_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The function of extracting HOG features is defined in cell #4 of `Code.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of some random sampled `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  

Here is an example using the `LUV` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

And here's visualization of Color Histogram features and Bin Sptial features:

![alt text][image3]

![alt text][image4]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters to extract HOG parameters. Whenever I use 'ALL' for hog_channel, it keeps giving "Memory Error". I spent a lot of time trying to make it work through, but it seems that the data is too big for the scaler to work (in memory calculation). Thus I decided to get the lowest accuracy by tuning other parameters. The parameters I tuned in the end is: color_space = 'LUV', orient = 8, pix_per_cell = 8, cell_per_block = 2, hog_channel = 0. 
For color hist features and bin spatial features, the parameter I set is: spatial_size = (16, 16), hist_bins = 32.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using hog featuers, bin spatial features and color histogram features and achieve accuracy 98%. The features are normalized by StandardScaler() to ensure that the classifier is not dominated by a subset of features. The functions are defined in "Train Classifier" section of `Code.ipynb`. 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I implemented 2 multi-scale sliding window and use 2 as overlap threshold. The function is defined in "Find Car Pipeline" section of `Code.ipynb`. The further the vehicles are from me, the smaller scale it will have. 

Fixed sliding window size (scale = 2):
![alt text][image5]

Multi-scale sliding window with threshold = 2:

![alt text][image6]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I searched on two scales using LUV channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Then I created heatmap with positive detections and then threshold the map to identify vehicle positions. Here are some example images:

![alt text][image7]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  
I also implemented using the rectangles from last several video frames to help improve the performance of the model.
The "process_frame_for_video" function is defined in "Test on Video" section of `Code.ipynb`. 


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Problems (possible solutions) I faced with:
1. Memory error issue when using StandardScaler();
2. When using detections from previous frames to reduce misclassifications, there's another problem introduced: vehicles sometimes significantly change positions from one frame to the next (we can try considering the speed and predict the locations of vehicles)
3. The pipeline might fail in case of bad light conditions (improve classification algorithm, data augmentation, hard negative mining)
