---
permalink: /_alphatrading/module_livetrading
title: "Live Trading module"
toc: true
toc_label: "Live Trading module"
toc_sticky: true
---

The Live Trading module is one of the main features of the Alpha-trading python library. It allows to run live and performance track trading systems developped 
thanks to the Simulation module. It also allows to share trading signals and live reinforce trading strategies. 

# 1. Trader module 



## 1.1 Presentation of the tool 

The live trading module of the Alpha-Trading tool has been developped to make run trading strategies developped and backtested thanks to the 
integrated backtester tool without needing to modify one line of code from backtesting to live trading. One of the main features of the live trading module 
is to allow the user to have choice between a large panel of brokers. The trading tool doesn't interact directly with the broker but trading stations 
softwares which allows interaction with algorithms via socket connexion.  

Note: This tool has been designed to work on Forex, Stocks, CFDs and Indices markets and for mid to low-frequency trading strategies. It is not suitable for 
high frequency trading strategies. 

## 1.2 Features 

The Alpha-Trading tool live trading module embeds some important features: 
- **No-code from backtest to live** 
- **Multi-broker connexion** 
- **100% customizable logging system** 
- **Crypted signal sharing system (soon)** 
- **Simulation/Performance tracker (soon)**

The above features are detailed here below. 

### 1.2.1 From backtest to live

Alpha-Trading backtest and live trading modules have been designed in a such way that any trading strategy is compatible with both module. This 
way one can run live the trading strategy right after backtest without changing any live of code. Only external trackers systems will need to be adapted 
to the environment change. This can be done using a ```STRATEGY``` class attribute ```self.simulation = True/False```. 

### 1.2.2 Multi-broker connexion 

The live trading Alpha-Trading module has been designed in a such way that any trading strategy is compatible with a large panel of brokers. For 
instance the tool is compatible with IBKR and all the brokers working with the MT4 trading plateform. Depending on your needs, the list will be 
extended in the future. 

### 1.2.3 100% Customizable logging system 

The live trading Alpha-Trading module embeds a 100% customizable logging system which records every action made by the trading strategy and responses 
from the trading plateform. This way, the user can register absolutely any detail of the live/paper trading session and track for bugs, ineficiencies... . 
Moreover, this tool can be used to track for alternative data and search for correlations which can be used to reinforce the trading strategy. The 
logging system writes a SQLite database where a default ```log``` table is created by the system and any other table can be created by the user. 

### 1.2.4 Crypted signal sharing system (soon...) 

The live trading module of the Alpha-Trading tool embeds a low latency (< 1s) trading signal sharing functionality. There are two ways to use this 
module: 
- Signals flux generation (text and images) from the trading system to a Telegram conversation. 
- Signals exchange between two Alpha-Trading live trading instances using Slack servers. 

This tool also includes an encryption-decryption module. 

### 1.2.5 Simulation/Performance traker (soon ...)

In order to allow an efficient and performant live trading tracking without needing to send portfolio state requests to any brokers involved in the 
trading system, the Alpha-Trading tool embeds a live tracking module. This module creates a parallel backtest instance which simulates the trading strategy 
using live trading data. It allows to respond to two different needs: 
- It allows to verify that the live trading behaviour of the trading strategy is exactly the same as the one from simulations. 
- If applicable, it allows to track the live strategy performance and to correlate it with alternative data 
  selected by the user in order to reinforce the trading strategy 

## 1.3 Structure and components


## 1.4 Relevant links 


# 2. Tracking module

Soon ... 