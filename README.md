# GEBI
 Global Explanations for Bias Identification

!()['anim.gif']

###  Global Explanations for Bias Identification

Abstarct:
In the paper, we propose attention-based summarized
post-hoc explanations for detection and identification of
bias in data. We propose a global explanation and
introduce a step-by-step framework on how to detect and
test bias. Then, the bias is evaluated with proposed
counterfactual approach to bias insertion. Because
removing the unwanted bias is often a complicated and
tremendous task, we automatically insert it, instead. We
validate our results on the example of the skin lesion
dataset. Using the method, we successfully identified and
confirmed part of the possible bias-causing artifacts in
dermoscopy images. We confirmed that the commonplace
black frames in the training dataset images have a strong
influence on the Convolutional Neural Network’s
prediction. After artificially adding a black frame to all
images, around 22% of them changed the prediction from
benign to malignant. We have shown that bias detection is
an important step of making more robust models, and we
discuss how to improve them

### Bias insertion
