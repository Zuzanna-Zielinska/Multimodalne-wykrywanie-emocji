# Multimodal emotion detection
The repository contains a description of my master’s thesis, the result of which is a recurrent neural network for emotion detection. The work involved designing and conducting a study that collected recordings of emotional reactions to stimuli using an eye tracker, camera, and heart rate monitor; processing raw data into training datasets; designing and training neural networks. [The full thesis text is available here.](https://github.com/Zuzanna-Zielinska/Multimodalne-wykrywanie-emocji/blob/main/Multimodalne%20wykrywanie%20emodji.pdf)

Study
During the study, eye movement, facial expressions and heart rate were recorded while participants viewed images from the NAPS database. After each image, participants reported their emotions in a questionnaire.



## Study
During the study, eye movement, facial expressions and heart rate were recorded while participants viewed images from the NAPS database. After each image, participants reported their emotions in a questionnaire.

<p align="center">
<img src=".\images\diagramy-stanowisko pomiarowe.drawio.png" alt="test-bench">
</p>

### Questionnaire
In the questionnaire, participants were asked to describe their emotions verbally and complete the SAM questionnaire.
The SAM questionnaire (Self-Assessment Manikin) is based on the assumption that emotions can be described using three nine-point scales:

•	pleasure – whether the participant feels negative or positive emotions,
<p align="center">
<img src=".\images\wartościowość.JPG" alt="wartosciowosc">
</p>
•	arousal – whether the participant is calm or excited,
<p align="center">
<img src=".\images\pobudzenie.JPG" alt="pobudzenie">
</p>
•	dominance – whether the participant is in control of the emotion or controlled by it.
<p align="center">
<img src=".\images\dominacja.JPG" alt="dominacja">
</p>

### Image Database
The study used the NAPS database (Nencki Affective Picture System). It consists of 1,356 images that evoke emotions, along with their average ratings for pleasure, arousal, and dominance. Each image is categorized into one of five groups: people, faces, landscapes, animals, and objects. Additionally, 510 images were already selected and analysed to determine which of the six basic emotions they evoke.

<p align="center">
<img src=".\images\NAPS.png" alt="NAPS">
</p>

From these 510 images, two sets were selected based on the most diverse SAM ratings. The first set contains 20 images, and the second contains 30.
In the smaller set, images were sorted by valence in ascending order, and 5 evenly spaced images were selected. The same process was repeated for arousal and dominance. The remaining images were chosen to ensure that all six basic emotions were represented.
The larger set was selected analogously, except that 7 evenly spaced images were chosen instead of 5.


## Raw Data Format
### Facial expressions
Each video recording of facial expressions had an associated CSV file containing a table where each row stored the frame number and its corresponding timestamp.

### Heart rate
Each row contained a heart rate value recorded by the monitor and its corresponding timestamp.

### Eye tracking
Eye-tracking recordings were imported from Tobii Pro Lab in TSV format. The table had 102 columns, including detailed information about:

•	the experiment (e.g., timestamps, sensor name, experiment start time),
•	the participant (e.g., name, gender, age),
•	the displayed image (e.g., name, resolution),
•	calibration results,
•	eye movement,
•	pupil size,
•	eye openness.

## Data Preparation for Training
Data preparation consisted of the following steps:
•	removing irrelevant data,
•	checking whether each sample contains all modalities,
•	extracting facial landmark points,
•	synchronizing data,
•	filling missing values with zeros,
•	encoding words,
•	normalizing data.

To convert a frame of facial expression video into a feature vector, face detection was first performed using the HOG (Histogram of Gradients) algorithm and a trained SVM classifier. Then, feature extraction was carried out using the Kazemi-Sullivan algorithm.


<p align="center">
<img src=".\images\punkty.png" alt="punkty">
</p>

## Network Architecture
The goal of the network is to predict SAM questionnaire values based on tables of recorded emotional expressions. Inputs consist of label tables and data tables with 208 columns and varying numbers of rows. This variability arises because participants decided themselves when to move from viewing an image to answering the questionnaire. The dataset was split into: training set (258 samples), test set (54 samples), validation set (45 samples).

<p align="center">
<img src=".\images\projekt rozłożony na części2.png" alt="a1">
</p>

The batch size was set to 1 due to varying input lengths and the implementation of the TensorFlow library.
Each input table had a different number of rows. To compensate for this, weighted metrics were used during training, with weights inversely proportional to the table length.
The network outputs three values ranging from 1 to 9. This problem can be approached in two ways: as classification, since the values are discrete, as regression, since each value represents a point on a scale. Additionally, one network can predict all three values, or three separate networks can predict each value independently. Based on this, 20 regression architectures predicting all three values and 20 classification architectures (three networks each predicting one value) were trained. For simplicity, the classification models shared a single input that branched into three separate sequences of identical layers.

<p align="center">
<img src=".\images\architektura1.png" alt="arch1">
</p>

<p align="center">
<img src=".\images\architektura2.png" alt="arch2">
</p>

Regression models were trained with following parameters:
•	optimizer: Adam,
•	learning rate: 8×10⁻⁷,
•	loss function: mean squared error,
•	weighted metric: mean squared error.
Classification models were trained with following parameters:
•	optimizer: Adam,
•	learning rate: 8×10⁻⁷,
•	loss function: sparse categorical cross-entropy,
•	weighted metric: accuracy.

Both types of models started with an input layer. Regression models ended with a fully connected layer with three neurons and sigmoid activation and scaling layer mapping values from (0,1) to (1,9). Their outputs directly corresponded to SAM scores.
Classification models ended with three fully connected layers with nine neurons and sigmoid activation, producing class probabilities. The predicted class was the one with the highest probability.
Hidden layers varied between models and included at least one LSTM layer and optional combinations of additional LSTM layers, dropout layers (dropping 20% of connections), convolutional layers (kernel size 5), dense layers with hyperbolic tangent activation.


| Name   | Architecture                                   |
|---------|-----------------------------------------------|
| model1  | lstm                                          |
| model2  | lstm, dense, dense                            |
| model3  | dense, dense, lstm                            |
| model4  | dense, dense, lstm, dense, dense              |
| model5  | dense, dense, lstm, dense, dropout, dense     |
| model6  | conv, lstm                                    |
| model7  | conv, conv, lstm                              |
| model8  | conv, dense, conv, lstm                       |
| model9  | conv, dense, conv, dropout, lstm             |
| model10 | conv, lstm, dense, dense                      |
| model11 | conv, conv, lstm, dense, dense                |
| model12 | lstm, lstm                                    |
| model13 | lstm, lstm, dense, dense                      |
| model14 | dense, dense, lstm, lstm                      |
| model15 | dense, dense, lstm, lstm, dense, dense        |
| model16 | dense, dense, lstm, lstm, dense, dropout, dense |
| model17 | dense, dense, lstm, dense, lstm, dense, dense |
| model18 | dense, dense, lstm, dense, lstm, dense, dropout, dense |
| model19 | conv, lstm, lstm                              |
| model20 | lstm, lstm, lstm                              |


The LSTM layer aggregates predictions across the entire data table into a single feature vector. If multiple LSTM layers were used, earlier layers passed predictions for each row forward.
Each model was trained for 50 epochs. After each epoch: if validation performance improved, weights were saved, if no improvement occurred for 3 epochs, training was stopped early. Training for each model was repeated 5 times, and the best-performing version was saved.Additionally, all architectures were trained once for 50 epochs without early stopping.
