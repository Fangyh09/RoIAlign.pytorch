# RoIAlign for PyTorch
This is a PyTorch version of [RoIAlign](https://arxiv.org/abs/1703.06870).
This implementation is based on `crop_and_resize`
and supports both forward and backward on CPU and GPU.


## Introduction
The `crop_and_resize` function is ported from [tensorflow](https://www.tensorflow.org/api_docs/python/tf/image/crop_and_resize),
and has the same interface with tensorflow version, except the input feature map
should be in `NCHW` order in PyTorch.
They also have the same output value (error < 1e-5) for both forward and backward as we expected,
see the comparision in `test.py`.

**Note:**
Document of `crop_and_resize` can be found [here](https://www.tensorflow.org/api_docs/python/tf/image/crop_and_resize).
And `RoIAlign` is a wrap of `crop_and_resize`
that uses boxes with *unnormalized `(x1, y1, x2, y2)`* as input
(while `crop_and_resize` use *normalized `(y1, x1, y2, x2)`* as input).
See more details about the difference of
 `RoIAlign` and `crop_and_resize` in [tensorpack](https://github.com/ppwwyyxx/tensorpack/blob/6d5ba6a970710eaaa14b89d24aace179eb8ee1af/examples/FasterRCNN/model.py#L301).

**Warning:**
Currently it only works using the default GPU (index 0)

## Usage
+ Install and test
    ```
    conda activate <CONDA_ENVIRONMENT>
    ./install.sh
    ./test.sh
    ```

+ Use RoIAlign or crop_and_resize
    ```python
    from roi_align.roi_align import RoIAlign      # RoIAlign module
    from roi_align.roi_align import CropAndResize # crop_and_resize module

    # input data
    image = to_varabile(image_data, requires_grad=True, is_cuda=is_cuda)
    boxes = to_varabile(boxes_data, requires_grad=False, is_cuda=is_cuda)
    box_index = to_varabile(box_index_data, requires_grad=False, is_cuda=is_cuda)

    # RoIAlign layer
    roi_align = RoIAlign(crop_height, crop_width)
    crops = roi_align(image, boxes, box_index)
    ```

+ [Issue1](https://github.com/longcw/RoIAlign.pytorch/issues/1): gradcheck
    and difference of `RoIAlign` and `crop_and_resize`.

+ Changing `-arch` in `make.sh` for your GPU
    ```
    # Which CUDA capabilities do we want to pre-build for?
    # https://developer.nvidia.com/cuda-gpus
    # Compute/shader model   Cards
    # 6.1                    P4, P40, Titan Xp, GTX 1080 Ti, GTX 1080
    # 6.0                    P100
    # 5.2                    M40, Titan X, GTX 980
    # 3.7                    K80
    # 3.5                    K40, K20
    # 3.0                    K10, Grid K520 (AWS G2)
    ```
## Usage
```python
import numpy as np
import torch
from torch.autograd import Variable

from roi_align.roi_align import RoIAlign


def to_varabile(arr, requires_grad=False, is_cuda=True):
    tensor = torch.from_numpy(arr)
    if is_cuda:
        tensor = tensor.cuda()
    var = Variable(tensor, requires_grad=requires_grad)
    return var


# inputs
is_cuda = False
image_data = np.tile(np.arange(7, dtype=np.float32), 7).reshape(7, 7) # attention the type
image_data = image_data[np.newaxis, np.newaxis]
boxes_data = np.asarray([[0, 0, 2, 2]], dtype=np.float32) # attention the type
box_index_data = np.asarray([0], dtype=np.int32) # attention the type

image_torch = to_varabile(image_data, requires_grad=True, is_cuda=is_cuda)
boxes = to_varabile(boxes_data, requires_grad=False, is_cuda=is_cuda)
box_index = to_varabile(box_index_data, requires_grad=False, is_cuda=is_cuda)

# set transform_fpcoor to False is the crop_and_resize
roi_align = RoIAlign(3, 3, transform_fpcoor=False)
print(roi_align(image_torch, boxes, box_index))
```
```python
(0 ,0 ,.,.) = 
  0  1  2
  0  1  2
  0  1  2
[torch.cuda.FloatTensor of size 1x1x3x3 (GPU 0)]
```
