import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
from keras import applications
from keras.models import Model
from keras.layers import Dense, Dropout, Activation, Flatten, GlobalAveragePooling2D
from keras.callbacks import ModelCheckpoint
import skimage.transform as sktrans
from sklearn.preprocessing import LabelEncoder
from keras.utils import np_utils 

input_shape = (148, 148, 3)
#create model
model = applications.VGG19(weights='imagenet', include_top = False, input_shape=input_shape)
print("model is: \n{!s}".format(model.summary()))
#load dataset
imgs_csv = pd.read_csv('labels.csv')
train_img_path = 'train/'
img_list = imgs_csv['id']
total_images = len(imgs_csv.index)
train_images = np.zeros((total_images, *input_shape))
print("loading data...")
for i, c in enumerate(img_list):
	img = sktrans.resize(mpimg.imread(train_img_path+ c+'.jpg'), tuple(input_shape[0:2]))
	img = img[:, :, [0, 1, 2]]
	train_images[i] = img
print("loading labels...")
yTrain = imgs_csv['breed']
xTrain = train_images
print("making predictions...")
predictions = model.predict(xTrain).reshape((xTrain.shape[0], -1))
print("predictions are: {!s}".format(predictions))
df = pd.concat([pd.DataFrame(yTrain.reshape(-1, 1), columns=['breed']), pd.DataFrame(predictions, columns=['f_' + str(i) for i in range(predictions.shape[1])])], axis=1)
df.to_csv('through_inception.csv')
