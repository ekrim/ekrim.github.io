---
layout: post
title:  "Logging in TensorFlow: Hooks, Metrics, and Summaries"
date:   2018-01-03 23:10:45 -0800
categories: tensorflow
---

Hooks:
- name softmax "softmax_tensor" for the logging_hook
-  
  where

  what

  exclusions


Eval metric ops:
  where
    - estimator spec for eval

  what 
 
  exclusions
  
  


Jekyll also offers powerful support for code snippets:

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

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
