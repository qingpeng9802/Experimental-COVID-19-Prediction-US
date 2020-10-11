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
The model is trained by using China's Data, and the validation data is Korea's Data. Repeatly train `70` epochs, and get `- loss: 6.9204e-04 - val_loss: 0.1572`. 

## Result Analysis
Use the model to evaluate the time series of each country. In each iteration, move the time series to left, and test the `Best Fit Day`. Move the curve (time series) of each country to the best position, and draw the figure below. 
  
Assuming that each country's time series obey the same pattern as China's Data, the slowing down date of the US can be predicted. The Day is `Stage Day`.  
  
### Normalization Method 1 (total MSE=0.0203977, Iran excepted)  
<img src="./figs/result.png">  
  
3/28: The actual increasing speed is slightly slower than the curve predicted at 3/26.  
4/5: The actual increasing speed is 1 day later than the curve predicted at 3/28.  
4/13: The actual increasing speed is 4 day later than the curve predicted at 4/5, which means the increasing speed is lower quickly (but probably this means the method has a larger error near to the midpoint of the curve according to Method 2 result).  
4/21: The actual increasing speed is still 4 day later than the curve predicted at 4/13. Continuous predict delay might means that the model is about 2x faster than actual speed. Thus, the slowing down date of the US is about 8+4=12 days later, which is 5/3.  
  
### Normalization Method 2 (total MSE=0.0291249, Iran excepted)  
<img src="./figs/resultlogi.png">  
  
4/5: The actual increasing speed is 4 day later than the curve predicted at 3/28, which means the increasing speed is significantly lower than before, and the social distancing is working.  
4/13: The actual increasing speed is 2 day later than the curve predicted at 4/5, which means the increasing speed is still keeping lower (the curve will be slowing down soon based on the curve).  
4/21: The actual increasing speed is still 2 day later than the curve predicted at 4/13, which means this model might be 25% faster. Bias the predict by 6+2=8 days.

By 3/28, the slowing down date of the US is about 4/15.  
By 4/5, the slowing down date of the US is about 4/18.  
By 4/13, the slowing down date of the US is about 4/21.  
By 4/21, the slowing down date of the US is about 4/29. 
  
## Logistic Method  
Use `curve_fit(logistic_func, x, y)` to fit each country's time series. The Day is `From 500 Day`.  
`Mid`: sigmoid's midpoint  
`Max`: curve's maximum value  
`Lrate`: logistic growth rate  
  
Assuming that each country's time series obey the logistic function (fitted by itself), the 
future curve can be predicted.
  
<img src="./figs/logi.png">  
  
3/28: The logistic rate is slightly lower, and need more time to reach the maximum value.   
4/5: The logistic rate become reasonable compared to other countries. However, due to the initial speed is too high, the final cases might reach 50,0000+.  
4/13: The logistic rate is lower and more close to other countries. The final cases might reach about 700,000.  
4/21: The logistic rate is at the same level compared to other countries. The final cases might reach about 900,000+300,000=1,200,000. (since current predict model is underestimated about 200,000 per 8 days, 300,000 is added by 12 days.)  
  
By 3/28, the slowing down date of the US is about 4/11.  
By 4/5, the slowing down date of the US is about 4/18.  
By 4/13, the slowing down date of the US is about 4/22.  
By 4/21, the slowing down date of the US is about 5/3.  

## Final Update
The initial idea is that since this virus is a new virus, we didn't have enough information about it, but we still want to simulate and project the case curve with limited data and information. Thus, we just assume that each country's time series obey the same pattern as China's Data, which is "pattern matching" method. However, we noticed that the Institute for Health Metrics and Evaluation (IHME) used similar method in their very early version, and through further observation, the accuracy of their prediction is very poor. Thus, we just gave up this model and idea. Moreover, due to the influence of multiple factors, the statistics of the number of positive cases are not accurate and stable in fact, so it is better to use the death cases to model. 
  
Currently, we think the best projection for COVID-19 is [covid19-projections.com](https://covid19-projections.com/), [Youyang Gu GitHub](https://github.com/youyanggu/covid19_projections). [More info](https://youyanggu.com/blog/six-months-later) after the website ends its projection after Nov 1.

## Reference and Acknowledgment  
CSSEGISandData, [CSSE](https://systems.jhu.edu/), [CSSE GitHub](https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data)
  
## Disclaimer  
All information, content, and material of this project is for informational and academic purposes only and are not intended to serve as a substitute for the consultation, diagnosis, and/or medical treatment of a qualified physician or healthcare provider.
