# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

This writeup gives an overview of the following activities conducted as part of this project:
* A set of routines developed that process an image to find lanes in the image
* Thoughts on how problem was approached, challenges encountered and insights learnt.


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

Most of the enhancements in provided template was at `draw_lines` routine which does the following now:
* Receive optional info from previous pass on same image using left_vertices/right_vertices
* Store all line segments from input image along with slope/intercept/length (called segment-info)
* Define heuristic of what is acceptable range of slopes for left/right lanes
* Classify segment-info list into left/right portions or build up using `left_vertices` / `right_vertices`
* Calculate median slope and intercept from left/right segment-info list
* Use heuristics to filter out segments that deviate more than 
  + 5% of median left slope or 3% of median left intercept
  + 5% of median right slope or 200% of median right intercept
* Build polylines list and calculate valid left and right lane detected lengths by using heuristics
  to filter out line segments that differ by more than 20% of median slope from previous endpoint
  in polylines list
* Extrapolate 'x' to bottom of image using max-y from input image as 'y'
* Verify 'x' distance between bottom-left and bottom-right of lanes is within 70-77% of width of input image.
  Anything outside this range is likely incorrect assumption of lane marker. In such case use greater of
  valid lengths of left/right lanes to determine which to drop as detected
* If both lanes detected, extrapolate to top of image using intersection point of left/right lanes.
  This might potentially need clipping so that lanes don't cross-over in x/y dimension
* Draw the polylines onto input image
* Return detected lane start/end co-ordinates along with their lengths


### 2. Identify potential shortcomings with your current pipeline


There is a fair bit of heuristics used in detection, some of which is reasonable:
  * for given camera/country ratio at bottom of image between lanes to overall picture width
  * slope ranges for left/right lanes especially at bottom of image

Some other heuristics might be more brittle:
  * range of acceptable valid intercepts
  * slope ranges for left/right lanes at top of image

Model is also susceptible for 
  * Strong incorrect noise (cracks, tire-marks etc.) on road that are long
  * Strong signals (shadow, painting) of close by neighboring cars/structures
  * Very curvy road and large change in gradients (uphill/downhill)
  * Performance might be a problem if used for real-time application


### 3. Possible improvements

If this model could be scored against any labeled dataset, more heuristics can be developed or tweaked.
Retaining/consulting information across video frames might lead to better detection outcomes.

### 4. Insights

It was surprising (being a computer vision novice) that HSV lead to better detection especially in yellow-left-solid when lighting is bright.

Memo-ization (remembering info collected so far across call-stack + repeated invocations) is powerful to
try different strategies on image while reducing performance overhead.

Fundamentally I get the feeling that developing a completely hand coded model to deal
with all kinds of roads/situations is hard if not impossible. A better/robust approach might be to use deep learning for building a model with many parameters that is auto learnt.
