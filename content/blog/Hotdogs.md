---
title: Hotdog Classifier in R
date: 2018-06-20

categories:
- R

tags:
- Image Classification
- R
- Hotdog
- Not Hotdog
---
I was inspired by Silicon Valley's Jian Yang to make a Hotdog classifier in R with very little data (even though Python is way faster).

 
## Demonstration of the classifier we are looking to make:

{{< youtube ACmydtFDTGs >}}


## Step 1: Collecting the Data
The first to making a classifier is to collect data and we will need to find pictures of hotdogs and not hotdogs.

Following a [guide](https://www.pyimagesearch.com/2017/12/04/how-to-create-a-deep-learning-dataset-using-google-images/), I managed to quickly download a collection of hotdog and not hotdog pictures from Google Images.

Here are some examples of the pictures I collected:

### Example Hotdogs
![Hotdog1](/img/hotdog001.jpg)
![Hotdog2](/img/hotdog002.jpg)
![Hotdog3](/img/hotdog003.jpg)

## Example Not Hotdogs
![NotHotdog1](/img/nothotdog001.jpg)
![NotHotdog2](/img/nothotdog002.jpg)
![NotHotdog3](/img/nothotdog003.jpg)
![NotHotdog4](/img/nothotdog004.jpg)


## Step 2: Setting up the files
After scraping the web for hundreds of hotdog and nothotdog pictures, I had two folders organized as below:

```{r}
data/
    hotdog/
           hotdog001.jpg
           hotdog002.jpg
           ...
    nothotdog/
            nothotdog001.jpg
            nothotdog002.jpg
            ...
```

To set up my data for the training, validation, and testing processes, I organized the dataset (~ 720 photos in total) into 3 sub-folders:

```{r}
to-hotdog-or-not-to-hotdog/
                          train/
                                hotdog/
                                       hotdog001.jpg
                                       hotdog002.jpg
                                       ...
                                nothotdog/
                                       nothotdog001.jpg
                                       nothotdog002.jpg
                                       ...
                     validation/
                                hotdog/
                                      hotdog001.jpg
                                      hotdog002.jpg
                                      ...
                                nothotdog/
                                      nothotdog001.jpg
                                      nothotdog002.jpg
                                      ...
                           test/
                                hotdog/
                                      hotdog001.jpg
                                      hotdog002.jpg
                                      ...
                                nothotdog/
                                      nothotdog001.jpg
                                      nothotdog002.jpg
                                      ...
                                    
```
## Step 3: Using a Pretrained Covnet
Since I have quite a small dataset, it makes sense to use a pretrained network. A pretrained network is a saved network that was previously trained on a large dataset. In this case, we'll be using the [VGG16 architecture](https://arxiv.org/abs/1409.1556), which is a simple and widely used convnet trained on the ImageNet dataset (1.4 million labelled images and 1000 different classes). Luckily, the VGG16 model comes prepackaged with Keras. Although Keras is a Python library, we can somewhat use it in R too.

```
# Load libraries
library(dplyr)
library(keras)

```
```
# Instantiate the VGG16 model

conv_base <- application_vgg16(
  weights = "imagenet",
  include_top = FALSE,
  input_shape = c(64, 64, 3)
)
```
`weights` is the path to the weights file to be loaded.  
`include_top` is whether to include the densely layered classifier (1000 classes) at the top of the network. However, we will only be using our own classifier with two classes: hotdog and not hotdog.  
`input_shape` is the shape of images we'll be feeding the network.  


We'll be using the Sequential Model from Keras. In short, the Sequential model is a linear stack of layers. Here, we add `conv_base` to a sequential model just like we would add a layer.
```
model <- keras_model_sequential() %>%  
  conv_base %>%                         
  layer_flatten() %>% 
  layer_dense(units = 256, activation = "relu") %>% 
  layer_dense(units = 1, activation = "sigmoid")
```

Before I go ahead and train the model, it's important to freeze the convolutional base. Essentially, *Freezing* a layer or a set of layers prevents their weights from being changed during training.

```
freeze_weights(conv_base)
```
Now that the `conv_base` layer is frozen, only weights from the `relu` and `sigmoid` layers will be updated during training.  

## Step 4: Dealing with Overfitting
Overfitting is a common problem in data science, generally meaning that our model has too little data to learn from and tends to perform poorly on new data. In a perfect world, the model would be exposed to every possible aspect of the data distribution at hand and will never overfit. However, since we just don't have the time to download more hotdog photos, we used data augmentation to randomly transform our existing sample of images to generate more training data.

```
# Image generator
train_datagen = image_data_generator(
  rescale = 1/255,
  rotation_range = 40,
  width_shift_range = 0.2,
  height_shift_range = 0.2,
  shear_range = 0.2,
  zoom_range = 0.2,
  horizontal_flip = TRUE,
  fill_mode = "nearest"
)
```
Going quickly over the parameters of this code:  

`rotation_range` is a value in degrees (0–180), a range within which to randomly rotate pictures.  
`width_shift` and `height_shift` are ranges (as a fraction of total width or height) within which to randomly translate pictures vertically or horizontally.  
`shear_range` is for randomly applying shearing transformations.   
`zoom_range` is for randomly zooming inside pictures.  
`horizontal_flip` is for randomly flipping half the images horizontally.  
`fill_mode` is the strategy used for filling in newly created pixels.  

## Step 5: Training our Model

Now we can finally train our model using the image generator:

```
# Note that the validation data shouldn't be augmented!
test_datagen <- image_data_generator(rescale = 1/255)  

train_generator <- flow_images_from_directory(
  train_dir,                  # Target directory  
  train_datagen,              # Data generator
  target_size = c(64, 64),  # Resizes all images to 64 × 64
  batch_size = 20,
  class_mode = "binary"       # binary_crossentropy loss for binary labels
)

validation_generator <- flow_images_from_directory(
  validation_dir,
  test_datagen,
  target_size = c(64, 64),
  batch_size = 20,
  class_mode = "binary")

model %>% compile(
  loss = "binary_crossentropy",
  optimizer = optimizer_rmsprop(lr = 1e-5),
  metrics = c("accuracy"))

history <- model %>% fit_generator(
  train_generator,
  steps_per_epoch = 200,
  epochs = 5,
  validation_data = validation_generator,
  validation_steps = 100)
```
A visualization of the [training process](http://rpubs.com/kail/hotdogs).  

## Step 6: Evaluation 
We can now evaluate the model:

```{r}
test_generator <- flow_images_from_directory(
  test_dir,
  test_datagen,
  target_size = c(64, 64),
  batch_size = 20,
  class_mode = "binary"
)
```

```
model %>% evaluate_generator(test_generator, steps = 25)

$loss
[1] 0.3901784

$acc
[1] 0.912
```
> 91% Accuracy. Spectacular!