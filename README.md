### Motion Prediction
This repo implements a mixture of experts approach integrated with the state of the art Spatio-Temporal Transformer building upon the open source Fairmotion library. The research paper for this repo is located here: [Project Report PDF](https://github.com/edshieh/motionprediction/blob/main/Motion_Prediction.pdf). You can also find the arxiv link here: https://arxiv.org/abs/2405.06088.


### Installation

To install motionprediction from source, first clone the git repository, use pip to download dependencies and build the project.
```
$ git clone TODO
$ cd fairmotion
$ pip install -e .
```
**Locate the soft_moe_pytorch Installation Directory**
   Find the installation directory of `soft_moe_pytorch` to correctly apply the patch:
   ```bash
   SM_PYTORCH_DIR=$(python -c "import soft_moe_pytorch; import os; print(os.path.dirname(soft_moe_pytorch.__file__))")
   ```

**Apply the Patch**
   Apply the patch to `soft_moe_pytorch` with the following command:
   ```bash
   echo "Applying patch to soft_moe_pytorch..."
   patch -p1 -d "$SM_PYTORCH_DIR" < softmoe_changes.patch
   ```

### Data Loading

Here, we load a motion capture file in the BVH file format in a python console. Similarly, there are loaders to import files from [ASF/AMC](https://research.cs.wisc.edu/graphics/Courses/cs-838-1999/Jeff/ASF-AMC.html), [AMASS](https://amass.is.tue.mpg.de/dataset) and [AMASS DIP](http://dip.is.tuebingen.mpg.de/pre_download) formats.
```
from fairmotion.data import bvh

BVH_FILENAME = “PATH_TO_BVH_FILE”
motion = bvh.load(BVH_FILENAME)
```

If you recieve errors like the ones below, you can find the workaround [here](https://stackoverflow.com/questions/65202395/pyopengl-on-macos-bigsur-and-opengl-error-nullfunctionerror#:~:text=if%20name%20%3D%3D%20%27OpenGL%27%3A%0A%20%20%20%20fullName%20%3D%20%27/System/Library/Frameworks/OpenGL.framework/OpenGL%27%0Aelif%20name%20%3D%3D%20%27GLUT%27%3A%0A%20%20%20%20fullName%20%3D%20%27/System/Library/Frameworks/GLUT.framework/GLUT%27).
```
ImportError: ('Unable to load OpenGL library', 'dlopen(OpenGL, 10): image not found', 'OpenGL', None)
or
OpenGL.error.NullFunctionError: Attempt to call an undefined function glutInit, check for bool(glutInit) before calling
```
### Motion manipulation

The motion object can be manipulated in both modular and matrix forms. Here, we translate the object to a fixed global position `[1, 1, 1]` and select a time slice from frame `20` to frame `30`.
```
from fairmotion.ops import motion as motion_ops

translated_motion = motion_ops.translate(motion, np.array([1, 1, 1]))
sliced_motion = motion_ops.cut(translated_motion, 10, 20)
```
We can perform the same operations in the matrix representation of motion.
```
from fairmotion.core.motion import Motion

# motion_matrix has shape (num_frames, num_joints, 4, 4) where 4x4 is transformation matrix
motion_matrix = motion.to_matrix()

translation_matrix = np.zeros((4, 4))
translation_matrix[3, :3] = np.array([1, 1, 1])

translated_motion_matrix = motion_matrix + translation_matrix
sliced_motion_matrix = translated_motion_matrix[10:20]
sliced_motion = Motion.from_matrix(sliced_motion_matrix, motion.skel)
```
### Data saving

We can save the manipulated motion object back into the bvh file format for us to visualize the result.
```
NEW_BVH_FILENAME = "PATH_TO_NEW_BVH_FILE"
bvh.save(sliced_motion, NEW_BVH_FILENAME)
```
### Visualization

We visualize the results using the `bvh_visualizer` tool.
```
$ python fairmotion/viz/bvh_visualizer.py --bvh-files $NEW_BVH_FILENAME
```

## Tasks
The `tasks` module showcases practical usage of fairmotion modules as building blocks in developing projects.

- [Motion Prediction](https://github.com/facebookresearch/fairmotion/tree/master/fairmotion/tasks/motion_prediction)
- [Motion Graph](https://github.com/facebookresearch/fairmotion/tree/master/fairmotion/tasks/motion_graph)
- [Clustering of motion capture dataset](https://github.com/facebookresearch/fairmotion/tree/master/fairmotion/tasks/clustering)
- [Changepoint Detection](https://github.com/facebookresearch/fairmotion/tree/master/fairmotion/tasks/changepoint_detection)

fairmotion has been used in some form in the following works:

* Jungdam Won, Deepak Gopinath, and Jessica Hodgins. “A Scalable Approach to Control Diverse Behaviors for Physically Simulated Characters” to be presented at SIGGRAPH 2020 [[Project page with code and paper](https://research.fb.com/publications/a-scalable-approach-to-control-diverse-behaviors-for-physically-simulated-characters/)]
* Tanmay Shankar, and Abhinav Gupta. "Learning Robot Skills with Temporal Variational Inference." ICML 2020
* Jungdam Won, and Jehee Lee. "Learning body shape variation in physics-based characters." ACM Transactions on Graphics (TOG) 2019

## Citation
If you find fairmotion useful in your research, please cite our repository using the following BibTeX entry.
```
@Misc{gopinath2020fairmotion,
  author =       {Gopinath, Deepak and Won, Jungdam},
  title =        {fairmotion - Tools to load, process and visualize motion capture data},
  howpublished = {Github},
  year =         {2020},
  url =          {https://github.com/facebookresearch/fairmotion}
}
```
## License
fairmotion is released under the [BSD-3-Clause License](https://github.com/facebookresearch/fairmotion/blob/master/LICENSE).
