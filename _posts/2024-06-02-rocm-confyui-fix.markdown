---
layout: post
title: "Fixing ComfyUI with ROCm on Fedora"
date: 2024-06-02 18:00:00 +0200
categories: post
---

## The Problem

The environment variable PYTORCH_HIP_ALLOC_CONF wasn't modifying the GPU memory allocation effectively.

## Diagnostic

Running the command:

{% highlight bash %}
watch -n 1 rocm-smi
{% endhighlight %}

showed that VRAM usage was always between 60-70% when using ComfyUI with AnimateDiff after the first pass. Eventually, VRAM usage would hit 100%, causing the system to crash or become unstable.

## The Solution

Adding the following code snippet to the main.py file of ComfyUI resolved the issue:

```

import os

os.environ['PYTORCH_HIP_ALLOC_CONF']='garbage_collection_threshold:0.7,max_split_size_mb:128'

```

![Image]({{ site.baseurl }}/images/s1.png)

Setting the PYTORCH_HIP_ALLOC_CONF environment variable in Python is crucial. Specifically, this configuration does the following:

garbage_collection_threshold: Sets the threshold for garbage collection to 0.7, which is a reasonable value.
max_split_size_mb: Limits the maximum size of memory splits (in megabytes) to prevent excessive fragmentation.

## The Result

After setting the PYTORCH_HIP_ALLOC_CONF environment variable and restarting the Python script, ComfyUI began working smoothly. There were no more crashes or memory-related issues. The GPU was utilized efficiently, and deep learning tasks ran without hiccups.
