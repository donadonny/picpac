Basic Usage
===========

```
import picpac

seed = 1996
config = dict(seed=seed,
	loop=False,
	shuffle=True,
	reshuffle=True,
	batch=batch,
	# unless batch is 1, always set resize_width and resize_height
	# as samples in the same batch must have the same size
	resize_width=256,
	resize_height=256,
	split=K,
	# for K-fold cross validation, run K times with split_fold = 0, 1, ..., K-1
	split_fold=0,
	channels=1,
	stratify=False,
	pert_color1=10,
	pert_angle=5,
	pert_min_scale=0.8,
	pert_max_scale=1.2,
	pad=False,
	pert_hflip=True,
	)
train_stream = picpac.ImageStream(db_path, negate=False, perturb=True, **config)
valid_stream = picpac.ImageStream(db_path, negate=True, perturb=False, **config)

while True:
    try:
        images, labels, pad = train_stream.next()
	# when config.pad is True, partial batches might be returned.
	# the returned images and labels are still the batch size,
	# and the number of padding items are returned as "pad".
	# "pad" is always 0 if config.pad is False.

	# do training
    except StopIteration:
	break

```



MXNet Usage
===========

```
from picpac.mxnet import ImageStream


# create ImageStream using the same configuration parameters
# picpac.mxnet.ImageStream is inherited from mxnet.io.DataIter

```

Neon Usage
==========

```
from picpac.neon import ImageStream

# create ImageStream using the same configuration parameters
# picpac.meon.ImageStream is inherited from neon.data.dataiterator.NervanaDataIterator

```

Caffe Usage
===========

Use this Caffe (fork)[https://github.com/aaalgo/caffe-picpac].

```
layer {
  name: "data"
  type: "PicPac"
  top: "data"
  top: "label"
  picpac_param {
    path: "path_to_db"
    batch: 16
    channels: 3
    split: 5
    split_fold: 0
    resize_width: 224
    resize_height: 224
    threads: 4
    perturb: true
    pert_color1: 10
    pert_color2: 10
    pert_color3: 10
    pert_angle: 20
    pert_min_scale: 0.8
    pert_max_scale: 1.2
  }
}

```

Note that the above specification always go through the following touchup:
```
  config_.loop = true;
  if (this->phase_ == TRAIN) {
      config_.reshuffle = true;
      config_.shuffle = true;
      config_.stratify = true;
      config_.split_negate = false;
  }
  else {
      config_.reshuffle = false;
      config_.shuffle = true;
      config_.stratify = true;
      config_.split_negate = true;
      config_.perturb = false;
      config_.mixin.clear();
  }

```



Parameter Documentation
=======================
The following parameters are exposed to Python and Caffe.

* General Streaming Parameters
	- int32 seed: randomization seed
	- bool loop: loop through records for endless streaming, instead of throwing StopIteration when all data are consumed.
	- bool shuffle: shuffle input records order before splitting the data.
	- bool reshuffle: shuffle input records in the beginning of every loop from the second loop on.
	- bool stratify: divide data by category, split and loop within each category, so any sliding window on the stream contain about same number of images for each category.
	- bool cache: cache all data in memory after they are read.  Default is true, set to false when data do not fit in memory.
	- uint32 preload: pre-load queue size.
	- uint32 threads: number of decoding/pre-processing threads, default = 4.
	- uint32 batch: batch size
	- bool pad: if loop is false and there's not enough to produce a whole batch, throw StopIteration if not pad, otherwise return a whole batch with random padding.  Number of padded samples are also returned.

* Cross validation
	- uint32 split: equally split all data (or each category if stratify = true) into this number of partitions.
	- int32 split_fold: exclude examples from this split.
	- bool split_negate: use the rest of samples.

For example, 5-fold cross validation can be run with split = 5, and with split_fold = 0, 1, 2, 3, 4.
For each value of split_fold, set split_netage = false when training, and true when validating.

* Image Control
	- int32 channels: number of image channels, ignored if <= 0
	- int32 min_size: 
	- int32 max_size:
	- int32 resize_width: resize to this, ignored if <= 0
	- int32 resize_height: resize to this, ignore if <= 0
	- int32 decode_mode: as in the second parameter of cv::imread.  Set to -1 to load images as is.
	- bool bgr2rgb: default 3-channel images are stored as BGR (opencv behavior), set to true for RGB.

* Image Annotation
	- string annotate: for annotation, set to "json" or "image"
	- int32 anno_type: annotation image type, as in cv::Mat::type()
	- bool anno_copy: copy image to label, for visualization.  Set to false for training.
	- float anno_color1: annotation channel 1.
	- float anno_color2: annotation channel 2, typically not used. 
	- float anno_color3: annotation channel 3, typically not used.
	- int32 anno_thickness: set to -1 to fill the annotation region with anno_color.

* Image Augmentation
	- bool perturb: set to true to enable augmentation
	- float pert_color1: 
	- float pert_color2
	- float pert_color3
	- float pert_angle: image/label rotated randomly within [-pert_angle, pert_angle], in degrees.
	- float pert_min_scale: image/label rescaled to a scale uniformly sampled from [pert_min_scale, pert_max_scale].
	- float pert_max_scale
	- bool pert_hflip: enable random horizontal flip.
	- bool pert_vflip: enable random vertical flip.

* Label Encoding
	- uint32 onehot: one hot encoding of label.  If onehot > 0, specify the number of categories.


