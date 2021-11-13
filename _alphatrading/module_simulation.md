---
permalink: /_alphatrading/module_simulation
title: "Simulation module"
toc: true
toc_label: "Simulation module"
toc_sticky: true
---

The Simulation module is one of the main features of the Alpha-Trading python library. It allows both to build, test trading strategies and build and test 
more complex models that can be used to improve strategies, analyse financial data, stress-test strategies and build risk-management systems around signal 
generators. 

# 1. Backtester module

## 1.1 Presentation of the tool 

I initialy developed the backtest tool in order to test my own trading strategies in an environment 100% controllable, fully customizable and designed to 
garantee absence of look-ahead bias during strategies backtest. It has been structured in a such way that simulations are as much as possible close to the 
reality of a *live-trading* environment. The trading process on financial market can be extremely complex and cover a wide range of possible actions. Because it is 
impossible at a *human-scale* to think about every way to trade on financial markets, I tried to develop a solution that reponds to the *standard way of trading* on 
financial markets. 

That is why for instance, the backtester is designed to simulate trading strategies dedicated to Forex, Stocks and CFDs markets. Simulated trading orders have a 
*bracket* shape. 

```python 
# Structure of the function allowing to place trading orders
placeOrder(
    symbolName,
    action     = "long",      # "long" or "short"
    orderType  = "MKT",       # Order kind : "MKT", "MIT", "LMT"
    volume     = 0.1,         # Volume 
    stoploss   = None,          
    takeprofit = None, 
    lmtPrice   = None, 
    auxPrice   = None) :

```

This structure is actually sufficient for most of trading strategies. However, the backtester includes some features that make it unique! 

## 1.2 Features 

### Historical data correction 

The backtest tool includes a tool allowing to correct and prepare financial data for a high-precision and performance backtest. This tool embeds the 
following functions: 

- **A missing data filling model**: Very often financial data presents gaps at different times and for different reasons. It can be because of a lack 
of recording quality from the provider or simply because markets are closed during the weekends. In the backtest module of the Alpha-Trading library, 
we fill every gap existing in the data (at the resolution time-scale) according a data-filling model. For instance missing data is replaced by 
constant data but the user can create his/her own missing data filling model. Soon I will propose a missing data filling model based on 
GANs.

- **Time alignement of data**: Depending on the location of the data provider, the time zone associated to data is not corresponding to your neither the 
one of the exchange corresponding to the retrieved data. In the backtest module of the Alpha-Trading library, we shift the historical data in the 
time zone corresponding to the one where the exchange is located. This way we ensure (for most of data) that if the market close during the night, it will 
close the night of the day $$N$$ and re-open the morning of the day $$N+1$$. This facilitates the process of data re-sampling (see two points below). 

- **Exchange state data**: Because the Alpha-Trading backtest tool tends to fill all the time gaps in the data. One need to provide to the user the information 
about the state of the market at each timestep. This functionality allows for example to a user to specify dates at which the exchange was closed for a 
non-cyclic reason, so it prevents the algorithm to operate over periods which have never been existing and consequently increase the precision of the backtests. 

- **Data resampling tool**: Often, trading strategies involves indicators based on a data with a large time-scale resolution such as H1 for example. But to 
have a great backtest precision and illustrate all possible situations in real life, one needs to backtest using small time-scale resolution data such as 
M1 for example. There are usually two solutions to overcome this problem: the first one consists in downloading M1 data and H1 data from the same asset but this 
can lead to data inconsistencies for a lot of reasons, the second solution consists in building H1 data from the M1 data, and a lot of resampling tools exists to 
do so (such as pandas.resample for example), but all these tools doesn't take in account the structure of an exchange day range and re-sample data in a 
random way. In the Alpha-Trading backtest tool, the data resampling tool takes in account the structure of a trading day and resample data accordingly so that 
after resampling M1 data into H1, user will automatically have access to H1 data candles EXACTLY similar to what he/her would obtain by retrieving H1 data 
from the broker. 

- **Data synchronisation tool**: Resample data before a backtest in usually a dangerous operation because it open the doors to a possible look-ahead bias. 
In the Alpha-Trading backtest tool, we propose a (multi-scale and multi-asset) synchronization tool which takes care that user cannot have access to future 
data from resampled data. For example the 9:00 H1 candle data will be available to the user only at the same time step where the 10:01 M1 candle data will be 
available. The synchronization tool also allows to synchronize data from different assets. Lets say you want to create a statistical arbitrage strategy 
using GOOG and APPL data but your GOOG data goes from 2010-01-10 12:30 to 2020-10-15 22:35 and your APPL data goes from 2009-09-17 04:25 to 
2019-02-03 15:42. After filled every gaps and resampled both data according your needs, the algorithm will cut the edges of your data and index-align them 
together in a such way that your final dataset time start will be 2010-01-10 12:30 and your final dataset time end will be 2019-02-03 15:42. Moreover 
for a given index, the corresponding time for APPL and GOOG will be the exact same, this no matter what is the current exchange state of each instrument. 

### Contract properties simulation 

Often differences between backtest and live trading can comes from the absence of considerations of constraints defined by the contract selected for 
trading. To overcome this potential simulation bias, the Alpha-Trading backtest tool includes a tool to simulate contract properties as described 
below: 

- **Contract size**: Depending on the Broker and the instrument, a lot can have different size an hence necessite a minimal amount of liquidity to 
be traded. 

- **Currency**: (Not implemented yet!). One can define instruments based on a currency and trade them on another currency thanks to a supplementary 
dataset allowing to convert your currency in the instrument's one. 

- **Contract type**: CFD, Forex, Stock, CFD-Index. Depending on the contract type, the margin is calculated differently. These models allow to calculate the 
real returns you would get depending on the instrument type. This property is important in the case where you backtest a trading strategy working on two different kind of contracts for example. 

- **Required margin percentage**: To ensure your solvability to any market event, brokers often define a minimal margin percentage required to trade an instrument. 
In a backtest performed without a such constraint one can fall in the trap where simulated orders have been placed and won't have been in the reality of trading. 
In the Alpha-Trading tool we take in account a such constraint and we let the user define his/her own required margin percentage. This value is specific to each 
involved contract. 

- **Execution mode**: (Not implemented yet!). Depending on the state of the market or for some instruments, the trades execution mode can be different. The common 
execution mode is time to market, but one can be different and be *close only* for example. 

- **Volume constraints**: For obvious reasons, every Broker imposes volume constraints. These are in terms of volume size : minimal volue $$\rightarrow$$ maximal 
volue, or in terms of volume discretization : number of digits. To not consider such constraints can lead to consider trading order that would have been refused 
in the reality. In the Alpha-Trading backtest tool we take in account this constraint. 

- **Fees**: (Not implemented yet!). Depending on the broker and the trading status, some fees can apply. These fees can have a non-negligible impact on the 
strategy's return. 


### Portfolio constraints simulation 

The Alpha-Trading tool allows to simulate one or more portfolios and simulate constraints on them. While each backtester propose to ajust 
standard portfolio parameters such as: initial income, leverage, currency (not implemented yet), we worked to the implementation of 
constraints on the portfolio defined at the begining of the simulation and such that if the portfolio meets one of them, trading is not 
allowed anymore. This is an important point which can happen in the reality but which is often not considered in backtest mode. 
The available constraints are: 

- **Position type**: The market can be constrained to only *long* positions for example, or *long and short*, or rarely (probably never) only *short*. This means that if your trading strategy wants to place a non-allowed order, it will be refused by the simulator. 
  
- **Margin call treeshold percentage**: If your available margin becomes lower than the margin call treeshold, your strategy won't be able to open any position as long 
as you do not increase your margin. 

- **Margin Minimum percentage**: If your available margin becomes lower than the minimum margin treeshold, the simulation will start to close the position in which 
you are the most losing, check again your margin, and either close another losing position if your margin is still below the treeshold, either nothing. 

- **Minimum balance**: Below the minimum balance, trading is not allowed anymore. 
  
- **Maximum profit**: Above a certain amount of positive profit, trading is not allowed anymore. This constraint has to be used as a signal that something 
may be wrong in the way that the trading strategy is backtested and the profit made is abnormally high. 

- **Maximum dradown**: Above a certain drawdown, your trading strategy may be dangerous. Trading won't be allowed anymore. Correct it. 
  
- **Maximum consecutive loss(gain)**: Above a certain summed value of consecutive losses(gains), trading is not allowed anymore. These constraints have to be 
used as detectors of a wrong backtest way of the trading strategy on any other strange (not realistic) behaviour. 

- **Maximum number of consecutive gain**: Same constrain as above but the metric is the NUMBER of gains and not the total summed gains value. 

### Multi-Instruments backtest 

The Alpha-Trading backtest module, thanks to its features of filling missing data and data synchronization, is able to involve more than one instrument in a unique 
backtest. This feature opens the doors to backtests of statistical arbitrage like strategies and any trading strategy dedicated in trading simultaneously on 
multiple instruments. Event if it is possible to backtest a multiple-instrument strategy on each instrument separately and re-construct the results after both 
backtest, here, we allow to do it in a totally mastered environment where each small bias or default of the strategy will be detected. 

### Multi-Portfolio backtest 

The Alpha-Trading backtest module is structured in a such way that for a given strategy it allows to simultaneously backtest it on multiple portfolios and with 
multiple strategy parameter sets. To do so the program create couples (portfolio instance, strategy instance) and each couple can be backtested with three different methods: sequencialy, sequencialy (step-by-step), or in parallel (calculations distributed over computer cores). 

### 100% Customizable logging system 

The Alpha-Trading backtest module embeds a 100% customizable logging system which records every action made by the simulator or every action made by the trading strategy 
specified by the user. This way one can register absolutely any detail in the simulation and use it either to find bugs in the strategy, in the simulator, 
or to improve the strategy through learning on meta-labeling for example. The logging system write create SQLite databases where a default *log* table is 
created by the simulator and any other table can be created by the user. 

### Sub-candle simulation modding 

In order to improve the quality of the simulation, and to explore all the space of possibilities, the simulation can run the price data according different modes: 

- **close only**: This mode only take in account the *close* values of each candle 
- **ohlc**: This mode scrolls each candle price in the following order: *Open*, *High*, *Low* and then *Close* 
- **ohlc random**: (Not implemented yet!) This mode corresponds to the above mode but *High* and *Low* order is randomized. 
- **ohlc GAN**: (Not implemented yet!) After observing data behaviour at tick scale, this mode allows to generate data at sub candle space in order 
to simulate very small price fluctuations. 

### Data perturbation models (soon...)

## 1.3 Structure and components 

## 1.4 Components importation 

## 1.5 Relevant links 


# Models 

(Soon...)