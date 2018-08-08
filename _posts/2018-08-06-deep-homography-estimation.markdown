---
layout: post
title:  "Deep Homography Estimation in PyTorch"
date:   2018-08-06 23:10:45 -0800
categories: computer vision,pytorch,homography 
---

In 2016, Magic Leap released [*Deep Image Homography Estimation*][paper], in which a homography is estimated with a neural network instead of the traditional geometric approach. No code was released, so I was curious to play around with it. My code can be found on [__github__][github].

A homography defines a transformation between two planes, including:
- An image plane and a planar object.
- Two different views of a planar object (or a very distant scene).
- Two images taken by a rotating camera.

Homographies are used in panorama stitching, camera calibration, and estimation of the camera pose.

As a 2D perspective transform in homogeneous coordinates, a homography can be represented as a 3x3 matrix, with 8 degrees of freedom. Estimating the homography requires 4 matching points in each image. If exact correspondences are not known, more points are provided and a technique like RANSAC is used. 

The authors of this paper do not learn the 8 parameters of the homography matrix directly, as it is notorously difficult to learn a target consisting of rotational and translational terms. The target is instead the x and y perturbations (in pixels) to the 4 corners of the patch extracted in estimating the homography.

The main contribution from the authors is a means to generate an endless supply of homography data. MS-COCO images are resized to 320x240, converted to grayscale, then a 128 pixel patch is extracted. The x and y coordinates of each patch corner are perturbed by a random amount between -32 and 32. With the 4 corresponding corners, the transform from the perturbed corners to the original can be estimated. The transform is applied to the image and then a 128x128 patch, at the same location as the original image, is extracted. The result is a pair of images that can be stacked in the channel dimension and a set 8 values, which define a homography. This serves as the input and target data for a neural network.

Notes:
- The authors took the inverse of the homography between the original and perturbed corners, when they could have just estimated the homography between the perturbed and original.
- This dataset is created to simulate two different views of the same object. It would not work for homographies defining rotational views, as in panorama stitching, since there would be very little overlap in the patches.
- The authors did not mention scaling the corner perturbations, but I found dividing by 32 and putting a tanh layer on the output of the network was necessary for training.

I implemented the regression form of their network, and followed the dataset prep, architecture, and training parameters specified in the paper.   

![good performance on test image](assets/good_img_1.png){:class="img-responsive"}

![good performance on test image](assets/good_img_2.png){:class="img-responsive"}

![bad performance on test image](assets/bad_img_1.png){:class="img-responsive"}

 
{% highlight python %}
def model_fn(x):
  '''x is a placeholder'''
  x = tf.layers.dense(x, 10, activation=tf.nn.softmax)

def train():
  input_tens = tf.placeholder(tf.float32, (None, 784))
  output_tens = model_fn(input_tens)
 
  y_target = tf.placeholder(tf.float32, (None, 10))

#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

[paper]: https://arxiv.org/pdf/1606.03798.pdf
[github]: https://github.com/ekrim/deep-homography
