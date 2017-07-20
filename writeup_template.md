# **Finding Lane Lines on the Road** 

## Writeup

### How the pipeline was implemented, shortcoming and possible enhancements

My pipeline started with steps as directed during the lessons; after a preliminary call to blur (that would be in addition to the blurring performed by the canny algorithm), the code then calls the canny edge detection, strips out all the area defined by a trapezoid, which overestimates where the road is meant to be in the images, and pass the output to hough_lines.
With the provided implementation of draw_lines, and a little of trial-and-error to tune the paramenters, the output was already quite acceptable on the example images.
Also the execution of the code in that state against the first video performed quite well.
I then moved into reimplementing draw_lines to extrapolate the two lane lines; I purposedly left the drawing of the red lines returned by houghTrasformP() for verification purposes; as suggested by the comment in the method definition, I started with checking the slope inside the loop iterating thru the lines; assigning to left or right lane array (of points) dependening on the sign, and the fitting a line using openCV fitLine call (using blue for left and green for right lane).
The result on the images was good just at this step:

<img src="test_images_output/solidWhiteCurve.jpg" width="300"><img src="test_images_output/solidYellowCurve.jpg" width="300">
<img src="test_images_output/solidYellowLeft.jpg" width="300"><img src="test_images_output/solidWhiteRight.jpg" width="300">
<img src="test_images_output/solidYellowCurve2.jpg" width="300"><img src="test_images_output/whiteCarLaneSwitch.jpg" width="300">


Also the output on the first video was acceptable. What I had to amend here was adding a second call to intereseting_region() to crop the area of interest after drawing the lane lines, reusing the same trapezoid as used for canny output, in order to fit them correctly.

The second video instead was giving some issues, about lines being identified by hough_lines that were not part of the lanes, lines that would be missing on the lanes themselves, generating glitches as the green and blue lane lines being off (not being shown at all for certain frames), or being blatantly mal-positioned. To workaround this effect I first added second pass of gaussian blur at the start of the pipeline: this had no benefits, but no other downsides either on the image output or the first video, so I decided to keep it.

Second attempt was to add a dilate-erode-dilate pass just after the blurring, it is a technique I used past summer on a pet project to facilitate canny identify edges, with somewhat good results before running watershed, but in this case I got no real advance for the second video.

The third attempt was at the same time, in draw_lines, to only include to the left lane array of points, points that would be in the left half of the screen, and likewise for the right lane only points that would be in the right side of the screen. This gave a few good results, but made the challenge video a complete disaster. Decided to tackle the challenge video later and moved to the second change at this stage, which was introducing a top-level double-ended queue instance for the right and left lanes, to be used as a cache for the fitted lines returned by cv2.fitLine(). On any new frame, I calculate the mean and the standard deviation of the fitted lines in cache, and only accept new lines that would be inside a threshold. If no lines are present, just return the most recent fitted line; if the line is accepted or not, return the mean of all the lines in cache. Note that the caching is parametrized as of number of lines for initialization (before calculating the mean), number of total objects and multiplier for the standard deviation conditional. I have to say I can probably find better values for the parameters with more time, but in any case, in a mature implemnetation those would need to be computed given the number of frames per second in the video. 

That latest change fixed the problem of missing lanes in certain frames, and also the problem of lanes drawn completely off place; unfortunately, it introduced some level of "lazyness", making the fitted lines follow the hough lines with some retard. This could have very detrimental effects in a real self driving car I guess, nevertheless, I think that the values could still be found so that the response time would still be much better that an human driver.

Next, the challenge video. It was still a disaster because a big number of hough lines is found that was unexpected. I believe a possible fix is determining a better interesting area, by using the intersection of the two extrapoleted lane lines as the height of the trapzoid used for seleting the interesting area. I started to implement that but could not get to it in time before the deadline.  Another possible solution would trying to apply a threshold twice to the frame (with BINARY and BINARY_INVERTED) in roder to try to separate the foreground from the background, then work on the background only (which I belive would contain the road and the sky, but couldn't verify that).








