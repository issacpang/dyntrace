# `dyntrace`

**Dyn**amic **T**ime-series **R**esponse **A**nalysis and **C**omputational **E**stimation

This guidance note provides a brief summary of how to use the developed
LSTM code for testing. In general, the LSTM code consists of two main
modules: **Training** and **Utility Functions**. The **Training** module
contains all LSTM algorithms (e.g., LSTM(Vanilla) and TCN). If you wish
to use/develop the LSTM algorithms to train a new model, please go to
the **Training** module. The **Utility Functions** module is mainly used
to generate the 3D CSMIP data for training and to evaluate the
performance of trained LSTM models.

<figure>
<embed src="images/LSTM_arhitecture2.pdf" id="fig:LSTM_arhitecture" style="width:60.0%" /><figcaption aria-hidden="true">LSTM Code Organization</figcaption>
</figure>

To use the code for testing or for other research purposes, please
download all files in the **Training** and **Utility Functions**
modules. Then, update the directory of `train_folder_path` (line 9)
and `modelPath` (line 39). The `train_folder_path` is used to store
all CSMIP training data, and the `modelPath` is used to save the
trained LSTM model. Number of time steps may also need to be updated
(line 10), depending on the CSMIP data.

After training the LSTM model, `Evaluation.py` can be used to evaluate
the accuracy of the prediction. Please update the directory of
`train_folder_path` (line 32), `test_folder_path` (line 33) and
`modelPath` (line 48). The `train_folder_path` and
`test_folder_path` are used to store all CSMIP training and testing
data respectively, and the `modelPath` is used to load the trained
LSTM model. Please also update the name of the trained LSTM model
(e.g., `vanilla_30_addFC`, no need to include `"_model.h5"`). The number
of time steps may also need to be updated (line 10), depending on the
CSMIP data. Lines 60-92 are used to plot the acceleration responses of
the predicted results and the target results for both training and
testing sets. The "3" in lines 62, 70, 79 and 87 indicates that the
index of the interested CSMIP data (it should be the third item in the
**train_folder_path** and **test_folder_path**). If required, please
change the number to plot the interested CSMIP dataset. Lines 99-104 are
used to calculate the correlation coefficient of the predicted and the
target results. It should be noted that
`np.corrcoef()` generates a $2 \times 2$ coefficient matrix,
which shows 1.0 in both diagonal elements, indicating that each dataset
is perfectly correlated with itself, and  ≤ 1.0 in the off-diagonal
elements, indicating that how the predicted and target results are
correlated with each other. The code (lines 99-104) automatically
extracts the off-diagonal element, showing the correlation between the
predicted and the target results. For the details of calculation, please
go to:
<https://numpy.org/doc/stable/reference/generated/numpy.corrcoef.html>.

```python
def save_model_dict(dictionary, name, modelPath):
    dictionary['model'].save(modelPath+r"model_response"+name+"_model.h5") 
    f = open(modelPath+r"model_response"+name+"\_history.pkl","wb")
pickle.dump(dictionary['history'].history, f) f.close() f =
open("runtime.txt", "a") f.write(name+" runtime: ")
f.write(str(dictionary['runtime']/60)) f.write("") f.close()

def load_model_dict(name, modelPath): dictionary = try: model =
load_model(modelPath+r"model_response  
"+name+".hdf5") except: model = load_model(modelPath+r"model_response  
"+name+"\_model.h5") dictionary['model'] = model f =
open(modelPath+r"model_response  
"+name+"\_history.pkl", 'rb') history = pickle.load(f) f.close()
dictionary['history'] = history return dictionary

if \_\_name\_\_ == "\_\_main\_\_":

#specify the train_folder_path and the test_folder_path #specify time
step (each seismic event has different timesteps, so need to define
manually) #specify output_response (e.g. Accel data in Channel 3, 5, 8
=> then 3) #specify window size (stack size, please refer to Zhang2019
for details) train_folder_path = r'C:' test_folder_path = r'C:'

time_step = 7200 output_response = 3 window_size = 2

#CSMIP data is read into "train_file" and "test_file". train_file =
Read_CSMIP_data.Read_CSMIP_data(train_folder_path, time_step,
output_response, window_size) test_file =
Read_CSMIP_data.Read_CSMIP_data(test_folder_path, time_step,
output_response, window_size)

#The required 3d array will be generated for training/testing datax,
datay = train_file.generate_3d_array() testx, testy =
test_file.generate_3d_array()

#load the trained model modelPath = r'C:  
' model_dict = load_model_dict("vanilla_30_addFC", modelPath) model =
model_dict['model']

#perform prediction and load the results in datapredict and testpredict
datapredict = model.predict(datax) testpredict = model.predict(testx)
print("train_loss:") print(model.evaluate(datax, datay, verbose=0))
print("test_loss:") print(model.evaluate(testx, testy, verbose=0))

#Sample Plot of training data plt.figure()
plt.plot(model.predict(datax)[3,:,0], color='blue', lw=1.0)
plt.plot(datay[3,:,0],':', color='red', alpha=0.8, lw=1.0)
plt.title('Training Set: 3rd Floor Acceleration (x-direction)')
plt.legend(["Predicted", "Real"]) plt.xlabel("Time Step")
plt.ylabel("Acceleration (cm/sec<sup>2</sup>)")

plt.figure() plt.plot(model.predict(datax)[3,:,1],
color='blue',lw=1.0) plt.plot(datay[3,:,1],':', color='red',
alpha=0.8, lw=1.0) plt.title('Training Set: Roof Acceleration
(x-direction)') plt.legend(["Predicted", "Real"]) plt.xlabel("Time
Step") plt.ylabel("Acceleration (cm/sec<sup>2</sup>)")

#Sample Plot of testing data plt.figure()
plt.plot(model.predict(testx)[3,:,0], color='blue', lw=1.0)
plt.plot(testy[3,:,1],':', color='red', alpha=0.8, lw=1.0)
plt.title('Testing Set: 3rd Floor Acceleration (x-direction)')
plt.legend(["Predicted", "Real"]) plt.xlabel("Time Step")
plt.ylabel("Acceleration (cm/sec<sup>2</sup>)")

plt.figure() plt.plot(model.predict(testx)[3,:,1],
color='blue',lw=1.0) plt.plot(testy[3,:,0],':', color='red',
alpha=0.8, lw=1.0) plt.title('Testing Set: Roof Acceleration
(x-direction)') plt.legend(["Predicted", "Real"]) plt.xlabel("Time
Step") plt.ylabel("Acceleration (cm/sec<sup>2</sup>)")

# Correlation Coefficient # Note: The resulting matrix from np.corrcoef
shows this by having 1.0 # in both diagonal elements, indicating that
each array is perfectly correlated # with itself, and \< 1.0 in the
off-diagonal elements, indicating that how the two arrays # are
correlated with each other. print("training corr") train_corr =
np.corrcoef(datapredict.flatten(), datay.flatten())[0,1]
print(train_corr) print("testing corr") test_corr =
np.corrcoef(testpredict.flatten(), testy.flatten())[0,1]
print(test_corr)

# Error - evaluate the error between the predicted result and the real
result errors = np.array([])

x = (datapredict[:,:,0] - datay[:,:,0]) / np.max(np.abs(datay[:,:,0]), axis=1).reshape((-1,1)) 
hist = np.histogram(x.flatten(), np.arange(-0.2, 0.201, 0.001))[0] 
errors = np.append(errors, hist) 
x = (datapredict[:,:,1] - datay[:,:,1]) / np.max(np.abs(datay[:,:,1]), axis=1).reshape((-1,1)) 
hist = np.histogram(x.flatten(), np.arange(-0.2, 0.201, 0.001))[0] 
errors = np.append(errors, hist)
x = (testpredict[:,:,0] - testy[:,:,0]) / np.max(np.abs(testy[:,:,0]), axis=1).reshape((-1,1)) 
hist = np.histogram(x.flatten(), np.arange(-0.2, 0.201, 0.001))[0] 
errors = np.append(errors, hist)
x = (testpredict[:,:,1] - testy[:,:,1]) / np.max(np.abs(testy[:,:,1]), axis=1).reshape((-1,1)) 
hist = np.histogram(x.flatten(), np.arange(-0.2, 0.201, 0.001))[0] 
errors = np.append(errors, hist)

errors = errors.reshape((-1, 4, 400)) np.save("errors_new.npy", errors)
error = np.load("errors_new.npy")

print(error.shape)

# Print the error graph, a better result will lead to an error curve centralized to 0. 
plt.figure() 
plt.plot(np.arange(-20, 20, 0.1), error[0][0] / (np.sum(error[0][0]) \* 0.001))
plt.plot(np.arange(-20, 20, 0.1), error[0][1] / (np.sum(error[0][1]) \* 0.001)) 
plt.legend(["Third floor", "Roof"]) 
plt.xlim(-20,20) 
plt.xlabel("Normalized Error (
plt.ylabel("PDF") plt.title('Training Set')

plt.figure() 
plt.plot(np.arange(-20, 20, 0.1), error[0][2] / (np.sum(error[0][2]) \* 0.001)) 
plt.plot(np.arange(-20, 20, 0.1), error[0][3] / (np.sum(error[0][3]) \* 0.001))
plt.legend(["Third floor", "Roof"]) plt.xlim(-20,20)
plt.xlabel("Normalized Error ( 
plt.ylabel("PDF") 
plt.title('Testing Set')
```

Lines 130-146 are used to generate the normalized error curves of
different floors. The errors of all the response cases and time steps
are collected and the distribution can be plotted. The more the
distribution is centered around 0.0, the better the model.
