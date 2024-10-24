# QNPy Latte
### Latent ATTEntive Neural Processes for Quasar Light Curves with parametric recovery

#### By Aman Nadimpalli Raju, Andjelka Kovacevic, Marina Pavlovic, Dragana Ilic, Iva Cvorovic-Hajdinjak (SER-SAG-S1 team)

![QNPy Latte Logo](https://github.com/user-attachments/assets/b408d1fa-ac51-4801-ab2a-514f5721d195) 
###### Logo Generated by Google Gemini

![Nonsensical_AI Comic About QNPy](https://github.com/user-attachments/assets/467a2c61-4c73-47ce-bbf9-a7e4b291e6c1)
###### Comic generated through ChatGPT

## Introduction
Many of the secrets of quasars lie hidden within their variations. Analysis of quasar light curves can unlock some of these secrets, giving a holistic view of the region around the central black hole engine, as well as properties of the black hole. However, studying quasar variability itself is a challenge. The stochastic nature of the variations, coupled with the need for short cadence observations over a long period, make analysis difficult. 

The LSST (Legacy Survey of Space and Time) will provide a plethora of quasar light curves, with short cadence observations throughout its 10 year run. However, there will be seasonal gaps in the observations, to keep up with the LSST goal of surveying the entire Chilean night sky. In order to utilize the LSST data for variational analysis, it is imperitive to model this data to fill in these gaps. To process the amount of data coming in, any such model should be non-parametric, computational inexpensive, data-driven, and capable of interpolation. We also would like to draw insights from the hidden layers of these models. Thus, we utilize Latent Attentive Neural Processes (AttnLNPs). 

As part of the Serbian In-Kind software contribution to the LSST team (SER-SAG-S1), we present the QNPy Latte package to model and determine parameters (especially the transfer function) of quasar light curves. This work has been done by the guest student Aman N. Raju as part of his masters thesis and presented at the following:

Parts of this thesis have been presented as:

• [“A Deep Learning Approach for Understanding Quasar Light Curves in the
Legacy Survey of Space and Time"](https://simpozijum.math.rs/s2023/Aman_Raju_Maths_Symposium_Deep_Learning_LSST.pdf) - Symposium Mathematics and Application, Faculty of Mathematics, University of Belgrade, 2023

• “QNPy and QhX Workshop” - LSST AGN Science Collaboration Quarterly
Report, March 2024

• [“Attentive Latent Neural Processes for modeling Quasar Variability in the
LSST”](https://www.youtube.com/watch?v=qSy-ZKhry5E) - LSST TVS Colloquium, 21st May 2024

• [“Pay Attention: Neural Processes with Latent Spaces and Attention to model
Quasar Light Curves for the LSST”](https://astro.matf.bg.ac.rs/beta/lat/stud/workshop/13SAR.pdf) - XIII SAW, Astronomical Society “Ruđer
Bošković", 18th May 2024

• [“UPGRADING QNPY: MODELLING QUASAR LIGHT CURVES IN LARGE
SURVEYS”](https://www.aob.rs/en/publishing-en/doi/conferences-papers-and-proceedings/425-2024-015-upgrading-qnpy-modelling-quasar-light-curves-in-large-surveys) - VI Conference on Active Galactic Nuclei and Gravitational Lensing, Zlatibor Mt., Serbia, June 02-06, 2024

• “Connecting the Dots: Attentive Latent Neural Processes” - 2nd MASS Summer School, Nice, France, July 2024 (First Prize 180s Thesis Talk)

• [“LSST SER-SAG-S1: upgrade of QNPy package”](https://youtu.be/FltMIHpNscQ?t=8600) - Catching supermassive
black holes with Rubin-LSST: Towards novel insights and discoveries into AGN
science, Torino, Italy, July 22nd-25th 2024

Furthermore, this repository will be used in the paper, Raju et. al. 2024 (in prep).

## Latent Attentive Neural Processes
At SER-SAG-S1, we have used Conditonal Neural Processes (CNPs) to model quasar light curves (see [QNPy](https://github.com/kittytheastronaut/QNPy-0.0.2/tree/main)). However, CNPs suffer from issues with underfitting and produce poor samples. Thus, the addition of attentive mechanisms and a latent path allow for an upgraded model. 

The AttnLNP has two paths. One path is the deterministic path that operates similarly to CNPs to create a representation of the light curve. The input data (observation time and magnitudes) are encoded using a multilayer perceptron (MLP) to create a representation for each point. With the addition of self-attention, the points that are more important are weighted more highly in the representation. Then, the representations are used with the input and target times to create a attention weighted representation for each target time, allowing for flexible representations that can adapt to different datapoints. 

The latent path operates similarly. However, the representations are simply averaged to create a global representation. This representation is passed through another MLP to generate a multidimensional gaussian latent space that can be sampled from. The latent space samples are combined with the determinstic representation to generate a combined representation for each target point in the light curve. These combined representations are passed through an MLP decoder to generate a predicted magnitude at each target time. 

![image](https://github.com/user-attachments/assets/50c1bdbd-30b3-44e5-a461-876062d0e486)

Image and original implementation from (https://arxiv.org/abs/1901.05761)[Kim et al. 2019]

In our model, we also include the possibility to encode the input and target times with an LSTM RNN layer in order to better reconstruct the quasar light curves as detailed in (https://arxiv.org/abs/1910.09323)[Qin et. al. 2019]


## Self Organizing Maps
Just like [QNPy](https://github.com/kittytheastronaut/QNPy-0.0.2/tree/main), we incorporate Self Organizing Maps (SOMs) (see  into our models to cluster the light curves before modelling. This serves a two-fold purpose. The dataset can be partitioned for quicker modelling and the resulting clusters are more balanced topologically than unclustered light curves. SOMs are an unsupervised model that can be customized to generate many clusters and easily updated to include new data without having to retrain the entire model.  

SOMs are composed of a grid of nodes that assign a best matching unit (BMU) to each of the input points (in this case, light curves). Then it updates the BMU to move closer to the datapoint at a rate specified by the learning rate and the neighborhood function that controls the updating of the nodes around the BMU. Thus, the entire SOM node is trained on the input data and every datapoint can have a BMU. These BMUs can be clusters themselves or they can be grouped together by density analysis to generate larger but fewer clusters. 

Our SOMs are based on the [minisom](https://github.com/JustGlowing/minisom) package and follow the same implementation as [QNPy](https://github.com/kittytheastronaut/QNPy-0.0.2/tree/main). Through the use of SOMs, AttnLNPs and RNNs, we hope to create a multimodal approach to analyzing light curves.

## Prediction of Parameters
Similar to [Park et. al. 2021](https://inspirehep.net/literature/1867003), we use the hidden representation of the light curve to infer parameters. We specifically include the ability to reconstructe the transfer function of the quasar, informing us of the properties of the region around the central black hole engine. We do this through attaching an MLP to the model either during training, or on an already trained model and train the model on theoretical transfer functions. 

Thus, we are able to not just perform reconstruction of the light curve itself, but also analyze key properties about the system driving the quasar in a purely data-driven manner.

## Installation

It is recommended to use a virtual enviornment with Anaconda to install QNPy_Latte. Please install one in Python 3.9 as:
```
conda create --name my_env -c anaconda python=3.9
```

Activate this enviornment and then install QNPy_Latte by:
```
pip install QNPy_Latte
```

We recommend creating a new anaconda enviornment first and then running the above command.

## Tutorial Notebooks and Scripts
Along with the standard package, we provide tutorial notebooks and scripts (COMING SOON) to help in analysis. We provide three different notebook folders here. They include:

1) **Clustering_Tutorial_Notebooks**
2) **Modelling_Tutorial_Notebooks**
3) **Modelling_Tutorial_Notebooks_w_Params**
4) **Modelling_Scripts** (COMING SOON)

It is important that your light curves are in the form of csv files with three columns, mjd, mag and magerr. Store all of them in a single folder, or seperate the folders by band under one main directory. Example, Light_Curves\name.csv or Light_Curves\u_band\name.csv

### Clustering Tutorial Notebooks
These notebooks walk you through creating clusters through the SOM. **Basic_Example_One_Band** demonstrates how to cluster a light curves in a single band. **Advanced_Visualization_One_Band** brings in more visualizations. **Advanced_Clustering_One_Band** introduces gradient-based clustering for more meaningful clusters from large SOMs. Finally, **Multiband_Clustering** (as the name suggests) implements multiband clusters.

### Modelling Tutorial Notebooks
These notebooks show you how to use the Latte model to reconstruct light curves. These notebooks are recommended for use in most real cases, where information on the parameters and the transfer function are not included. The **Preprocessing** notebook makes your data suited for use with the Latte model. It pads, transforms to -2,2 and arranges the data into train, test, and validation data folders. Then **Training_Model** actually trains the model on the training data, with validation data to help choose the best epoch. The metrics associated with training are also saved as readable csv files. Finally, **Modelling** uses the trained model to provide both visualizations of and the actual data associated with reconstructed light curves.

### Modelling Tutorial Notebooks w Params
These notebooks work and are named similar to before. The only difference is that for this folder, you need to provide transfer functions and a dataframe containing parameters associated with the generation of the light curve. The final result will include the comparison of the reconstructed transfer function and the parameters as well.

### Modelling Scripts (COMING SOON)
Don't have the time to go through the notebook or want to run your analysis on a supercomputer? Use one of the scripts that we provide. We provide scripts that are capable of clustering and then training seperate models on each cluster, running through the entire process on real light curves and a script to generate both reconstructions and recovered transfer functions and parameters.
