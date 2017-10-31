import numpy as np
from keras.models import Sequential, Model
from keras.layers import Dense
from keras.layers import Dropout
from keras.layers import Flatten, Input, Add
from keras.layers.convolutional import Conv2D
from keras.layers.convolutional import MaxPooling2D
from keras.utils import np_utils
from keras import backend as K
from keras.constraints import maxnorm
from keras.optimizers import SGD
from sklearn.model_selection import train_test_split
import loadDataSet
import keras.layers
import pandas as pd
seed = 11
np.random.seed(seed)
print("loading dataset")
Xtrain, Ytrain, Xtest = loadDataSet.ld()
Xtrain_mean = Xtrain.mean(axis = 0)
Xtest_mean = Xtest.mean(axis = 0)
print("data preprocessing")
Xtrain = Xtrain - Xtrain_mean
Xtest = Xtest - Xtest_mean
print("hyper parmeter tuning and Hot Encoding")
Ytrain_hot_encoded = np_utils.to_categorical(Ytrain, num_classes = 10)
X_train, X_test, Y_train, Y_test = train_test_split(Xtrain, Ytrain_hot_encoded, train_size=0.7)
print("Model building")
print("train data shape {!s}".format(Xtrain.shape))
def createModel():
	print("creating model...")
	input_shape = None
	if K.image_data_format() == 'channels_first':
		input_shape = (1, 28, 28)
	else:
		input_shape = (28, 28, 1)
	input_img = Input(shape=input_shape)
	conv2D_1a_3x3_1 = Conv2D(64, (3, 3), strides=1, padding="same", activation="relu")(input_img)
	conv2D_1b_3x3_1 = Conv2D(128, (3, 3), strides=1, padding="same", activation="relu")(conv2D_1a_3x3_1)
	conv2D_1c_3x3_1 = Conv2D(192, (3, 3), strides=1, padding="same", activation="relu")(conv2D_1b_3x3_1)
	maxpool_1_3x3_1 = MaxPooling2D(pool_size=(2, 2))(conv2D_1c_3x3_1)
	conv2D_2a_3x3_1 = Conv2D(128, (3, 3), strides=1, padding="same", activation="relu")(maxpool_1_3x3_1)
	conv2D_2b_3x3_1 = Conv2D(128, (3, 3), strides=1, padding="same", activation="relu")(conv2D_2a_3x3_1)
	maxpool_2_3x3_1 = MaxPooling2D(pool_size=(2, 2))(conv2D_2b_3x3_1)
	conv2D_3a_3x3_1 = Conv2D(128, (3, 3), strides=1, padding="same", activation="relu")(maxpool_2_3x3_1)
	conv2D_3b_3x3_1 = Conv2D(32, (3, 3), strides=1, padding="same", activation="relu")(conv2D_3a_3x3_1)
	maxpool_3_3x3_1 = MaxPooling2D(pool_size=(2, 2))(conv2D_3b_3x3_1)
	final = Flatten()(maxpool_3_3x3_1)
	final = Dense(1024, activation="relu")(final)
	final = Dense(512, activation='relu')(final)
	final = Dropout(0.2)(final) 
	final = Dense(128, activation='relu')(final)
	final = Dense(10, activation="softmax")(final)
	model = Model(inputs=input_img, outputs = final)
	sgd = SGD(lr=0.1, decay=1e-6, momentum=0.9, nesterov=True)
	model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy', 'categorical_accuracy'])
	print("printing Model Summary...\n{!s}".format(model.summary()))
	return model
model = createModel()
epochs = 10
print("Model Compilation")
print("training...")
model.fit(X_train, Y_train, validation_data = (X_test, Y_test), epochs=epochs, batch_size=32, shuffle=True)
print("Done!")
print("predicting...")
score = model.predict(Xtest, batch_size = 32)
print("Done!")
print("calculating predictions ...")
pred = np.argmax(score, axis = 1)
print("Saving predictions...")
pred = np.column_stack([np.arange(1, Xtest.shape[0]+1).reshape(-1, 1), pred])
#pred = np.column_stack([1, prd])
res = pd.DataFrame(pred, columns=['ImageId', 'Label'], dtype=np.int64)
res.to_csv("final_pred.csv", index=None)
print("Done!...")
