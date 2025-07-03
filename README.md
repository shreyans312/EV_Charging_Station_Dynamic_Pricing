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
- `e_price.csv`: Electricity price for specific stations (Unit: Yuan/kWh).
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

![zone_998_forecast_plot](https://github.com/user-attachments/assets/c0dc0835-40e4-4a2c-b978-51640678ecdc)

## 2. Elasticity-aware Dynamic Pricing Optimization
- Currently implementation consists of hourly elasticity estimates [2]
  `hourly_epsilons = {
    0:  0.15, 1: 0.25, 2: 0.25, 3: 0.3, 4: 0.35,
    5:  0.4, 6: 0.5, 7: 0.7, 8: 0.8, 9: 0.9,
    10: 0.8, 11: 0.7, 12: 0.6, 13: 0.7, 14: 0.75,
    15: 0.8, 16: 0.85, 17: 1.0, 18: 1.1, 19: 1.0,
    20: 0.9, 21: 0.8, 22: 0.5, 23: 0.3
  }`
  The current dataset lacks the variation in electricity prices therefore making elasticity extraction from data inaccurate and difficult. Current approach is based on the following assumptions:
  - **Low elasticity** (0.1–0.4, hours 0–5): These overnight/early morning hours align with times when EV owners typically charge near home and after returning from work. Low elasticity suggests users are less responsive to price changes, which is reasonable as overnight charging is convenient and often coincides with lower baseline electricity prices.
  - **Moderate elasticity** (0.5–0.9, hours 6–16): These hours cover morning to late afternoon and late evening, where users may have flexibility to adjust charging based on price incentives. This aligns with the TOU pricing strategy to encourage demand shifting.
  - **High elasticity** (1.0–1.1, hours 17–19): The peak evening hours show the highest elasticity, indicating users are highly responsive to price changes. This supports the goal of reducing peak load through higher TOU prices during these hours.
 
## 3. Optimisation Objective and Framework
I have implemented a Gurobi based optimisation solution that uses the price elasticity model and also a penalize peak demand charges

**Objective function**

![image](https://github.com/user-attachments/assets/ec4354df-c9cc-4e38-acb0-2f31f48bee24)

**Where:**
- `p_t`: Optimized price at time `t`
- 's_t': Static baseline price at time `t`
- `D_t`: Forecasted demand at time `t`
- `e_t`: Price elasticity at time `t`
- `lambda`: Penalty coefficient to discourage large price deviations
- `T`: Total time intervals (eg: 72 hours)




## References
[1] >Li, H., Qu, H., Tan, X. et al. (2025) UrbanEV: An Open Benchmark Dataset for Urban Electric Vehicle Charging Demand Prediction. Scientific Data. [Paper in Spring Nature](https://doi.org/10.1038/s41597-025-04874-4)
[2] >Kuang, H., Zhang, X., Qu, H., and You, L., and Zhu, R. and Li, J. (2024). Unravelling the effect of electricity price on electric vehicle charging behavior: A case study in Shenzhen, China. Sustainable Cities and Society. [DOI](https://doi.org/10.1016/j.scs.2024.105836)
