---
layout: post
title: Parsing of FlickrLogos32 dataset
categories: 
  - Computer Science
tags:
  - Computer Science
  - parsing
  - script
  - Darknet
  - KITTI
  - DetectNet
  - flickrlogos
---

In my master thesis I had to detect brand logos in different images and considered the convolutional neural networks [DetectNet](https://devblogs.nvidia.com/parallelforall/detectnet-deep-neural-network-object-detection-digits/) and [YOLOv2](https://pjreddie.com/darknet/yolo/). The dataset [FlickrLogos32](http://www.multimedia-computing.de/flickrlogos/) and an expansion called [Logos-32plus](http://www.ivl.disco.unimib.it/activities/logo-recognition/) were chosen for the training and validation.

Following I will describe some issues I had preparing the datasets and how the parsing script works, which you can find in my [GitHub repository](https://github.com/cfloeth/Tools/blob/master/ParseFlickrLogos.cs).

### Problems

During searching for a working solution, which let me train without errors and is able to detect brand logos in images, a few problems arised:

* Merging of FlickrLogos32 and Logos-32Plus was not as successful as I thought, because the class 'guinness' was written with just one 'n' in FlickrLogos32, but correctly in Logos-32Plus
* Parsing to the KITTI format and the format used by Darknet

### Solving the issues

The first issue is of course a rather simple one, but it took me some time to realize, what was happening. Just change the class name in the FlickrLogos32 dataset and you are set. Or leave it as it is, if you are not using the expansion.

The second "issue" is the process to convert the bounding box notation of FlickrLogos32 to the one used by Darknet. Every bounding box has four values in the FlickrLogos32 dataset: x, y, width, height, where (x,y) is top left corner of the bounding box.

#### KITTI

The KITTI format requires different values for each bounding box. In my case only five of them were set and the rest could be initialized as 0. Following I parse a bounding box of an class object from FlickrLogos32 format to KITTI format:

	50 10 100 200 --> class 0.00 0 0.0 50 10 150 210 0.0 0.0 0.0 0.0 0.0 0.0 0.0

You might notice the new notation of the bounding box. It is described by the top left and bottom right corner. The first value is the class name of the object in the box. Three dimensional bounding boxes can be described by the other values.

#### Darknet

A feature of training with Darknet is that it will change the resolution of the training images after a set amount of training steps. Therefore you do not have to resize your images beforehand - as a consequence the bounding boxes can not be described by absolute values. The darknet format requires the center of the bounding box, the width and height relative to the image dimensions.

We use a 100x50 image as an example to parse the bounding box at (30,20) and dimensions of 10x10 to a relative representation. To get the center of the bounding box we can use the following way:

	center_x = x + (width / 2)
	center_y = y + (height / 2)

	center_x = 30 + (10 / 2) = 35
	center_y = 20 + (10 / 2) = 25

With the center and the bounding box dimensions it is possible to calculate the values relative to the image dimensions:

	center_x = 35 / 100 = 0.35
	center_y = 25 / 50 = 0.5
	width = 10 / 100 = 0.1
	height = 10 / 50 = 0.2

	30 20 10 10 --> 1 0.35 0.5 0.1 0.2

The '1' is the index of the class, which is declared in another file.

### Conclusion

In this post I showed my way to parse the FlickrLogos32 dataset into the KITTI and Darknet format. I hope I could save you some time with this little explanation. Maybe I will go into some other details later one.

A small note to my [script](https://github.com/cfloeth/Tools/blob/master/ParseFlickrLogos.cs): the five parameters needed are 'classes directory', 'split percentage for validation set', 'include images without logos', 'side length for resizing' and 'format'. At the moment you can not use 'true' to use images without logos. I chose '500' for the resized side length to get a dataset, which holds images of similar sizes, but it is not necessary. It could lead to more problems, if the image has to be scaled down and the objects displayed in it are too small for detection. 

An example of usage for the parameters is:


	"Path\to\FlickrLogos-32plus_dataset_v2\FlickrLogos-v2\classes" "0.1" "false" "500" "darknet"


Thanks for reading!