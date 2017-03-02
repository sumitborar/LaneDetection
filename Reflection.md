
**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

### Reflection

###1. Pipeline description

My pipeline consisted of 8 steps as shown below.

![alt text][image1]: ./LaneDetectionPipleline.jpg

####Step 1: Input image
Each video frame is passed as input image to the pipeline. For the challenge video, I resize input image to (540, 960) size. This is not necessary and can be avoided by passing size parameters to subsequent steps.

####Step 2: Color thresholding
I use RGB color thresholds to return lane image. I recognize white color lanes in the image using RGB palate and area of interest mask to generate grayscale masked image in which only the lanes are shown.

####Step 3: Canny algorithm
I use canny algorithm to high contrast areas in the image. The region of interest further cleans up the resultant image.  Resultant image is grayscale and only the lanes are shown. As part of preprocessing, following steps are used
1. Convert image to HSV space for better recognition of lanes
2. Yellow and White lane masks are applied to get both types of lanes.
3. Resultant image is then converted to greyscale
4. In order to improve local contrast CLAHE is applied. This technique is mentioned in paper "Curved Lane Detection using improved Hough Tansform and CLAHE in a multichannel ROI" by Gurjyot Kaur et al.
5. Finally Gaussian blur is applied to the resultant image

####Step 4: Blended mask
Both masked lane images from step 2 and step 3 are blended in order to provide a better edge mask

####Step 5: Hough transform  
Next Hough transform is applied. This process returns a bunch of line segments available in the lane image. Following hyperparameters are configured
rho = 2# distance resolution in pixels of the Hough grid
   theta = np.pi/180
   threshold = 5  
   min_line_len = 20
   max_line_gap = 60

####Step 6: Lane points
Takes input as array of HoughLine segments generated in previous step and returns left and right lane segment points in an array. Left and right lanes are separated based on their slopes. Any line segment with slope close to zero ( horizontal lines ) and infinity (vertical lines) are filtered out.

####Step 7: Draw lane lines
Following steps are used to draw lanes based on the left and right lane segments
1. A blank mask of same size as the image is created.
2. If both left and right lane points are available, we use least square regression method to fit a line through these points. In cases when we are unable to get left and right lane points, previous frames lane markers are utilized. A lane marker is defined as starting and ending points of a lane.
3. In order to avoid jittery effect in the video, previous frames lane markers are regressed with current lane markers. This ensures smoothness of the lanes across frames
4. These newly calculated lane markers are checked with previous ones to ensure that lines don't deviate too much. If a newly calculated lane marker is off by too many pixels then the one from previous frame is used.
5. Fit left and right lane lines based on the lane markers on the masked image.

####Step 8: Overlap with original image
Blend the calculated lane lines with the original image to create the results.


###2. Identify potential shortcomings with your current pipeline

1. Current code transforms given frames to fixed size before detecting lanes on it. This can be easily fixed by using image shape instead of hard parameters in functions
2. Lanes with markings in center will not be detected properly by the current pipeline.


###3. Suggest possible improvements to your pipeline

Possible areas of improvement:
1. Using splines to fit the lanes instead of using lines.
2. Make use of distance between the lanes at different positions to keep both lines uniform with each other.
3. Use adaptive regions of interests based on histograms for lane detection
4. LSD a linear-time Line Segment Detector, proposed by Rafael Grompone von Gioi could potentially provide more accurate results.
