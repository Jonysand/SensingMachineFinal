# Light Up Where You Are! - SensingMachine Final Project

<!-- vim-markdown-toc GFM -->

* [Brief introduction](#brief-introduction)
* [Tech detail](#tech-detail)
	* [Kinect tracking](#kinect-tracking)
	* [Position Mapping](#position-mapping)
	* [Physical Computing](#physical-computing)
* [Other Credit](#other-credit)

<!-- vim-markdown-toc -->

## Brief introduction
This is a course project On Sensing Machine 2019 Fall in ITP by <b>*Yongkun Li*</b>. The project is installed in a room, where the kinect is on the ceiling. When people enter the room, the kinect will detect the position of the people, and the LED on the matrix will light up according to the position.

The idea is inspired by <b>*Jason Xu*</b> and <b>*Rae Huang*</b>'s PCOMP final project. The link will be updated in the future.

## Tech detail

### Kinect tracking
To detect the position of the person in the room, I use thresholded blob finding. After getting the depth data from Kinect, I do double sided threshold, then use `findContour()` to get where the position. Ideally, this will be the position of people's head. The threshold can be modified in the GUI panel.

There's still one problem remains to be solved, which is that when the people in the room are shorter than furnitures, this tracking method can be biased. 

Thanks <b>*Yeseul Song*</b> for helping with the blob finding algorithm.

### Position Mapping
Since the angle of the Kinect varies when installing in the room, we need to map the position from Kinect perspective to the floor. To do this, the first step is to find the transforming matrix.

First we get 8 points, 4 on on the Kinect perspective, 4 on the simulated floor. When adjusting the mapping matrix, I use `cv::findHomography()` to get the transformation matrix `homographyMat`.

Then we need to calculate the mapped position using the transforming matrix. For image warping the function `warpPerspective()` is used, and in this function people use the following formula to get the transformed position:

![Transforming Matrix](./TransMatrix.png) 

After getting the position from the Kinect's perspective, I calculate the cross product of `homographyMat` and source point position matrix, then get the destination position. The calculation of $src(S_{x}, S_{y})$ to $dst(D_{x}, D_{y})$ is shown as the following:

<img src="https://latex.codecogs.com/gif.latex?src = (S_{x}, S_{y});$" />

<img src="https://latex.codecogs.com/gif.latex?
holographyMat = \begin{bmatrix} 
M_{11} & M_{12} & M_{13} \\
M_{21} & M_{22} & M_{23} \\
M_{31} & M_{32} & M_{33} \\
\end{bmatrix};" />

<img src="https://latex.codecogs.com/gif.latex?
tempMat = holographyMat \cdot 
\begin{bmatrix}S_{x}\\S_{y}\\1\end{bmatrix} 
= \begin{bmatrix}t_{1}\\t_{2}\\t_{3}\end{bmatrix};" />

<img src="https://latex.codecogs.com/gif.latex?
dst = (\frac{t_{1}}{t_{3}}, \frac{t_{2}}{t_{3}}) = (D_{x}, D_{y});" />

### Physical Computing
After getting the position of people, we calculate the index of grid where the position belongs to. Since I only use 84 LEDs (12*7), whose amount is lower than 256, I can just send the index in the format of `byte` to the Arduino. 

I use `NEO Pixel Library` to program the LEDs, which just needs `Index` and `Color` to address each LEDs. The Arduino keeps receiving data from Openframework, and lights up the LED according to the position.

## Other Credit
Thanks <b>*Fanyi Pan*</b> for helping with the fabrication of LED matrix.

