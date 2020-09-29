# FDA  Submission

**Your Name:** Mercedes Riveira

**Name of your Device:** Pneumonia Detector

## Algorithm Description 

### 1. General Information

**Intended Use Statement:** Assisting radiologist with classifiyng pneumonia or no-pneumonia images within a screening program. 

**Indications for Use:** This algorithm is intended for use in chest X-Ray images, taken at both positions PA and AP in people aging 80 or less years. It works better. It is not concluded if its performance is worsened in patients with other diseases present.  

**Device Limitations:** The algorithm was not trained for patients over 80 years old. The images must be chest X-Rays in PA or AP views. It has been seen that when fibrosis and pleural thickening diseases are comorbid with pneumonia, the algorithm's sensitivity decreases, so is is not recomended to use it for patients with these other diseases.   

**Clinical Impact of Performance:** 

+ Threshold: 0.4
    + Accuracy: 85%
    + Precision: 17%
    + Recall: 93% (sensitivity)

The threshold has been weighed in favour of recall (sensitivity), which means that most of the true positive cases (TPR = Sensitivity) are being well classified at the expense of classify much of the negative cases as positive cases (that is, a high False Positive Rate and therefore a low specificity)

Since recall = TP / (TP + FN), if recall is closer to 1, FN is approximately cero (no false negatives). This means that when the algorithm runs an image and returns a **negative** result, you can be pretty sure that the result is truely a negative. This is so useful in **screening** situations (very important nowadays with COVID-19). 

### 2. Algorithm Design and Function

![Captura de pantalla 2020-09-29 a las 14.08.29](/Users/mercedes/Library/Application Support/typora-user-images/Captura de pantalla 2020-09-29 a las 14.08.29.png)

**DICOM Checking Steps:** When an image is fed into the algorithm, the ``check_dicom`` function reads the DICOM file and checks the important fields for our device (Body part examinated, modality and patient position). If the image is from the CHEST, in modality DX and taken in a PA or AP position, then the DICOM image (as a pixel array) is returned. 

**Preprocessing Steps:** this function takes the image (pixel array) obtained from che ``check_dicom`` function, its mean, standard deviation and size (image size corresponding with that of VGG16 input image size of (1,224,224, 3)) as arguments, and returns the same image but procesed. This procesing consist of a rescaling and resizing. Finally, the valid images that are preprocessed are fed as an input to the algorithm. 

**CNN Architecture:** This algorithm is build over the first 17 layers of a pretrained model (VGG16 architecture): 

![Captura de pantalla 2020-09-29 a las 13.57.25](/Users/mercedes/Desktop/Captura de pantalla 2020-09-29 a las 13.57.25.png)



Then, the following layers were added: flattening layer, three dropouts (at 0.5) to prevent overfitting, and three dense layers (of 512, 256 and 1 parameters, both firsts relu, and the last one a sigmoid). 

![Captura de pantalla 2020-09-29 a las 13.47.10](/Users/mercedes/Desktop/Captura de pantalla 2020-09-29 a las 13.47.10.png)




### 3. Algorithm Training

**Parameters:**
* Types of augmentation used during training:

   ![Captura de pantalla 2020-09-29 a las 14.09.57](/Users/mercedes/Library/Application Support/typora-user-images/Captura de pantalla 2020-09-29 a las 14.09.57.png)

* Batch size:

  * Train: 90
  * Validation: 32

* Optimizer learning rate: 

  *  Adam(lr = 1e-4)

* Layers of pre-existing architecture that were frozen: first 17 

* Layers of pre-existing architecture that were fine-tuned: 

  ```R
  block5_conv3 True
  block5_pool True
  ```

* Layers added to pre-existing architecture: flattening layer, three dropouts (at 0.5) to prevent overfitting, and three dense layers (of 512, 256 and 1 parameters, both firsts relu, and the last one a sigmoid). 

### Algorithm training performance visualization   

![Captura de pantalla 2020-09-29 a las 14.13.08](/Users/mercedes/Library/Application Support/typora-user-images/Captura de pantalla 2020-09-29 a las 14.13.08.png)

### Insert P-R curve 

![Captura de pantalla 2020-09-29 a las 14.13.23](/Users/mercedes/Library/Application Support/typora-user-images/Captura de pantalla 2020-09-29 a las 14.13.23.png)

**Final Threshold and Explanation:** The final threshold was set at 0.4 because this number favours recall. Since this is a medical algorithm, it is very important to classificate all true positives at expenses of classifying some negatives as positives (false positives). The highest F1 score is 0.36 and corresponds to a threshold of 0.44. 

![Captura de pantalla 2020-09-29 a las 14.16.27](/Users/mercedes/Library/Application Support/typora-user-images/Captura de pantalla 2020-09-29 a las 14.16.27.png)

### 4. Databases
The dataset was split following the 80-20% rule, that is: 80% for training and 20% for validation. 

**Description of Training Dataset:** The proportion of Pneumonia cases in the training set is 1.2% (1145 cases out of 89696 images). This set was reshaped  in order to balance the proportion of pneumonia and no pneumonia cases (50%-50%). Finally, our training datset contains 2290 images, 1145 showing pneumonia. 

**Description of Validation Dataset:** The proportion of Pneumonia cases in the validation set is 1.2% (286 cases out of 22424 images), the same as the training set. This set was reshaped  in order to adjust the proportion of pneumonia and no pneumonia cases to  20%-80%. 

### 5. Ground Truth

The disease labels were created using Natural Language Processing (NLP) to mine the associated radiological reports. The labels include 14 common thoracic pathologies: Atelectasis, Consolidation, Infiltration, Pneumothorax, Edema, Emphysema,Fibrosis, Effusion, Pneumonia, Pleural thickening, Cardiomegaly, Nodule, Mass, Hernia. 

The biggest limitation of this dataset is that image labels were NLP-extracted so there could be some erroneous labels but the NLP labeling accuracy is estimated to be >90%.

### 6. FDA Validation Plan

**Patient Population Description for FDA Validation Dataset:**

This dataset contains 112,120 x-ray chest images, obtained from the NIH database. The distribution of the diseases (figure below) shows that the 'No-finding' label is the most common  (53%) followed by Infiltration, Effusion and Atelectasis. Pneumonia only constitutes 1.2% of the hole dataset

![Captura de pantalla 2020-09-29 a las 15.49.32](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2015.49.32.png?lastModify=1601389037)

 ![Captura de pantalla 2020-09-29 a las 15.46.33](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2015.46.33.png?lastModify=1601389037)

As well, the 30 most common concurrences are shown in the figure below. Stands our infiltration, followed by edema + infiltration and so on. 

![Captura de pantalla 2020-09-29 a las 15.51.14](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2015.51.14.png?lastModify=1601389037)



The gender distribution shows that there are more male than female figures (ratio 3:2 approximately). 

![Captura de pantalla 2020-09-29 a las 15.53.20](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2015.53.20.png?lastModify=1601389037)

The hole dataset contains images from patients aging less than 90 years, but the bulk of patients covers 20-70 years.. 

![Captura de pantalla 2020-09-29 a las 15.54.44](file:///Users/mercedes/Desktop/Captura%20de%20pantalla%202020-09-29%20a%20las%2015.54.44.png?lastModify=1601389037)



There are two possible patient positions: PA or AP. As we can see, the PA view dominates over the AP. 

![Captura de pantalla 2020-09-29 a las 15.59.38](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2015.59.38.png?lastModify=1601389037)



For patients contaning Pneumonia, this graphs are as follows: 

![Captura de pantalla 2020-09-29 a las 16.02.52](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2016.02.52.png?lastModify=1601389037)

![Captura de pantalla 2020-09-29 a las 16.03.05](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2016.03.05.png?lastModify=1601389037)

![Captura de pantalla 2020-09-29 a las 16.03.15](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2016.03.15.png?lastModify=1601389037)



That is: the ratio of male:female images is approximately equal, patients with pneumonia are mostly aging 45-65 years and for pneumonia cases the dominant view is AP. An example of 20 images is shown below. 

![Captura de pantalla 2020-09-29 a las 16.13.01](file:///Users/mercedes/Library/Application%20Support/typora-user-images/Captura%20de%20pantalla%202020-09-29%20a%20las%2016.13.01.png?lastModify=1601389037)

**Ground Truth Acquisition Methodology:** The NIH dataset radiologist reports were mined using Natural Language Processing (NLP) to create the disease labels from the associated radiology reports. The silver standard involves hiring several radiologists to each make their own diagnosis of an image. The final diagnosis is then determined by a voting system across all of the radiologists’ labels for each image. Note, sometimes radiologists’ experience levels are taken into account and votes are weighted by years of experience.

**Algorithm Performance Standard:** the threshold was chose to favour recall and then minimise the number of false negatives. The mean F1 score for radiologist is about 0.387 (95% CI 0.330, 0.442) [1], and this CNN scores a 0.36, which is pretty close to the standard of reference. 



## References

[1] Rajpurkar, Pranav, et al. "Chexnet: Radiologist-level pneumonia detection on chest x-rays with deep learning." *arXiv preprint arXiv:1711.05225* (2017).