# PainterByNumber

Hello all,

This is my solution to the Kaggle competition PainterByNumbers: https://www.kaggle.com/c/painter-by-numbers

The goal of the competition is to build a network that learns artists' painting style. 

The network is built to generate feature vectors for painting in a way that paintings of same artists' feature vectors are closer to each other than feature vectors of paintings of different artists.  
Example: left and middle paintings are Van Gogh's, while the right painting is Picasso's.   
<p float="left">
  <img src="photos/n-3861-00-000045-hd.jpg" height="300" />
  <img src="photos/self-portrait.jpg" height="300"/> 
  <img src="photos/Jeun-Fille-Endormie-by-Picasso.jpg" height="300"/>
</p>
  

**Approach: Siamese CNN network.**


<p float="left">
  <img src="0_SszXblCjQOPiLhjZ.png" width="600" />
</p>
0_SszXblCjQOPiLhjZ.png
