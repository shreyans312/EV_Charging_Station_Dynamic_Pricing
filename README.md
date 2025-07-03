# Dynamic Pricing for EV Charging Stations using Demand Forecasting and Optimization

This project uses a time series demand forecasting via LSTM model and optimisation using Gurobi to maximize revenue under the price elasticity constraints.

## Limitations of Existing Systems
Commercial EV charging stations face several critical limitations that reduce efficiency and profitability
Thus, optimizing electricity pricing at public charging stations is critical to:
- **Balance demand fluctuations**
- **Maximize revenue**
- **Enhance customer satisfaction**

## Dataset
I have used the **UrbanEV dataset**[1] which includes:
- `inf.csv`: Important information of the charging stations, including coordinates and charging capacities.
- `occupancy.csv`: Hourly EV charging occupancy (busy count) in certain stations.
- `duration.csv`: Hourly EV charging duration in specific stations (Unit: hour).
- `volume.csv`: Hourly EV charging volume in specific stations (Unit: kWh).
- * `e_price.csv`: Electricity price for specific stations (Unit: Yuan/kWh).
- `s_price.csv`: Service price for specific stations (Unit: Yuan/kWh).
- `weather_airport.csv`: Weather data collected from the meteorological station at Bao'an Airport (Shenzhen).
- `weather_central.csv`: Weather data collected from Futian Meteorological Station located in the city centre area of Shenzhen.
- `weather_header.csv`: Descriptions of the table headers presented in `weather_airport.csv` and `weather_central.csv`.

## Model Structure
### 1. LSTM based Demand Forecasting
- Zone-wise models trained with:
  - log transformed 'volume'
  - selected relevant features only using a random forest regression
  - created additional features like power
- **Neighbour-aware features**: used average `volume` and `duration` from adjacent zones as additional features
- Scaled the features to handle the skewness
- Trained each zone with early stopping and RMSE tracking. Model stored using `joblib` along with all the scalers and feature sets zone-wise.

**Structue of LSTM Model:**
![image](https://github.com/user-attachments/assets/3b05e2ac-6af9-4540-80c4-23374f0914c6)

**Innovation:**
- Dense layers with ReLU activation for non-linear demand mapping
- Predicts future demand using past 72 hours data to capture hourly trends and also include any weekday effect

**Training Results**
- `\models\training_log.csv` has the zone-wise RMSE loss for each model
- Each zone has also a forecast plot (First 100 sequences) on the validation dataset stored
Eg: Zone 998:
RMSE: `train_loss= 19.942, validation_loss= 6.863`

![Uploading zone_998_forecast_plot.png…]()

## 2. Elasticity-aware Dynamic Pricing Optimization
- 

## References
[1] >Li, H., Qu, H., Tan, X. et al. (2025) UrbanEV: An Open Benchmark Dataset for Urban Electric Vehicle Charging Demand Prediction. Scientific Data. [Paper in Spring Nature](https://doi.org/10.1038/s41597-025-04874-4)
