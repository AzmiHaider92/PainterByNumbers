# Painter By Numbers

Hello world!

This is our solution to the Kaggle competition PainterByNumbers: https://www.kaggle.com/c/painter-by-numbers

The goal of the competition is to build a network that learns artists' painting style. 

The network aims to build an organized feature space where paintings are represented by feature vectors (of size 2048); Regulating of the feature space means insuring that paintings of the same artist are clustered together. Paintings which share an "artist style" are closer to each other than to any different artist's paintings (closer = the euclidean distance between their feature vector representation is smaller).     
Example: left and middle paintings are Van Gogh's, while the right painting is Picasso's.  
In feature space the distance between Van Gogh's paintings is smaller than to the Picasso painting.  


<p align="center">
  <img src="photos/n-3861-00-000045-hd.jpg" height="250" />
  <img src="photos/self-portrait.jpg" height="250"/> 
  <img src="photos/Jeun-Fille-Endormie-by-Picasso.jpg" height="250"/>
</p>
  


# Approach: Siamese CNN network with triplet loss
Siamese CNN consists of three Convolutional Neural Networks where **weights are shared** between the CNNs.  
First input goes through the CNN to produce a feature vector, then the second and the third go throught the **same** CNN.    
The fist image -we term Anchor- is a painting which belongs to artist1.     
The second image -we term Positive- is a **different** painting of the **same** artist1.  
The third image -we term Negative- is a painting of a **different** artist2.    
**Note: The two artists and paintings in each step are all randomly chosen.**  

The triplet loss (explained in loss section) is then calculated on the three feature vectors produced and gradients are calculated on the shared weights.  
The final result of the network is an organized feature space: each painting image is mapped (through the network) to a feature vector Dx1; feature vectors of paintings of same artist are clustered together and are pushed away from clusters of different artists.  

<p align="center">
  <img src="photos/0_SszXblCjQOPiLhjZ.png" width="600"/>
</p>

  
  
**Contrastive network**: We've also exprimented with two CNNs (instead of three, still with **shared weight** between the two CNNs). For this, a contrastive loss was used. In this network, two paintings and a label are given; The goal is to push their feature vectors closer if the label=0 (paintings are from the same artist) or push their feature vectors away from each other incase the label=1 (paintings are from different artists).   
In each step, we first randomly choose label=(0,1) and then randomly choose paintings based on the label (if label=0 we randomly choose an artist, then randomly choose two of his paintings. If label=1, we randomly choose 2 artits and randomly choose a painting of each of them). The label is saved with the pair (see contrastive loss below).  

  
  

# The architechture
The input image is of size 256x256x3.  
The shared CNN consists of 5 convolutional blocks - the sizes of the images after each block is shown below. Inside each block there are 2-3 conv layers (preserving the size) ; conv2d layers are followed by a BatchNorm and a Relu activation. Each block is followed by a maxpool(2).  
**The output feature vector is of size 2048**.

<p align="center">
  <img src="photos/Picture3.png" width="1000"/>
</p>
(the architecture figure was drawn with this online tool: http://alexlenail.me/NN-SVG/LeNet.html)

  
  
  
# The loss:

Cosider 3 inputs - from left to right: Anchor, Positive and Negative. Where Anchor and Positive are paintings of the same artist, and negative is a painting of a different artist.  
<p align="center">
  <img src="photos/anchor.png" width="200"/>
  <img src="photos/positive.png" width="200"/>
  <img src="photos/negative.png" width="200"/>
</p>

**Triplet loss:**  
The aim of the triplet loss is to push feature vectors of Anchor and Positive paintings closer to each other pushing them away from the feature vector of the Negative painting. (A default margin=2).

<p align="center">
  <img src="photos/tripletloss.jpg" width="600"/>
</p>

The loss aims to be zero (Ideally); to achieve this: D(A,P) - D(A,N) needs to become smaller than -margin (this forces the term is the max to be zero). Meaning:    
* D(A,P) aims to become small (this is the distance between Anchor and Positive).  
* D(A,N) aims to become large (this is the distance between Anchor and Negative).  
* Both distances keep updating until they fulfill the term: D(A,P) - D(A,N) < -margin.
The margin forces clusters of painters to be far away from each other to a certain extent. The larger the margin the more the network forces the clusters to be farther. 


**Contrastive loss:**  
As mentioned above, We've expiremented with a two CNN arhictecture where only two paintings and a label Y are handled.  
<p align="center">
  <img src="photos/Contrastiveloss.jpg" width="700"/>
</p>

The contrastive loss is similar to the triplet, but it only does one of the other at each given time. It either:  
pushes the two feature vectors closer if the label Y=0:    
* Same painter
* The right side of the equation is zero; left side is the distance between the feature vectors which we aim to minimize in order to minimize the loss.   
  
pushes the two feature vectors farther if the label Y=1:
* different painter  
* the left side of the equation is zero; the right side pushes the distance to be larger than the given margin in order to zero the term inside the max. Hence, to minimize the loss, we need to maximize the distance between the feature vectors. 


**Notes:**  
**D, Dw = Euclidean distance.**  
**We think the triplet approach is much stronger as it does what the contrastive approach does and more.**   
**Nevertheless, the two approaches are implemented and can be switched with a simple flag *pair_triplet* ( False=contrastive ; True=Triplet).**  


# Data cleaning and preprocessing
First, We've divided the paintings to classes based on artists. We eliminated artists who had only one painting, this was done after noticing that some classes (artists) are not even artists (they had paintings' names) - it is clearly a mistake in data collection.  
After cleaning, the dataset has 1701 artists, each with at least two paintings.  
Second, the dataset artists were divided to **training** (80% - 1352 painters) and **test** (20% - 349 painters).        
A fraction (10%) from training **paintings** was saved for **validation**.  
**The test set has only artists not seen in training (not one painting of theirs)!**   
**The validation set has paintings not seen in training, but the artists who painted them are in the training.**  

The number of paintings per artist are different and shown below (ordered):  
<p align="center">
  <img src="photos/NumberOfPaintingsPerArtist.jpg" width="800"/>
</p>

  
Number of paintings in training set: 73183  
Number of paintings in validation set: 7083  
Number of paintings in the test set: 22237

**A triplet sample from training:**  
<p align="center">
  <img src="photos/sample.jpg" width="800"/>
</p>


# HyperParameters

**Optimizer = Adam with a learning rate=0.001**  
**Epochs = 60**  
**miniBatch size = 4**  
**Scheduler = MultiStepLR(optimizer, milestones=[11,25,40], gamma=0.1)**  

**Note: the model was trained on a machine with 1 GPU.**  
**Number of workers = 8**  

Images were center cropped to size 256x256.  
Random transformations used to augment:  
-RandomAffine (degrees=(0, 0), translate=(0.2, 0.2), scale=(1.2, 1.2))  
-RandomVerticalFlip (p=0.5)  
-RandomRotation (degrees=(0, 180))  
-RandomHorizontalFlip (p=0.5)  



# Training Results  

**Reminder: the validation set consists of paintings whose artists are in the training but the paintings themselves are not.**  
**Training and validation:**   

Training accuracy =~ 90%.  

<p align="center">
  <img src="photos/batch4acc.jpg" width="600"/>
  <img src="photos/batch4loss.jpg" width="600"/>
</p>

# Test Results

**Reminder: the test set consists of paintings whose artists are not in the training.** 
the test set is a proof that the model **extrapolates** new painting styles!  

**Test accuracy - 0.8928057553956834** (in the same way the training accuracy was calculated)

We add the ROC - tested with multiple thresholds from 0 to 10 (jumps of 0.1).

<p align="center">
  <img src="photos/ROC.png" width="600"/>
</p>


**visualizing a couple of test samples:**  
(the distance between the anchor and the image is in the title - distance between anchor and itself is 0)  
<p align="center">
  <img src="photos/testSamples.jpg" width="800"/>
</p>

Admittedly, the misclassified test samples are really confusing even for the human eye:  
<p align="center">
  <img src="photos/testSamplesHardMis.jpg" width="800"/>
</p>



# Conclusion
* Based on the results of the experiment, we conclude that CNN networks are great for exctracting features from visual images (artwork in our case).
* Metric learning (like siamese networks) enables extrapolation to unseen classes (new painters in our case - not even one sample (painting) of these classes (painters) are seen during the training process). This is largly due to the fact that the network is not being fit to specific classes, but rather learning which features are important (and should be considered) when comparing paintings of two painters in order to distinguish an artist's unique painting style.
* We implemented two approaches to deal with building the feature space for the paintings. Both of them can be used in our code by turning a simple flag in the code: **pair_triplet = True/False ( False = two nets, contrastive loss ; True = Three nets, Triplet loss)**.



# Resources
* https://www.kaggle.com/c/painter-by-numbers  
* https://towardsdatascience.com/a-friendly-introduction-to-siamese-networks-85ab17522942  
* https://towardsdatascience.com/how-to-train-your-siamese-neural-network-4c6da3259463  
* https://github.com/AceEviliano/Siamese-network-on-MNIST-PyTorch/blob/master/Siamese/Siamese%20Net.ipynb  
* https://www.kaggle.com/code/hirotaka0122/triplet-loss-with-pytorch/notebook  


# You're welcome to checkout my own art gallery:   
**https://www.azmihaider.com/art** 

