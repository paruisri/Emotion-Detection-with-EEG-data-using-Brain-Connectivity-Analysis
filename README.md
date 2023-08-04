# Emotion-Detection-with-EEG-data-using-Brain-Connectivity-Analysis
This research presents the development of an accurate emotion detection algorithm using functional connectivity. We proposed a multi domain feature subset by combining connectivity domain features with Empirical Mode Domain features. 
Data format -	40 x 40 x 8064	trial x channel x data
data sampling rate = 128Hz
First calculation was done on a single trial and that contained information of 32 channels (32×8064).
Then from every channel, 3s baseline was removed, and after that the input data was filtered using IIR filters to four frequency bands: theta (4–8 Hz), alpha (8–12 Hz), beta (12–30 Hz) and gamma (30–48 Hz). 

Output data after this step = 32×7680

PSI, PLV values were calculated between two channels. So ultimately, we get 32×32 matrix as the adjacency matrix. Then from this matrix graph theoretic features were extracted.
PSI measurement: 
For calculating PSI values, I have used the function data2psi. The PSI resulted a directed graph.
function [psi, stdpsi, psisum, stdpsisum] = data2psi(data,segleng,epleng,freqbins);

data:  NxM matrix for N data points in M channels;// For our case 7680×32
% segleng: segment length in bins, (frequency resolution is determined by it) 
% epleng:  length of epochs in bins. This is needed only to estimate the 
%          standard deviation of PSI. Setting epleng=[] avoids estimation 
%          of the standard deviation (which is faster). 
% freqbins:  KxQ matrix. Each row contains the frequencies (in bins), over
%            which  PSI is calculated. (freqbins includes the last frequency
%            (f+delta f), i.e. the band F in the paper is given for the 
%             k.th row as F=freqbins(k,1:end-1).  
%             By setting freqbins=[] PSI is calculated across all frequencies (wide band). 
% 
% Output: 
% psi:  non-normalized PSI values. For M channels PSI is either an MxM matrix (if freqbins has one or zero rows) 
%                   or an MxMxK tensor if freqbins has K rows (with K>1).
%                   psi(i,j) is the (non-normalized) flow from channel i to
%                   channel j, (e.g., channel i is the sender if psi(i,j) is
%                   positive.) 
% stdpsi: estimated standard deviation for PSI. 
%         PSI in the paper is given by psi./(stdpsi+eps) (eps is included
%         to avoid 0/0 for the diagonal elements) 
% psisum =sum(psi,2) is the net flux for each channel. 
% stdpsisum  is the estimated standard deviation of psisum. (stdpsisum cannot be 
%             calculated from psi and stdpsi - therefore the extra output) 
PLV measurement: 
For PLV measurement the method described in paper [1] has been used.
SL Measurement:
Connectivity features calculation:
In this case BCT toolbox have been used. The connectivity features that were calculated from the weighted adjacency matrices are CLUSTERING COEFFICIENT, ASSORTATIVITY, GLOBAL EFFICIENCY, LOCAL EFFICIENCY, NODE BETWEEN CENTRALITY, and AVERAGE NEIGHBORHOOD DEGREE.
1. CLUSTERING COEFFICIENT
C = clustering_coef_wd(W);
The weighted clustering coefficient is the average "intensity"
    (geometric mean) of all triangles associated with each node.
 
    Input:      W,      weighted directed connection matrix
                        (all weights must be between 0 and 1)
 
    Output:     C,      clustering coefficient vector

2. ASSORTATIVITY

r = assortativity_wei(CIJ,flag);
 
    The assortativity coefficient is a correlation coefficient between the
    strengths (weighted degrees) of all nodes on two opposite ends of a link.
    A positive assortativity coefficient indicates that nodes tend to link to
    other nodes with the same or similar strength.
 
    Inputs:     CIJ,    weighted directed/undirected connection matrix
                flag,   0, undirected graph: strength/strength correlation
                        1, directed graph: out-strength/in-strength correlation
                        2, directed graph: in-strength/out-strength correlation
                        3, directed graph: out-strength/out-strength correlation
                        4, directed graph: in-strength/in-strength correlation
 
    Outputs:    r,      assortativity coefficient

3. GLOBAL EFFICIENCY, LOCAL EFFICIENCY

Eglob = efficiency_wei(W);
Eloc = efficiency_wei(W,2);
 
    The global efficiency is the average of inverse shortest path length,
    and is inversely related to the characteristic path length.
 
    The local efficiency is the global efficiency computed on the
    neighborhood of the node, and is related to the clustering coefficient.
 
    Inputs:     W,
                    weighted undirected or directed connection matrix
 
                local,
                    optional argument
                    local=0  computes the global efficiency (default).
                   	local=1  computes the original version of the local
                                efficiency.
                	   local=2  computes the modified version of the local
                                efficiency (recommended, see below). 
 
    Output:     Eglob,
                    global efficiency (scalar)
                Eloc,
                    local efficiency (vector)

4. NODE BETWEEN CENTRALITY

BC = betweenness_wei(L);
 
    Node betweenness centrality is the fraction of all shortest paths in 
    the network that contain a given node. Nodes with high values of 
    betweenness centrality participate in a large number of shortest paths.
 
    Input:      L,      Directed/undirected connection-length matrix.
 
    Output:     BC,     node betweenness centrality vector.


5. AVERAGE NEIGHBOURHOOD DEGREE

wAND=  1/s_i  ∑_(j∈N(i))▒〖w_ij k_j 〗(eq.1)

wij = weight of the edge that link channel i and channel j
kj= degree of node j
si = weighted degree of node i, this is also known as strength of that node
N(i) = neighbourhood of node i

First strength and degree of the node i and j respectively are calculated. 

str = strengths_und(CIJ);
 
    Node strength is the sum of weights of links connected to the node.
 
    Input:      CIJ,    undirected weighted connection matrix
 
    Output:     str,    node strength

deg = degrees_und(CIJ);
 
    Node degree is the number of links connected to the node.
 
    Input:      CIJ,    undirected (binary/weighted) connection matrix
 
    Output:     deg,    node degree

Then the value of Average Neighbourhood Degree was calculated according to eq.1.

Cortical Activation Calculation:

Here I have first segmented the data with 5s of non-overlapping window. For each window I have calculated the first three Intrinsic Mode Function for each time window. Then for each segment first difference of IMF time series, IMF’s phase, normalized Energy of IMF have been calculated. After that, these values are averaged over all the segments.

Other activation feature that is calculated, is Differential Entropy.

All these three features are normalised before creating the main feature matrix. The other components of the feature matrix are the above connectivity features.

 
Plot of linkage between nodes: (1st Alpha, onwards: Theta, Beta, Gamma)
Network linkages with significant weight differences have been calculated using ANNOVA with certain p value. This was done in python.

For our case, I have measured two types of feature dataset, one where the dataset was not segmented, and for the second case, each 60 sec EEG signal data was segmented to 11 segments with a window of 10 seconds. So, there are two separate results for each feature set.
The Feature selection method: 
All_fea: The results have been calculated using all the feature data.
MyFsel: The feature selection was done by first calculating the F score of all the feature vectors, then rearranging then to descending order of F-score. Then I applied SVM classifier, by first taking the feature with highest F score then taking the first two feature and so on. Then we got a loss curve (resulted after 10-fold cross Validation) for first set of features and accuracy for second set of features, after each classification of all the cumulative feature sets. We take the feature set for which there is minimum loss and maximum accuracy.
The below plots are the loss plots for the first set of features: (Classifier to calculate loss = SVM)





The below plots are the accuracy plots for the second set of features: (Classifier to calculate accuracy = RF)
 
PLV_fea: The features calculated from the PLV, was used to measure the accuracy.
PSI_fea: The features calculated from the PSI, was used to measure the accuracy.



 
For second set of feature data: 419 total features
For these features, the previous feature selection method, MyFsel, was not applied, because the maximum F score showed an erratic behaviour. So, we applied auto encoder-based feature selection method. There I had used the following neural network to encode and decode the feature data:
So, we can see the encoded feature number came out to be 64. The while encoding and decoding, the information was retained, this is proved by the loss curve vs iteration of the training (80%) and validation dataset (20%)
Models:
Model Description:
 

 
Machine Learning Models: For second set of features i.e. 419 features
Here first I have implemented Random forest, GLM, GBM, Bayesian Classifier, Stacked GLM-RF-GBM Classifier for all the feature data, amongst them, the Random forest Model gave better result. In the random forest model, the accuracy in prediction varied based on number of trees
Analysis for effect of no of trees in prediction:



