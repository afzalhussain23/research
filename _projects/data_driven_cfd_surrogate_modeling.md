---
layout: page
title: Data-Driven CFD Surrogate Modeling Using Deep Learning
description: A data-driven surrogate model using ConvLSTM to predict supersonic flow over a forward-facing step from OpenFOAM-generated training data.
img: assets/img/projects/data-driven-cfd-surrogate-modeling/geometry.png
importance: 1
category: research
toc:
  sidebar: left
---

[View on GitHub →](https://github.com/afzalhussain23/Data-Driven-CFD-Surrogate-Modeling-Using-Deep-Learning)

### Preface

This January, during the starting of the 7th semester I completed Andrew Ng’s [Deep Learning Specialization](https://www.coursera.org/specializations/deep-learning) from Coursera. This work was motivated by the potential application of deep learning algorithms in mechanical engineering. The core hypothesis was that deep learning models could predict fluid simulation outcomes, which motivated this research.

This report documents the complete pipeline, from simulation generation to deep learning model development and evaluation.

### Case Setup

For this research, I have used [OpenFOAM](https://openfoam.org/), a C++ open source implementation for pre-processing, solving and post-processing CFD simulation. The reason behind choosing OpenFOAM because of its flexibility and automation. Here [supersonic flow over a forward-facing step](https://www.openfoam.com/documentation/tutorial-guide/3-compressible-flow/3.2-supersonic-flow-over-a-forward-facing-step) is investigated. The problem description involves a flow of Mach 3 at an inlet to a rectangular geometry with a step near the inlet region that generates shock waves. The geometry is shown below:

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/data-driven-cfd-surrogate-modeling/geometry.png" title="Forward-facing step geometry" class="img-fluid rounded z-depth-1" %}
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
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/C-LSTM.png" title="ConvLSTM key equations" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ConvLSTM key equations (Shi et al., 2015). Standard LSTM gate operations with state-to-state and input-to-state matrix products replaced by convolutions (∗); ◦ denotes the Hadamard product.
</div>

### Deep Learning Model

As fluid simulation is time-dependent I have used three [TimeDistributed](https://keras.io/api/layers/recurrent_layers/time_distributed/) [Conv2D](https://keras.io/api/layers/convolution_layers/convolution2d/) followed by a TimeDistributed [MaxPooling2D](https://keras.io/api/layers/pooling_layers/max_pooling2d/). After that [ConvLSTM2D](https://keras.io/api/layers/recurrent_layers/conv_lstm2d/) has been performed. This model initially exhibited slow convergence, so a ResNet-style architecture was incorporated to accelerate training. Using ResNet has significantly improved the model performance and accuracy. The whole model is shown below:

<div class="row justify-content-sm-center">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/my_model.png" title="Model architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    End-to-end network architecture: three TimeDistributed Conv2D + MaxPooling2D blocks encode the input, followed by three stacked ConvLSTM2D blocks with TimeDistributed Conv2DTranspose decoders and ResNet-style `Add` skip connections, ending in a Conv3D output.
</div>

### Results and Discussion

The model is evaluated on geometries that are previously unknown to the model. The obtained results are realistic, competitive in accuracy; it successfully shows discontinuities in shock waves, emanating from ahead of the base of the step, and also captures the time-dependent development of the shock-waves. While this work has been performed on one problem specification, it illustrates the viability of data-driven approaches in computational fluid dynamics. Below a comparison between actual simulation and the predicted one by the model is shown for velocity, pressure and temperature at t = 1, 3 & 5 seconds.

##### Velocity

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/U/1s.png" title="Velocity at t = 1s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/U/3s.png" title="Velocity at t = 3s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/U/5s.png" title="Velocity at t = 5s" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Velocity field at t = 1, 3, and 5 seconds (left to right) on a previously unseen geometry. Each panel pairs the actual simulation (left half) with the model prediction (right half).
</div>

##### Pressure

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/p/1s.png" title="Pressure at t = 1s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/p/3s.png" title="Pressure at t = 3s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/p/5s.png" title="Pressure at t = 5s" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Pressure field at t = 1, 3, and 5 seconds (left to right) on a previously unseen geometry. Each panel pairs the actual simulation (left half) with the model prediction (right half).
</div>

##### Temperature

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/T/1s.png" title="Temperature at t = 1s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/T/3s.png" title="Temperature at t = 3s" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/data-driven-cfd-surrogate-modeling/plots/T/5s.png" title="Temperature at t = 5s" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Temperature field at t = 1, 3, and 5 seconds (left to right) on a previously unseen geometry. Each panel pairs the actual simulation (left half) with the model prediction (right half).
</div>

### Conclusion

In the past few years, deep learning has exhibited unprecedented competency and efficiency in image classification, speech recognition, weather forecasting, self-driving cars and many other domains, due to the large availability of data. In this work, we propose an end-to-end deep learning model operating on spatiotemporal flow field sequences, where both the input and output are grid-based representations of the domain, having convolutional structures in both the input-to-state and state-to-state transitions. The proposed model is trained and evaluated on supersonic flow over a forward-facing step; the obtained results are realistic, competitive in accuracy while illustrating the viability of data-driven approaches in computational fluid dynamics.

> ##### TAKEAWAY
>
> An image-based ConvLSTM, trained end-to-end on one CFD problem (supersonic flow over a forward-facing step), produces realistic, accurate predictions on geometries it has never seen — evidence that data-driven approaches are viable in computational fluid dynamics.
> {: .block-tip }

### Related Research

[Turbulence Modeling Gateway](https://tmbwg.github.io/turbmodels/) \
[Maziar Raissi's Research](https://github.com/maziarraissi) \
[Data-driven Fluid Simulations using Regression Forests](https://cgl.ethz.ch/Downloads/Publications/Papers/2015/Jeo15a/Jeo15a.pdf) \
[Convolutional Neural Networks for Steady Flow Approximation](https://www.research.autodesk.com/publications/convolutional-neural-networks-for-steady-flow-approximation/) \
[Application of Convolutional Neural Network to Predict Airfoil Lift Coefficient](https://arxiv.org/abs/1712.10082)
