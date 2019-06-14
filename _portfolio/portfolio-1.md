---
title: "Detecting ASL hand gestures using surface EMG"
excerpt: "Using electromyograph signals from a human subject's forearm, some colleagues and I attempted to recognize what American Sign Language hand gestures the subject was performing."
collection: portfolio
---

# Introduction
In my final year of undergrad, inspired by the work of Chen and Wang [1] on the recognition of Chinese number gestures using surface electromyograph, my colleagues [Aye Chan](https://www.linkedin.com/in/ayechanhtun/), [Angela](https://www.linkedin.com/in/angela-chao) and I attempted to apply the methodology used in [1] to a new set of gestures. We decided to attempt to recognize American Sign Language gestures. Being engineering students, we chose the letters in the phrase "hello world", specifically.
<br> <br>
![our chosen ASL gestures](/images/asl_gestures.png)
<br>
*Selected ASL hang gestures*

# Background
## Electromyography (EMG)
Electromyography is the measurement of electrical activity in muscle. The contractions of our muscles are controlled by special cells called motor neurons. These motor neurons are part of our nervous system, which allows our brain to control the rest of our body. When motor neurons excite the cells of our skeletal muscles, it initiates a cascade of electrical activity which can be observed using electrodes placed on the surface of our skin.

## Previous Work
Chen and Wang attempted number gesture recognition using a wireless surface EMG device and a variety of feature extraction and patter classification methods [1]. They were able to achieve very high recognition accuracy (97.93%) using a combination of feature fusion for feature extraction and multiple kernel learning to optimize their support vector machine (SVM) model. They found that using quadratic discriminant analysis (QDA) to classify the features extracted from the EMG signals yielded some of the best results for most feature extraction methods. In addition, this method is quite simple to implement, consequently, it was the one we chose for our own implementation.

# Method
## Data Acquisition
### Surface EMG Device
We used a custom-build, miniaturized, surface EMG device that was graciously lent to us by our instructor, [Prof. Ning Jiang](https://uwaterloo.ca/systems-design-engineering/profile/n34jiang). The device had 8 recording channels that we used to acquire EMG signals. The device had a simple serial COM interface that we leveraged using Matlab. To connect the sEMG device to our subject (me), we used Ag/AgCl electrodes which are commonly used for surface EMG and electrocardiogram recordings.

### Electrode Placement
In [1], electrodes were carefully placed at locations that allowed optimal exposure to the muscled involved in the manipulation of the subject's hand. For simplicity, we chose to arrange the electrodes in a ring around the subject's forearm.
<br> <br>
![electrode placement 1](/images/electrodes_on_arm_1.jpg)
<br>
*Electrode placement on subject's forearm*
<br> <br>
The electrode on the subject's elbow was to provide a reference voltage and the electrode on their hand was a [driven right leg](https://en.wikipedia.org/wiki/Driven_right_leg_circuit) connection to improve common-mode rejection.

### Motion Detection
The surface EMG (sEMG) device constantly streamed data from the electrode connections. To estimate when muscle contractions were occurring we needed to consider how the amplitude of the EMG signals varied over time. We used the average squared magnitude (ASM) of all the recording channels to measure the muscle activity. We compared a moving average of the current ASM to the median ASM over the last few seconds. If the moving average exceeded the median by a manually tuned threshold, a full second of EMG data was stored. We also manually tuned the amount of data that was stored, we found that a full second was enough to capture the necessary information for our gesture recognition problem.

## Data Collection
To collect training data for the system we had our subject perform each of the 7 hand gestures 50 times. The subject collected data for a single gesture at a time to avoid confusion. However, it could be argued that collecting the data in batches of 7 unique gestures since it reduces the chances of the subject becoming fatigued of biased in their gesturing performance. We used our aforementioned motion detection method during these recording sessions. We were also careful to collect the best samples possible, with minimal movement artefacts and error in the performance of the gesture, consequently, there was a substantial amount of repeated trails. As the subject, I can confirm this process was relatively dull, but absolutely essential.

## Feature Extraction
Chen and Wang considered a wide variety of the feature extraction methods in their experiments. These included, ad-hoc time-domain features inspired by earlier work by Hudgins et al. [2], auto and cross-correlation features, and frequency-domain spectral magnitude features. We hypothesized that the best approach would be to select a subset of featured from each category since the feature fusion method described in [1] proved to be the most effective for Chen and Wang. To narrow down our options, we performed two kinds of feature selection analysis. Firstly, we compared the intra-class distances of each feature to the inter-class distance of the features to identify the features that were most likely to be useful. Secondly, we investigated the correlation between features to avoid allow elimination of redundant features. The outcome was a collection of 45 features that we expected to be yield the best performance.

## Model Generation
We generated the pattern recognition model using QDA. This pattern recognition method works by generating a hyperplane in the feature space that separates samples of 2 different classes. New samples are projected onto the hyperplane and classified according to the sign of the result (which indicates which side of the plane the new sample is on). Since we had 7 classes that we needed to consider, we needed to use a more sophisticated decision rule. For each pair of classes a hyperplane was generated. Each new sample was projected onto each hyperplane and the class it was most often classified as was selected as the final classification of the new sample. This classification approach for multiple classes using binary classifiers is often referred to as "one-versus-one" classification.
<br><br>
We also attempted to use "one-versus-many" classification which involved generating a single hyperplane that separated each class from all other classes and using the new sample's projection onto each hyperplane as a measure of how likely the sample was to belong to each class. However, we found the "one-versus-one" approach was more effective.

# Results
We evaluated the system offline (using data we already collected) and online (in real-time using the collected samples as training data).

## Offline Evaluation
We evaluated the offline accuracy of our system by performing one-held-out cross validation on the training data we collected. In other words, we trained our QDA model on features from all but one of our 7 x 50 = 350 data samples and recorded how the held out sample was classified. Here's a confusion matrix of our results.
<br> <br>
![offline confusion matrix](/images/offline_cm.png)
<br>
*Confusion matrix for offline evaluation*
<br> <br>
The average classification accuracy was approximately 98.2%. This was quite a satisfying result given how close it was to the results achieved in [1]. However, our task was much simpler in a variety of ways. For instance, in [1], the results presented were from six different subjects performing 10 gestures, rather than a single subject performing 7. Another factor is that, the sEMG device used in [1] had only 4 recording channels available as opposed to our 8-channel device.
## Online Evaluation
As one might expect, the online results were not as good. This was not surprising since there was less experimental control during these trials that the original data collection session. We conducted 2 online testing sessions. During each session, the subject performed each gesture 20 times and the predicted gesture was observed and recorded.
<br><br>
![online confusion matrix from session 1](/images/online_cm1.png)
<br>
*Confusion matrix for online session #1*
<br><br>
![online confusion matrix from session 2](/images/online_cm2.png)
<br>
*Confusion matrix for online session #2*
<br><br>
In session \#1, we achieved and average accuracy of 67.9%. In preparation for session \#2, the 20 recordings of each gesture collected in the first session were included in the training data set. This appears to have had a significant effect of the performance of the online recognition, perhaps because these 20 new samples allow the system to accommodate the errors and artefacts that might be present in these real-time evaluation sessions. In the second session, the average accuracy was 85.7%.
<br><br>
**Bonus**: [here is a video](https://youtu.be/ULLmmtaTbqs) of me testing out our system. It is important to note that, the subject was allowed to view the waveforms and recognition outputs of the system. We did not consider it at the time, but this is a suboptimal arrangement since it makes the subject's behavior more likely to be biased. As the subject in question, I admit that seeing the results of the trials did have an influence on how I performed the next gestures.

# Conclusion
We successfully adapted the work of Chen and Wang [1] to a slightly different, simplified application. Specifically, the recognition of 7 ASL letter hand gestures performed by a single subject. We achieved an offline accuracy of 98.2% and online accuracies of 67.9% and 85.7%.

# References

[1] | X. Chen and Z. J. Wang, "Pattern recognition of number gestures based on a wireless surface emg system," Biomedical Signal Processing and Control, vol. 8, no. 2, pp. 184-192, 2013. (<https://www.sciencedirect.com/science/article/pii/S1746809412000870>)
[2] | B. Hudgins, P.A. Parker, R. Scott, A new strategy for multifunction myoelectric control, IEEE Transactions on Biomedical Engineering, 40 (1) (1993), pp. 82-94
