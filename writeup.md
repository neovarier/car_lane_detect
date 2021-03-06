#**Finding Lane Lines on the Road** 

The repo contains the following:
 - output folder contains the output images and videos of the pipeline
 - LaneDetect.ipynb source code for pipeline
 - Input images and videos

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road

[//]: # (Image References)

[image1]: ./whiteCarLaneSwitch.jpg "Input"
[image2]: ./output/masked.jpg "Masked"
[image3]: ./output/grayscale.jpg "Grayscale"
[image4]: ./output/edges.jpg "Edges"
[image5]: ./output/roi.jpg "Region of Interest"
[image6]: ./output/houghlines.jpg "Hough Lines"
[image7]: ./output/lines_filt.jpg "Filtered Lines"
[image8]: ./output/avg_lines.jpg "Average Lines"
[image9]: ./output/lines.jpg "Final Lines"
[image10]: ./output/final.jpg "Final"
---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline executes the following steps:
* Take the input RGB image
![alt text][image1]

* Use cv2.inRange() & cv2.bitwise_and() for retaining pixels with yellow & white color.
  And this will blacken out all the other pixels.
  The assumption here is that the lanes would be either yellow or white color
  This will result in an image with predominantly lanes in it.
![alt text][image2]

* Convert the output image in the previous step into greyscale.
![alt text][image3]

* Apply gaussian blur and then canny edge on the image.
![alt text][image4]

* Apply region of interest polygon on the image to the region where lane lines would get detected
  - Hardcoded vertices for region of interest will not work for videos with varying resolutions
  - Calculating the coordinates of the polygon vertices relative to the image resolution would be better
    to make the pipeline robust
![alt text][image5]

* Detect lines using HoughLinesP
* Next step is generate two lines  which denote the left and right lanes
  - Filtering
    In the previous step, HoughLninesP would output many lines that the algorithm has detected
  
![alt text][image6]
    It would detect lines which are
    1)left & right edges of lanes
    2)horizontal edges of lane blocks
    3)and many other outliers like edges of the small lane blocks, etc.
    To separate out the left and right lanes, it is calculating the slope of the detected lines
    and based on the slope being positive or negative, it is dividing the lines into two lane sets (left set & right set).
    It is rejecting the following lines as outliers:
    1)lines with positive slope but are on the left hand-side of the region of interest.
    2)lines with negative slope but are on the right hand-side of the region of interest.
    3)lines with slope = 0
    
![alt text][image7]

  - Averaging
    The filtering would output the two sets of lines (left & right) which are predominantly part of the right and left lanes.
    An average of lines in each set would generate a single line which should fall within the edges/boundaries of that lane.
    
![alt text][image8] 
  
  - Extrapolation
    Previous step gives two lines which represent the right and left lane.
    But these are small lines which do not cover the length of the lanes.
    By finding the representation of the lines in the form of y = mx + c,
    the x-intercept of the average line is found on top and bottom line of the polygon
    of region of interest
 ![alt text][image9]
 
* Superimposition of the  denotes lines on the original image
The annotates lines are superimposed onto the original image
using cv2.addWeighted
![alt text][image10]


###2. Methodologies tried
Using step 2 simplifies the problem as it discard other portions of the frame that
could potentially result in outlier edges and lines.
Specially the edges of the shadows on road, bonnet, etc.
If we do not use this step, then we end with many outlier edeges and lines.
I had tried to remove the outlier lines by usinggaussian curve where mu is the average of the slopes of
the detected lines and sigma is the standard deviation of the slopes.
I discarded the lines with (slope - mu)/sigma > 1.
But there were some cases where the number of outliers were more than the genuine lines
which was resulting in an inverse bell curve. Also there were cases where there were only
outliers. It was inefficient to apply multiple filtering to get the correct lines.

Using step 2, retaining the pixels with yellow and white color and blackening others,
helped in discarding all the potential outliers in an efficient manner.

###3. Identify potential shortcomings with your current pipeline
A shortcoming with the current pipeline is that I am making an assumption
that the lanes are of color - yellow or white.
If the lanes are of any other color, then my pipeline will not detect the lanes

Another short coming is that, if there are any yellow or white colored objects on the road,
they will get detected and will hamper the line detection


###4. Suggest possible improvements to your pipeline

A possible improvement would be to use information of lane colors in combination with
a robust filtering algorithm that can discard outliers

Another improvement would be to apply smoothing on the lines 
detected in the current frame by using the lines detected in the
previous frame.

