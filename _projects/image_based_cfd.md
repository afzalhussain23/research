---
layout: page
title: Image-Based CFD Using Deep Learning
description: A ConvLSTM model that predicts supersonic flow over a forward-facing step from OpenFOAM-generated training data.
img: assets/img/projects/image-based-cfd/geometry.png
importance: 1
category: research
toc:
  sidebar: left
---

[View on GitHub →](https://github.com/afzalhussain23/Image-Based-CFD-Using-Deep-Learning)

### Preface

This January, during the starting of the 7th semester I completed Andrew Ng’s [Deep Learning Specialization](https://www.coursera.org/specializations/deep-learning) from Coursera. I was really fascinated by how I can use different deep learning algorithms so that it can be useful in mechanical engineering. Then suddenly an idea came into my mind, deep learning models can be used to predict fluid simulation and later I started doing research on this.

This blog is about the whole procedure that I have gone through, from generating fluid simulation to deep learning model everything is explained here.

### Case Setup

For this research, I have used [OpenFOAM](https://en.wikipedia.org/wiki/OpenFOAM), a C++ open source implementation for pre-processing, solving and post-processing CFD simulation. The reason behind choosing OpenFOAM because of its flexibility and automatization. Here [supersonic flow over a forward-facing step](https://www.openfoam.com/documentation/tutorial-guide/tutorialse6.php) is investigated. The problem description involves a flow of Mach 3 at an inlet to a rectangular geometry with a step near the inlet region that generates shock waves. The geometry is shown below:

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/image-based-cfd/geometry.png" title="Forward-facing step geometry" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Forward-facing step geometry. A Mach 3 flow enters from the left into a rectangular domain with a step near the inlet, generating shock waves.
</div>

### Generating simulation

> ##### HEADS UP
>
> This was the most laborious part of the project — generating enough varied simulations to train on takes serious time and disk.
> {: .block-warning }

As deep learning requires plenty of data, I needed about thousands of simulations of varying geometries so that it can predict simulation of unknown geometries. For this purpose, I changed the position of step from near the inlet region to the outlet i.e. 0.1 < x < 2.9, ranging it height 0.1 < y < 0.4. This is done by a python script where each step are described with comments. Making the dataset contains the following steps:

1. Make 1500 random coordinates within some constraints.
2. Remove previous simulation file (if it exists).
3. Copy the OpenFOAM _forwardStep_ directory.
4. Remove _blockMeshDict_ file from system directory.
5. Execute `gen_blockMeshDict.py` to write _blockMeshDict_ and _cellInformation_ file.
   _cellInformation_ consists cell number of three rectangle (x_cell \* y_cell) (2D simulation).
6. Move _blockMeshDict_ file to system directory
7. Move _cellInformation_ file to home directory
8. Now execute `sim_cmd` from terminal.
9. It uses _sonicFoam_ to run simulation.
10. And _foamToVTK_ to convert the simulation result into .vtk file.

After this almost 168GB simulation data has been generated. But all this data is not necessary for training, I extract only velocity at x & y direction, pressure, and temperature of each cell. dl_data_generation is used to do this tasks.

> ##### DATASET SCALE
>
> The full OpenFOAM run produced ~168 GB of raw simulation output. Only velocity (`U_x`, `U_y`), pressure, and temperature per cell are extracted for training.
> {: .block-tip }

### Convolutional LSTM

For long-range dependencies in time-series data, [LSTM](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) has been using for a longer period of time, that has proven stable and powerful. But typical LSTM implementation deals with 1-D series data only, as fluid simulation involves with spatial data, I need to use a variant of LSTM, proposed by [X Shi et al.](https://arxiv.org/abs/1506.04214), where state-to-state and input-to-state transitions are replaced by convolution operation. The key equations are shown below, where ‘∗’ denotes the [convolution operator](https://en.wikipedia.org/wiki/Convolution) and ‘◦’ denotes the [Hadamard product](<https://en.wikipedia.org/wiki/Hadamard_product_(matrices)>):

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/C-LSTM.png" title="ConvLSTM key equations" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ConvLSTM key equations (Shi et al., 2015). Standard LSTM gate operations with state-to-state and input-to-state matrix products replaced by convolutions (∗); ◦ denotes the Hadamard product.
</div>

### Deep Learning Model

As fluid simulation is time-depended I have used three [TimeDistributed](https://keras.io/layers/wrappers/) [Conv2D](https://keras.io/layers/convolutional/#conv2d) followed by a TimeDistributed [MaxPolling2D](https://keras.io/layers/pooling/#maxpooling2d). After that [ConvLSTM2D](https://keras.io/layers/recurrent/#convlstm2d) has been performed. This model is initially taking a lot of time to converge, so I have used [ResNet](https://arxiv.org/abs/1512.03385) concept here to improve training time. Using ResNet has significantly improved the model performance and accuracy. The whole model is shown below:

<div class="row justify-content-sm-center">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/my_model.png" title="Model architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    End-to-end network architecture: three TimeDistributed Conv2D + MaxPooling2D blocks encode the input, followed by three stacked ConvLSTM2D blocks with TimeDistributed Conv2DTranspose decoders and ResNet-style `Add` skip connections, ending in a Conv3D output.
</div>

### Results so far

The model is evaluated on geometries that are previously unknown to the model. The obtained results are realistic, competitive in accuracy; it successfully shows discontinuities in shock waves, emanating from ahead of the base of the step, and also captures the time-dependent development of the shock-waves. While this work has been performed on one problem specification, it illustrates the viability of data-driven approaches in computational fluid dynamics. Below a comparison between actual simulation and the predicted one by the model is shown for velocity, pressure and temperature at t = 1, 3 & 5 seconds.

##### Velocity

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/U/1s.png" title="Velocity at t = 1s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/U/3s.png" title="Velocity at t = 3s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/U/5s.png" title="Velocity at t = 5s" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Velocity field at t = 1, 3, and 5 seconds (left to right) on a previously unseen geometry. Each panel pairs the actual simulation (left half) with the model prediction (right half).
</div>

##### Pressure

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/p/1s.png" title="Pressure at t = 1s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/p/3s.png" title="Pressure at t = 3s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/p/5s.png" title="Pressure at t = 5s" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Pressure field at t = 1, 3, and 5 seconds (left to right) on a previously unseen geometry. Each panel pairs the actual simulation (left half) with the model prediction (right half).
</div>

##### Temperature

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/T/1s.png" title="Temperature at t = 1s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/T/3s.png" title="Temperature at t = 3s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/image-based-cfd/plots/T/5s.png" title="Temperature at t = 5s" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Temperature field at t = 1, 3, and 5 seconds (left to right) on a previously unseen geometry. Each panel pairs the actual simulation (left half) with the model prediction (right half).
</div>

### Conclusion

In the past few years, deep learning has exhibited unprecedented competency and efficiency in image classification, speech recognition, weather forecasting, self-driving cars and many other domains, due to the large availability of data. In this work, we propose an image-based end-to-end deep learning model where both the input and output are spatiotemporal sequences of images, having convolutional structures in both the input-to-state and state-to-state transitions. The proposed model is trained and evaluated on supersonic flow over a forward-facing step; the obtained results are realistic, competitive in accuracy while illustrating the viability of data-driven approaches in computational fluid dynamics.

> ##### TAKEAWAY
>
> An image-based ConvLSTM, trained end-to-end on one CFD problem (supersonic flow over a forward-facing step), produces realistic, accurate predictions on geometries it has never seen — evidence that data-driven approaches are viable in computational fluid dynamics.
> {: .block-tip }

### Related Research

[Data-Driven Turbulence Modeling](https://www.aoe.vt.edu/people/faculty/xiaoheng/personal-page/research/data.html) \
[Turbulence Modeling Gateway](http://turbgate.engin.umich.edu/publications/) \
[Maziar Raissi's Research](http://www.dam.brown.edu/people/mraissi/research/) \
[Data-driven Fluid Simulations using Regression Forests](https://www.inf.ethz.ch/personal/ladickyl/fluid_sigasia15.pdf) \
[Convolutional Neural Networks for Steady Flow Approximation](https://autodeskresearch.com/sites/default/files/ADSK-KDD2016.pdf) \
[Application of Convolutional Neural Network to Predict Airfoil Lift Coefficient](https://pdfs.semanticscholar.org/ef39/ed630a8fca2e33fb2253e2a9faf4e3ad391d.pdf)
