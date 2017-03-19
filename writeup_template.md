##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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
[image1]: ./examples/Data.png
[image2]: ./examples/hog.png
[image3]: ./examples/search.png
[image4]: ./examples/final.png
[image5]: ./examples/finaldetection.png
[image6]: ./examples/heatmap.png
[image7]: ./examples/pipelineimages.png
[image8]: ./examples/multiple.png
[video1]: ./project_video_output7.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

All code for this project is in "vehicle_detection.ipynb". The code for this is in the function get_hog_features(). I first loaded up the training set for car and non-car images . Visualization for the same is as below

![alt text][image1]

Then put few of the test images  through the hog function to see the output

![alt text][image2]

I then used a function called "extract_features" to accept a array of images and HOGParameters to produce a flattened array of hog features for each image in the array


I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the test classes and displayed them to get a feel for what the `skimage.hog()` output looks like.


####2. Explain how you settled on your final choice of HOG parameters.

I decided  based upon the performance of the SVM classifier produced using them.I considered the accuracy as well as the speed with which classifier is able to make predictions. There is a balance to be struck between accuracy and speed of the classifier, and inorder to do more real time predictions i favored speed and then would try and strenthen the accuracy if i saw an issue.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

In the section titled "Train a Classifier" I trained a linear SVM with the default classifier parameters and using HOG features alone and was able to achieve a test accuracy of 98.17%. 

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used the find_cars() method from the lesson code and modified it to use HOGFeatures over a subsection of the image and then subsampled.
The method performs the classifier prediction on the HOG features for each window region and returns a list of rectangle objects corresponding to the windows that generated a positive ("car") prediction.

Example:-

![alt text][image3]

I explored several configurations of window sizes and positions, with various overlaps in the X and Y directions. The method getRectangles() was evolved after multiple trials to get the right coordinates when testing with the pipeline images

![alt text][image8]

I then used the heatmap function with thresholding to remove false detections and multiple detections.

![alt text][image6]

Final detection looked like this

![alt text][image5]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Below image shows the pipeline images.

![alt text][image7] 

I tried various combinations of colorspace/channel as well as parameters. Increasing the pixels_per_cell parameter increased the speed by quite a bit with little cost to accurary

Detection accurary was increased by changes to window sizing and overlap as described above, and lowering the heatmap threshold to improve accuracy of the detection (higher threshold values tended to underestimate the size of the vehicle).

Although this works well on the pipeline images it does show some false detections on the final video frame...


---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output7.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The code for processing frames of video is contained in the function titled "Process_frame_with_history" and is identical to the code for processing a single image described above, with the exception of storing the detections (returned by find_cars) from the previous 10 frames of video using the prev_rects parameter from a class called Vehicle_Storage. Rather than performing the heatmap/threshold/label steps for the current frame's detections, the detections for the past 10 frames are combined and added to the heatmap and the threshold for the heatmap is set to 1 + len(det.prev_rects)//2 (one more than half the number of rectangle sets contained in the history)


###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had some trouble figuring out the appropriate coordinates for the rectangles to be used and that took quite a lot of trials. I think this pipeline will fail if cars are different from the test set...also we haven't catered for car coming from wrong direction. The pipeline is also showing some false detections specially on the other side of the railing. I think if i tweak the rectangle boundaries a little more this issue would go away...
