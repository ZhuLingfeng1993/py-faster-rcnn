# 	py-faster-rcnn has been deprecated. Please see [Detectron](https://github.com/facebookresearch/Detectron), which includes an implementation of [Mask R-CNN](https://arxiv.org/abs/1703.06870).

### Disclaimer

The official Faster R-CNN code (written in MATLAB) is available [here](https://github.com/ShaoqingRen/faster_rcnn).
If your goal is to reproduce the results in our NIPS 2015 paper, please use the [official code](https://github.com/ShaoqingRen/faster_rcnn).

This repository contains a Python *reimplementation* of the MATLAB code.
This Python implementation is built on a fork of [Fast R-CNN](https://github.com/rbgirshick/fast-rcnn).
There are slight differences between the two implementations.
In particular, this Python port
 - is ~10% slower at test-time, because some operations execute on the CPU in Python layers (e.g., 220ms / image vs. 200ms / image for VGG16)
 - gives similar, but not exactly the same, mAP as the MATLAB version
 - is *not compatible* with models trained using the MATLAB code due to the minor implementation differences
 - **includes approximate joint training** that is 1.5x faster than alternating optimization (for VGG16) -- see these [slides](https://www.dropbox.com/s/xtr4yd4i5e0vw8g/iccv15_tutorial_training_rbg.pdf?dl=0) for more information

# *Faster* R-CNN: Towards Real-Time Object Detection with Region Proposal Networks

By Shaoqing Ren, Kaiming He, Ross Girshick, Jian Sun (Microsoft Research)

This Python implementation contains contributions from Sean Bell (Cornell) written during an MSR internship.

Please see the official [README.md](https://github.com/ShaoqingRen/faster_rcnn/blob/master/README.md) for more details.

Faster R-CNN was initially described in an [arXiv tech report](http://arxiv.org/abs/1506.01497) and was subsequently published in NIPS 2015.

### License

Faster R-CNN is released under the MIT License (refer to the LICENSE file for details).

### Citing Faster R-CNN

If you find Faster R-CNN useful in your research, please consider citing:

    @inproceedings{renNIPS15fasterrcnn,
        Author = {Shaoqing Ren and Kaiming He and Ross Girshick and Jian Sun},
        Title = {Faster {R-CNN}: Towards Real-Time Object Detection
                 with Region Proposal Networks},
        Booktitle = {Advances in Neural Information Processing Systems ({NIPS})},
        Year = {2015}
    }

### Contents
1. [Requirements: software](#requirements-software)
2. [Requirements: hardware](#requirements-hardware)
3. [Basic installation](#installation-sufficient-for-the-demo)
4. [Demo](#demo)
5. [Beyond the demo: training and testing](#beyond-the-demo-installation-for-training-and-testing-models)
6. [Usage](#usage)

### Requirements: software

**NOTE** If you are having issues compiling and you are using a recent version of CUDA/cuDNN, please consult [this issue](https://github.com/rbgirshick/py-faster-rcnn/issues/509?_pjax=%23js-repo-pjax-container#issuecomment-284133868) for a workaround

1. Requirements for `Caffe` and `pycaffe` (see: [Caffe installation instructions](http://caffe.berkeleyvision.org/installation.html))

  **Note:** Caffe *must* be built with support for Python layers!

  ```make
  # In your Makefile.config, make sure to have this line uncommented
  WITH_PYTHON_LAYER := 1
  # Unrelatedly, it's also recommended that you use CUDNN
  USE_CUDNN := 1
  ```

  You can download my [Makefile.config](https://dl.dropboxusercontent.com/s/6joa55k64xo2h68/Makefile.config?dl=0) for reference.

2. Python packages you might not have: `cython`, `python-opencv`, `easydict`, `python-tk`.
3. [Optional] MATLAB is required for **official** PASCAL VOC evaluation only. The code now includes unofficial Python evaluation code.

### Requirements: hardware

1. For training smaller networks (ZF, VGG_CNN_M_1024) a good GPU (e.g., Titan, K20, K40, ...) with at least 3G of memory suffices
2. For training Fast R-CNN with VGG16, you'll need a K40 (~11G of memory)
3. For training the end-to-end version of Faster R-CNN with VGG16, 3G of GPU memory is sufficient (using CUDNN)

### Installation supporting cpu-only

https://github.com/rbgirshick/py-faster-rcnn/pull/697/files

https://github.com/rbgirshick/py-faster-rcnn/pull/698 (docker)

### Installation (sufficient for the demo)

1. Clone the Faster R-CNN repository
  ```Shell
  # Make sure to clone with --recursive
  git clone --recursive https://github.com/rbgirshick/py-faster-rcnn.git
  ```

2. We'll call the directory that you cloned Faster R-CNN into `FRCN_ROOT`

   *Ignore notes 1 and 2 if you followed step 1 above.*

   **Note 1:** If you didn't clone Faster R-CNN with the `--recursive` flag, then you'll need to manually clone the `caffe-fast-rcnn` submodule:
    ```Shell
    git submodule update --init --recursive
    ```
    **Note 2:** The `caffe-fast-rcnn` submodule needs to be on the `faster-rcnn` branch (or equivalent detached state). This will happen automatically *if you followed step 1 instructions*.

3. Build the Cython modules.

    **Note:** If you are not using GPU, then change `__C.USE_GPU_NMS` to `False` in file `$FRCN_ROOT/lib/fast_rcnn/config.py`.

    Then run:
    ```Shell
    cd $FRCN_ROOT/lib
    make
    ```

4. Build Caffe and pycaffe

    **Note:** If you are not using GPU, then remember to configure the `Makefile.config` file as expected.
    Uncomment `CPU_ONLY := 1` line, comment `USE_CUDNN := 1` line and whatever necessary.
    Also, change `solver_mode: GPU` to `CPU` in file `$FRCN_ROOT/caffe-fast-rcnn/examples/mnist/lenet_solver.prototxt`.

    Then run:
    ```Shell
    cd $FRCN_ROOT/caffe-fast-rcnn
    # Now follow the Caffe installation instructions here:
    #   http://caffe.berkeleyvision.org/installation.html
    
    # If you're experienced with Caffe and have all of the requirements installed
    # and your Makefile.config in place, then simply do:
    make -j8 && make pycaffe
    ```

    报错:

    ```shell
    /usr/bin/ld: cannot find -lhdf5_hl
    /usr/bin/ld: cannot find -lhdf5
    ```

    [解决](https://github.com/rbgirshick/py-faster-rcnn/issues/474):

    f you are on Ubuntu, here is the solution:

    Go to `/usr/lib/x86_64-linux-gnu` and make two symbol links:

    ```
    sudo ln -s libhdf5_serial.so libhdf5.so
    sudo ln -s libhdf5_serial_hl.so libhdf5_hl.so
    ```

    Then, open your `Makefile.conf`, modify `INCLUDE_DIRS` and `LIBRARY_DIRS`:

    ```
    INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial/
    LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu/hdf5/serial/
    ```

    #### make runtest 报错

    [Can't make test! fatal error: caffe/vision_layers.hpp: No such file or directory #129](https://github.com/rbgirshick/py-faster-rcnn/issues/129)



    报错:
    
    ```
    src/caffe/test/test_roi_pooling_layer.cpp:28:26: error: ‘GPUDevice’ was not declared in this scope
    ```
    
     解决: 应该是只写了对GPU版实现的测试, 没有写对CPU版的测试, 所以报错, 把该文件删除再runtest, 通过

5. Download pre-computed Faster R-CNN detectors
    ```Shell
    cd $FRCN_ROOT
    ./data/scripts/fetch_faster_rcnn_models.sh
    ```

    This will populate the `$FRCN_ROOT/data` folder with `faster_rcnn_models`. See `data/README.md` for details.
    These models were trained on VOC 2007 trainval.
    
#### 训练时报错

##### smooth_L1_loss_layer Not Implemented Yet

##### roi_pooling_layer Not Implemented Yet

https://github.com/rbgirshick/caffe-fast-rcnn/pull/17

#### 训练时solving报错

##### 错误1

报错:

```shell
File "/home/zhulingfeng/Projects/py-faster-rcnn/tools/../lib/rpn/proposal_target_layer.py", line 127, in _get_bbox_regression_labels
    bbox_targets[ind, start:end] = bbox_target_data[ind, 1:]
TypeError: slice indices must be integers or None or have an __index__ method
```

```shell
  File "/home/zhulingfeng/Projects/py-faster-rcnn/tools/../lib/rpn/proposal_target_layer.py", line 168, in _sample_rois
    fg_inds = npr.choice(fg_inds, size=fg_rois_per_this_image, replace=False)
  File "mtrand.pyx", line 1192, in mtrand.RandomState.choice
TypeError: 'numpy.float64' object cannot be interpreted as an index
```

解决:

其实就是数据类型问题, 需要整数类型

https://github.com/rbgirshick/py-faster-rcnn/issues/480#issuecomment-305742882

注意根据实际情况仔细修改:

```python
bbox_targets[ind, start:end] = bbox_target_data[ind, 1:] 
```

修改为:

```python
start=int(start)
end=int(end)
bbox_targets[ind, start:end] = bbox_target_data[ind, 1:]

fg_rois_per_this_image=int(fg_rois_per_this_image)
```

```python
        fg_inds = npr.choice(fg_inds, size=fg_rois_per_this_image, replace=False)
```
修改为:
```python
        fg_rois_per_this_image=int(fg_rois_per_this_image)
        fg_inds = npr.choice(fg_inds, size=fg_rois_per_this_image, replace=False)
```

##### Check failed: *ptr host allocation of size 172800000 failed*

https://github.com/rbgirshick/py-faster-rcnn/issues/369#issuecomment-366471649

##### 其实就是内存不够

### Demo

*After successfully completing [basic installation](#installation-sufficient-for-the-demo)*, you'll be ready to run the demo.

To run the demo with GPU
```Shell
cd $FRCN_ROOT
./tools/demo.py
```
or without GPU
```Shell
cd $FRCN_ROOT
./tools/demo.py --cpu
```

The demo performs detection using a VGG16 network trained for detection on PASCAL VOC 2007.

### Beyond the demo: installation for training and testing models
1. Download the training, validation, test data and VOCdevkit

	```Shell
	wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtrainval_06-Nov-2007.tar
	wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtest_06-Nov-2007.tar
	wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCdevkit_08-Jun-2007.tar
	```

2. Extract all of these tars into one directory named `VOCdevkit`

	```Shell
	tar xvf VOCtrainval_06-Nov-2007.tar
	tar xvf VOCtest_06-Nov-2007.tar
	tar xvf VOCdevkit_08-Jun-2007.tar
	```

3. It should have this basic structure

  ```Shell
    	$VOCdevkit/                           # development kit
    	$VOCdevkit/VOCcode/                   # VOC utility code
    	$VOCdevkit/VOC2007                    # image sets, annotations, etc.
  ```
  	# ... and several other directories ...
  	```

4. Create symlinks for the PASCAL VOC dataset

	```Shell
    cd $FRCN_ROOT/data
    ln -s $VOCdevkit VOCdevkit2007
   ```
    Using symlinks is a good idea because you will likely want to share the same PASCAL dataset installation between multiple projects.
5. [Optional] follow similar steps to get PASCAL VOC 2010 and 2012
6. [Optional] If you want to use COCO, please see some notes under `data/README.md`
7. Follow the next sections to download pre-trained ImageNet models

### Download pre-trained ImageNet models

Pre-trained ImageNet models can be downloaded for the three networks described in the paper: ZF and VGG16.

```Shell
cd $FRCN_ROOT
./data/scripts/fetch_imagenet_models.sh
```
VGG16 comes from the [Caffe Model Zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo), but is provided here for your convenience.
ZF was trained at MSRA.

### Usage

To train and test a Faster R-CNN detector using the **alternating optimization** algorithm from our NIPS 2015 paper, use `experiments/scripts/faster_rcnn_alt_opt.sh`.
Output is written underneath `$FRCN_ROOT/output`.

```Shell
cd $FRCN_ROOT
./experiments/scripts/faster_rcnn_alt_opt.sh [GPU_ID] [NET] [--set ...]
# GPU_ID is the GPU you want to train on
# NET in {ZF, VGG_CNN_M_1024, VGG16} is the network arch to use
# --set ... allows you to specify fast_rcnn.config options, e.g.
#   --set EXP_DIR seed_rng1701 RNG_SEED 1701
```

("alt opt" refers to the alternating optimization training algorithm described in the NIPS paper.)

To train and test a Faster R-CNN detector using the **approximate joint training** method, use `experiments/scripts/faster_rcnn_end2end.sh`.
Output is written underneath `$FRCN_ROOT/output`.

```Shell
cd $FRCN_ROOT
./experiments/scripts/faster_rcnn_end2end.sh [GPU_ID] [NET] [--set ...]
# GPU_ID is the GPU you want to train on
# NET in {ZF, VGG_CNN_M_1024, VGG16} is the network arch to use
# --set ... allows you to specify fast_rcnn.config options, e.g.
#   --set EXP_DIR seed_rng1701 RNG_SEED 1701
```

This method trains the RPN module jointly with the Fast R-CNN network, rather than alternating between training the two. It results in faster (~ 1.5x speedup) training times and similar detection accuracy. See these [slides](https://www.dropbox.com/s/xtr4yd4i5e0vw8g/iccv15_tutorial_training_rbg.pdf?dl=0) for more details.

Artifacts generated by the scripts in `tools` are written in this directory.

Trained Fast R-CNN networks are saved under:

```
output/<experiment directory>/<dataset name>/
```

Test outputs are saved under:

```
output/<experiment directory>/<dataset name>/<network snapshot name>/
```
