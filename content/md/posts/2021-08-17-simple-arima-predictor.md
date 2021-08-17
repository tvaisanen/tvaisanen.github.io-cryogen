{:title "Simple ARIMA Predictor"
 :layout :post
 :tags  ["Python" "Data Science" "Pandas" "Statsmodels" "Numpy"]}


```python
import pandas as pd
import numpy as np
from statsmodels.tsa.arima.model import ARIMA
import seaborn as sns
import matplotlib.pyplot as plt

sns.set_theme(style="darkgrid")
```

## Create Random Sample Data


```python
sample = pd.DataFrame(columns=["data"], data=np.random.normal(0,1,250).cumsum())
sample.plot(figsize=(15,5))
```

![png](img/2021-08-17-simple-arima-predictor/output_3_1.png)


## Predict Values

Use `lookback` as training length for the predictor.


```python
lookback = 30
predictions = np.empty(sample.shape[0])

for i in range(lookback, sample.shape[0]):
    x = sample.data.values[i-lookback:i]
    # Initialize random walk ARIMA model
    # https://otexts.com/fpp2/non-seasonal-arima.html
    mod = ARIMA(x, order=(0,1,0))
    # Fit data
    res = mod.fit()
    # Predict the the value
    predictions[i] = res.forecast(1)

sample['pred'] = predictions
```


```python
fig,ax = plt.subplots(1,1,figsize=(15,5))

ax.axvspan(       0,        lookback, 0, 1, alpha=0.1, color='orange')
ax.axvspan(lookback, sample.shape[0], 0, 1, alpha=0.1, color='green')

sample.pred[lookback:-1].plot(ax=ax, linestyle='dashed')
sample.data.plot(ax=ax)
plt.show()
```



![png](img/2021-08-17-simple-arima-predictor/output_6_0.png)



## Evaluate Prediction


```python
from sklearn.metrics import mean_squared_error

mean_squared_error(sample.pred[lookback:-1], sample.data[lookback:-1])
```


    0.9786439154503027
