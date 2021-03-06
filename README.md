# CarND-Vehicle-Detection-P5

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
---

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I started by reading in all the `vehicle` and `non-vehicle` images.  The code for this step is contained in the first code cell of the IPython notebook.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

Then I wrote functions for getting color, spatial, hog features from the lessons. The code for this step is contained in the second code cell of the IPython notebook.

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed 3 random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like. The code for this step is contained in the third code cell of the IPython notebook.

Here is an example using the `LUV` color space and HOG parameters of `orientations=9`, `pixels_per_cell=16` and `cells_per_block=2`:

![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of HOG parameters and stopped on the following:

    colorspace = 'LUV'
    orient = 9
    pix_per_cell = 16
    cell_per_block = 2
    hog_channel = "ALL"
    
These were the parameters which led to the best classification accuracy on the training data (99.89%). I used `pix_per_cell = 16` because this value is tradeoff between feature vector length (1356) and accuracy. I used the HOG features from all 3 color channels in order to get the most consistent results in the project video.
    
#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using LinearSVC() from sklearn.svm. I used combined color and hog features for training SVM model. Before training the data, the data was normalized using StandardScaler() from sklearn.preprocessing. Then these normalized data were splitted into train and test sets with proportion 80% for train and 20 % for test sets. The code for this step is contained in the fourth and fifth code cells of the IPython notebook.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used HOG Sub-sampling windows search. I used 3 sizes of windows to detect vehicles on different locations of the road. 
* small windows (51x51) are located from 400 to 480 on y axis 
* medium windows (96x96) are located from 400 to 650 on y axis 
* large windows (128x128) are located from 400 to 700 on y axis. 
The search started from 400 on x axis to eliminate detection of vehicles on left lane of the road. The default overlap of 2 cells by step is changed to 1 to increase number search windows. The code for this step is contained in the 10th and 11th code cells of the IPython notebook.

Below results are shown for test images.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

When a car is detected, multiple boxes are drawn on the car, so I used a heatmap to combine boxes into a single box. To remove false positive I used threshold. Then I used label() from scipy.ndimage.measurements to draw box around detected cars. The code for this step is contained in the 12th and 13th code cells of the IPython notebook.

Below results are shown to demonstrate how pipeline is working.

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. 

Here are ten frames and their corresponding heatmaps:

![alt text][image5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* The pipeline is not a real-time. It takes about 1 fps with Lane line detection. To decrease time for frame processing I think to wrap C++ function for hog features extraction. Another approaches as YOLO or SSD may be used to improve fps.

* The algorithm may fail in different light conditions, in detection of pedastrains, motorbikes and other vehicles having different forms. To make it more robust it needs train classifier on datasets including pedastrains, motobykes and other vehicles.

