# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

This writeup gives an overview of the following activities conducted as part of this project:
* A set of routines developed that process an image to find lanes in the image
* Thoughts on how problem was approached, challenges encountered and insights learnt.


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Image processing pipeline
The pipeline (with a starting point at `process_test_image`) consists of following steps:

* Convert input image from RGB (red-green-blue) color scheme to Grayscale
* Feed the grayscale image to `process_grayscale_image` and receive back
    + a weighted image with left and right lanes marked (if detected)
    + lengths of left and right lanes detected
    + start/end co-ordinates of left and right lanes
* If the output of `process_grayscale_image` suggests both lanes were detected, return the weighted image.
  Otherwise convert input image from RGB color scheme to HSV (hue-saturation-value) and then to Grayscale
  and invoke `process_grayscale_image` once again this time passing start/end co-ordinates of any detected
  lanes and weighted image received from previous step
* Return the weighted image as output of the pipeline

So the call stack of pipeline is:


````
    1. process_image
       1.1 process_test_image
           1.1.1 process_grayscale_image
                 1.1.1.1 canny()
                 1.1.1.2 region_of_interest()
                 1.1.1.3 hough_lines()
                     1.1.1.3.1 cv2.HoughLinesP()
                     1.1.1.3.2 draw_lines()
                 1.1.1.4 weighted_lines()
           1.1.2 if 1.1.1 returns with either left/right lane undetected, call process_grayscale_image using HSV
````

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
