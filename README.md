# Experimental COVID-19 Prediction US

## Data processing
The original data is `time_series_covid19_confirmed_global.csv` from CSSEGISandData. 
The data contains `Province/State` field, and this field is compressed into `Country/Region`. 
Then, the countries of the top 10 number of confirmed cases are selected and transposed as `main DataFrame`.  
  
The dates of top 10 countries who reached first 500 or more cases are obtained as `From 500 Day`. 
Then, the time series of each country are normalized by the method below.  
Notice that China and Korea's Data is just used `Row ← Row/MaxOfCurrData` to normalize since the curves of China and Korea are long enough to reach a reasonable maximum value (not needed to estimate).

* Normalization Method 1  
`Ratio`  
`= MaxNumberOfCurrData/China'sData[CurrentDay-500Day]`  
`= MaxNumberOfCurrData/China'sData[DiffDaysFrom500OfCurrData]`  
`= MaxNumberOfCurrData/China'sNumberWithDiff`  
`EstimatedMaxOfCurrData = Ratio*China'sMax`  
`Row ← Row/EstimatedMaxOfCurrData`  

* Normalization Method 2  
`EstimatedMaxOfCurrData = MaxOfLogisticFunc(From Logistic Method)`  
`Row ← Row/EstimatedMaxOfCurrData`  

## Model Training
The model is trained by using China's Data, and the validation data is Korea's Data. Repeatly train `70` epochs, and get `- loss: 0.0023 - val_loss: 0.1351`. 

## Result Analysis
Use the model to evaluate the time series of each country. In each iteration, move the time series to left, and test the `Best Fit Day`. Move the curve (time series) of each country to the best position, and draw the figure below. 
  
Assuming that each country's time series obey the same pattern as China's Data, the slowing down date of the US can be predicted. The Day is `Stage Day`.  
  
### Normalization Method 1 (total MSE=0.010943, Iran excepted)  
<img src="./result.png">  
  
3/28: The actual increasing speed is slightly slower than the curve predicted at 3/26.  
  
### Normalization Method 2 (total MSE=0.007158, Iran excepted)  
<img src="./resultlogi.png">  
  
## Logistic Method  
Use `curve_fit(logistic_func, x, y)` to fit each country's time series. The Day is `From 500 Day`.  
`Mid`: sigmoid's midpoint  
`Max`: curve's maximum value  
`Lrate`: logistic growth rate  
  
Assuming that each country's time series obey the logistic function (fitted by itself), the 
future curve can be predicted.
  
<img src="./logi.png">  
  
3/28: The logistic rate keep the same, and more time to reach the maximum value.  
  
## Reference and Acknowledgment  
CSSEGISandData, https://systems.jhu.edu/, https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data
  
## Disclaimer  
All information, content, and material of this project is for informational and academic purposes only and are not intended to serve as a substitute for the consultation, diagnosis, and/or medical treatment of a qualified physician or healthcare provider.