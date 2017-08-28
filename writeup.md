# Finding Lane Lines on the Road

[//]: # (Image References)

[img1]: ./demo/thresholding.jpg "Grayscale"
[img2]: ./demo/thresholding2.jpg "Grayscale"
[img3]: ./demo/canny_roi.jpg "Grayscale"
[img4]: ./demo/hough_lines_averaging.jpg "Grayscale"
[img5]: ./demo/hough_lines_averaging2.jpg "Grayscale"
[img6]: ./demo/roi.jpg 
[gif1]: ./demo/hough_lines_and_canny_edges.gif

## Pipeline description

The pipeline is basically the following:
* grayscale conversion
* thresholding
* gaussian blur
* edge detection
* ROI masking
* line detection

#### Thresholding

Fixed-level thresholding is used to zero all pixels below `170`. Defining optimal value for all three videos is not trivial. So additionally adaptive thresholding is used to produce a binary mask (kernel size is `21` pixels). The target here is to improve thresholding on *challenge.mp4*, which requires different values on different image locations. This mask is then applied on fixed-level thresholded image to zero some further pixels.  

In the images below we can see 1. the original , 2. fixed-level thresholded image, 3. adaptive thresholded binary mask, 4. final masked thresholded grayscale image.

![alt text][img1]
Normal road conditions.  

![alt text][img2]
Road with different asphalt intensities.

#### Gaussian blur

A general step for noise reduction. The kernel size of `7` seems large enough to filter noise at resolutions of 960x540 and 1280x720.

#### Edge detection

Gradient-based Canny edge detection is used. Pixel gradients below `canny_low` `80` are rejected, pixel gradients above `canny_hi` `180` are accepted as an edge. Any mid-level pixel gradient is accepted as long as it is connected to an edge. Generally parameter ratio of 1:2 to 1:3 is recommended.  

![alt text][img3]
Thresholded image, result of edge detection, then ROI mask applied. 

#### ROI masking

The following polygon defines the mask (w/h denotes width/height):
```python
[[w*0.05,h],[w*0.45,h*0.6],[w*0.55,h*0.6],[w*0.95,h]]
```

![alt text][img6]
Image sample demonstrating ROI cutout.

#### Line detection

As first step lines with reasonable slopes are sorted to left/right lines according to their signum. Then there is outlier filtering and averaging, all based on lines from previous frames (hold in static containers per side). FInally the result is merged onto the original image.

![alt text][gif1]  
Hough lines drawn on edge detection image.

###### Slope
Lines with `fabs(slope)` between `0.4` and `0.8` are kept. Negative slope belonges to left lane and the others to the right. 

###### Outlier filtering
If relative difference of slope and offset are smaller than `diff_threshold` `0.2`, line is not discarded.  

###### Averaging
Filtered Hough lines on a new frame (represented as slope and offset) will be averaged and stored in buffer. Buffer is limited to a capacity of 5, it is also averaged to avoid jitter from frame-to-frame. If there is no Hough line currently, previous value is obtained based on buffer.  

So new Hough lines are averaged (slope, offset) and the result is then smoothed with lines of previous 4 frames. Finally with slope and offset calculated, lines are drawn from the bottom of image to a fixed proportion of height `0.6`.

![alt text][img4]
Normal road conditions.  

![alt text][img5]
Road with different asphalt intensities.

---
## Potential shortcomings

The pipeline is tested only on three videos which cover just a fraction of what might happen during driving. This is may be fine for demonstrating some basic concepts, but it is hardly applicable in general.

When pipeline is run on the videos, it might happen that left/right lanes cannot be detected on certain frames. In this case the lane from the last frame is used. Potential shortcoming could be that changes in the current scenario can ruin the performance. E.g. different weather/lighting conditions, significant road curves or hillside driving can have the result that many consecutive frames will miss line detection. 

---

## Possible improvements

#### Parameter tuning

Defining optimal parameter values for all three videos is not straight-forward, there is room for improvement here.  Most importantly the parameters of thresholding, edge and line detection could be more thoroughly tested.

#### More general approach
Another potential improvement could be to drop some presumed conditions (e.g. position of horizon) to enable a reworked pipeline perform better in more general scenarios. 
