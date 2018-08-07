---
layout: post
title:  "Deep Homography Estimation in PyTorch"
date:   2018-08-06 23:10:45 -0800
categories: computer vision,pytorch,homography 
---

A homography defines a transformation between two planes, including:
- An image plane and a planar object.
- Two different views of a planar object (or a very distant scene).
- Two images taken by a rotating camera.

As a 2D perspective transform in homogeneous coordiantes, a homography can be represented as a 3x3 matrix, with 8 degrees of freedom. Estimating the homography requires 4 matching points in each image. If exact correspondences are not known, more points are provided and a technique like RANSAC is used.

Knowing the homography is useful:

In [__Deep Image Homography Estimation__][paper] from Magic Leap, the

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
