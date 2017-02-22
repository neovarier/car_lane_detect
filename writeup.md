#**Finding Lane Lines on the Road** 


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
  - left & right edges of lanes
  - horizontal edges of lane blocks
  - and many other outliers like edges of the small lane blocks, etc.
  To separate out the left and right lanes, it is calculating the slope of the detected lines
  and based on the slope being positive or negative, it is dividing the lines into two lane sets (left set & right set).
  It is rejecting the following lines as outliers:
  - lines with positive slope but are on the left hand-side of the region of interest.
  - lines with negative slope but are on the right hand-side of the region of interest.
  - lines with slope = 0
![alt text][image7]    
  - Averaging
  The filtering would output the two sets of lines (left & right) which are predominantly part of the right and left lanes.
  An average of lines in each set would generate a single line which should fall within the edges/boundaries of that lane.
![alt text][image8] 
  
  - Extrapolation
  Previous step gives two lines which represent the right and left lane.
  But these are small lines which do not cover the length of the lanes.
  By finding the representation of the lines in the form of y = mx + c,
  the x-intercept of the average line on top and bottom line of the polygon
  of region of interest
![alt text][image9]
* Superimposition of the  denotes lines on the original image
![alt text][image10]


###2. Experience
Using step 2 simplifies the problem as it discard other portions of the frame that
could potentially result in outlier edges and lines.
Specially the edges of the shadows on road, bonnet, etc.
If we do not use this step, then we end with many outlier edeges and lines.
I had tried to remove the outlier lines by trying the fit the slopes of the
detected lines in a gaussian curve where mu is the average of the slopes of
the detected lines and sigma is the standard deviation of the slopes.
Any lines with slope where (slope - mu)/sigma > sigma should discard the outlier lines
But there were some cases the number of outlier are more than the genuine lines
which was resulting in an inverse bell curve. Also there were cases where there are only
outliers. It was difficult apply multiple filtering to get the correct lines.

After using step 2 of retaining the pixels with yellow and white color and blacking others
was discarding all the potential outliers

###3. Identify potential shortcomings with your current pipeline
A shortcoming with the current pipeline is that I am making an assumption
that the lanes are of color - yellow or white.
If the lanes are of any other color, then my pipeline will not detect the lanes

Another short coming is that, if there are any yellow or white colored objects on the road,
they will get detected and will hamper the line detection


###4. Suggest possible improvements to your pipeline

A possible improvement would be to not rely on lane colors and
come up with a robust algorithm that can discard outliers
