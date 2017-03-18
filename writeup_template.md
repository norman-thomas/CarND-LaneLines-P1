# Finding Lane Lines on the Road

#### The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[image1]: ./test_images_output/solidWhiteRight_1_Gray.jpg "Grayscale"
[image2]: ./test_images_output/solidWhiteRight_2_Blurred.jpg "Blurred"
[image3]: ./test_images_output/solidWhiteRight_3_Edges.jpg "Edges"
[image4]: ./test_images_output/solidWhiteRight_4_Masked.jpg "Masked"
[image5]: ./test_images_output/solidWhiteRight.jpg "Final"

---

# Reflection

## 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consists of 5 steps:

As learned in lesson 1 of the course, the input image is first converted from RGB to grayscale.

![grayscale image][image1]

Then, the grayscale image is blurred using a Gaussian blur with 5x5 kernel, which was judged to be a good value during lesson 1.

![blurred image][image2]

From the blurred image edges are detected using the Canny edge detection algorithm. Despite the recommended 1:2 or 1:3 ration between low and high threshold, I found that a low threshold of 140 and high threshold of 180 worked best for me with the provided imagery.

![detected edges][image3]

In order to avoid working with irrelevant and distracting data, a region of interest is defined. Any edges detected outside of that region are clipped. As a region of interest I defined a triangle stretching from the bottom left and right corners of the image to the approximate center (7% below the actual center) of the image.

![masked image][image4]

Out of the remaining edges, which are just discreet points, line segments which connect those dots are created via Hough line transformation.

![final result][image5]

In order to draw a single line on the left and right lanes, I modified the `draw_lines()` function. I calculate the slope of each detected line. The lines are group by positive and negative slopes. As the second video also has horizontal lane markings, I added a threshold (`horizontal_threshold`) to ignore horizontal lines with a slope below 10°. Then, all positive and all negative lines are merged separately. In order to do that, the average slope is calculated. An average intersect with the `y`-axis is also calculated. Both result in the required parameters for a linear function. With the help of the calculated function, for each rising and falling group of lines, a single line stretching from the bottom of the image to the top of the region of interest is drawn.


## 2. Identify potential shortcomings with your current pipeline

While calculating the slope, which is necessary to group the lines, perfectly vertical lines (where start and end point have the same `x` value) are ignored. When changing lanes on the highway this will inevitably happen, albeit very momentarily and only for a small amount of line segments. Other line segments on that lane would likely still be detected and marked correctly.

Distracting horizonal lines are ignored, which seems fine for a highway scenario as provided in the images and videos with this project. In other surroundings, though, driving across lanes is a valid scenario, in which horizonal lane markings should not be ignored.

Grouping by slope depends on the region of interest. If that region of interest accidentally includes lines from other lanes, the result will be far less accurate as the distant lines will be intergrated into the average calculation.

As seen in the result video of the challenge, this whole approach only works well with straight lanes. Curves are not handled very well. Furthermore, different materials and colors of the road impact the edge detection as the contrast between road colors and line colors is too low.


## 3. Suggest possible improvements to your pipeline

To tackle the shortcomings above, some changes could be made. Instead of ignoring horizontal lines completely or relying on a tight region of interest, I could imagine to group lines depending on their slope and position. One could iterate over all lines and group it by slope and position with those lines that have similar values, below a certain threshold.

Instead of ignoring other lanes, being aware of them makes more sense to me. With the stragegy just mentioned this could also be solved.

In order to be able to detect curved lanes, instead of the Hough line transformation, which just detects straight lines, another algorithm could be applied, which allows interpolation of points (from the edge detection) distributed in a curved shaped.
