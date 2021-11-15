---
permalink: /_alphatrading/handson_simplebacktest
title: "Handson - A simple backtest"
toc: true
toc_label: "Hands-on"
toc_sticky: true
# classes: wide
---

## [Introduction](#introduction)

The Alpha-Trading backtest system is dedicated to algorithmic traders developers who only want to dedicate them to build their trading strategies in a Python 3 environment. In this tutorial, I will show you how to prepare your simulation, strategies and data files in order to get a smart framework in which you can completely focus on your strategy development. 

The Alpha-Trading Backtest System needs at least 3 files to run : 
* The historical data file(s) 
* The trading strategy file(s) 
* A simulation header file 

Both, the historical data file(s), the trading strategy file(s) and the Alpha-Trading Backtest System are imported in the simulation header file. Thanks to this smart structure, it is possible to add a lot of automating in your backtesting experience to perform a lot more complex simulations such as strategy parameters optimization, backtest a large number of assets/strategies ... so a potential huge gain of time. The python environment being very famous in data sciences, you can have access to a lot modules to analyze your simulation results and improve your backtest experience. 

## [Package installation](#package-installation) 

Let first install the Alpha-Trading package. Open a terminal and type : 

```shell
pip install q26-alphatrading
```

Or, if you want to work with the Alpha-Trading development package (what I suggest to you for instance), go to the [Github](https://github.com/LoannData/Q26_AlphaTrading) repository of the project and clone it to your local system according the following command 

```shell 
git clone https://github.com/LoannData/Q26_AlphaTrading.git
```

Open a new terminal in the Q26_AlphaTrading folder and now you can locally install the package using the ```pip``` command: 

```shell 
bash ./compile.sh 
sleep(1)
bash ./install.sh 
```

**Note**: If you want to contribute to the developpement of the project, you can also fork it to your own. 

## [Simulation file](#simulation-file)

Let's by start building a simple simulation file header. At first you have to create a python file with the name of your choice (I will call it simulation.py to make it simple) and put it anywhere on your system. 

### [Importations & paths](#importations--path)

I suggest first to import some useful modules in order to manage any objects the Alpha-Trading Backtest System manipulates. 

```python
# Usual modules importations 
import sys, os 
import numpy as np 
import pandas as pd 
import datetime as dt 
import matplotlib.pyplot as plt 
import pprint
import copy 
```

Now you are ready to import the different Q26 Backtest System classes. 

```python 
# Alpha-Trading Backtest System class importations 
from alphatrading.simulation.backtester.symbol     import SYMBOL
from alphatrading.simulation.backtester.portfolio  import PORTFOLIO 
from alphatrading.simulation.backtester.data       import PRICE 
from alphatrading.simulation.backtester.data       import PRICE_TABLE
from alphatrading.simulation.backtester.simulation import SIMULATION
```
Let's describe a bit the role of each imported class : 
* The SYMBOL class allows to provide a symbol identity to the simulator. For example, if you want to backtest your trading strategy on XAU.USD symbol, you will need to provide some information such as the contract size, the minimum margin percentage, the margin request method (~ type of product) ... 
* The PORTFOLIO class simulates your trading portfolio but also the trading rules. This means you can define your initial balance but also a leverage value and a bunch of others parameters which can play an important role in the simulation. Moreover, PORTFOLIO member functions simulate with precision your interactions with the market and with the broker. 
* The PRICE and PRICE_TABLE classes are used to store and prepare the historical data. They propose a lot of functionalities and allow to backtest number of assets simultaneously. 
* The SIMULATION class contains all the important parameters and functions to perform a smart simulation.   

To get more information about the attributes and the member functions of these classes, I suggest you to check the detailed code documentation [here](/_alphatrading/html/namespacealphatrading_1_1simulation_1_1backtester.html). 


### [Data preparation](#data-preparation)

Let's start with the data preparation. At first, we have to specify the relative or absolute path to our datafile. Note that for instance the Q26 Backtest System can only read .csv datafiles. So if needed, you can reformat your datafile before integrating it inside the system. But don't worry, the majority of the data providers allow to easily get .csv datafiles. 

```python 
# We define the path to the dataset
path  = "./"
path += "exampleDataFile.csv"
```

Then, let's create a [PRICE](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html) object.  

```python 
# We create an object PRICE and give it a name 
price = PRICE("EUR.USD") 
```

The name of the object (here EUR.USD) is extremely important and have to be carefully chosen because it will be used by the trading strategy file(s) to get the data and place orders. 

Now, we have to explain to our system how to read and interpret the datafile we have provided. We start by linking the columns of our datafile with the data that we are going to read. Here is a piece of our datafile : 

| date       | hour  | open     | high     | low      | close    | vol |
|------------|-------|----------|----------|----------|----------|-----|
| 2020.01.01 | 17:00 | 1.121200 | 1.121210 | 1.121170 | 1.121200 | 0   |
| 2020.01.01 | 17:01 | 1.121060 | 1.121350 | 1.121060 | 1.121350 | 0   |
| 2020.01.01 | 17:02 | 1.121360 | 1.121390 | 1.121360 | 1.121390 | 0   |
| 2020.01.01 | 17:03 | 1.121350 | 1.121350 | 1.121200 | 1.121220 | 0   |
| 2020.01.01 | 17:04 | 1.121220 | 1.121250 | 1.121220 | 1.121250 | 0   |
| 2020.01.01 | 17:05 | 1.121250 | 1.121270 | 1.121250 | 1.121270 | 0   |
| 2020.01.01 | 17:06 | 1.121270 | 1.121270 | 1.121270 | 1.121270 | 0   |

To explain to the system how to read it, let's use the command : [setColumnsTitle](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html#a1eea86523468850f44d8c2557132fb45).

```python 
# We associate the name of the columns in the datafile with 
# the properties of the PRICE object.
# In the specific case of the exampleDataFile.csv dataset, we do not have 
# any information of ask/bid price, so we consider by default that ask = bid 
# in the reading process. This will lead to a spread = 0.
price.setColumnsTitle(askOpen        ="open", 
                      askHigh        ="high",
                      askLow         ="low",
                      askClose       ="close", 
                      bidOpen        ="open",   
                      bidHigh        ="high",
                      bidLow         ="low",
                      bidClose       ="close",
                      dateFormat     ="%Y.%m.%d %H:%M", 
                      volume         ="vol",
                      splitDaysHours =True,  
                      date           = None,
                      days           ="date", 
                      hours          ="hour")
```

Let's note some points : 
* Our historical data don't provide bid/ask information. But the Alpha-Trading Backtest System requires it. As made in the code, you can read ohlc columns two times, one for the bid and one for the ask, this will just lead to a 0 spread. 
* In some datasets, the time is presented in a unique column while in some others (like in this dataset), the time is splitted between days and hours. The setColumnsTitle has a parameter for that : splitDaysHours. If False, you have to provide the name of the time column in the parameter date. If True, you can let date to None and you have to provide the name of the columns corresponding to days and hours. 

Then, we can read the datafile with the command [read](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html#a523730f8f49f1bfa0f7bbb607be45ce5). 

```python 
# We read the data
price.read(path)
```

When this has been done, we have to specify the timeframe of the data we are going to simulate using the method [setBaseTimeframe](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html#a3d32bd897cd40ac0ec85de6febf05f74). 

```python 
# We define the timeframe associated to the loaded data 
price.setBaseTimeframe(timeframe = dt.timedelta(minutes = 1))
```

And we have to prevent the missing data in our datafile by using the following function: [fillMissingData](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html#a0c2cfd3bd19b38ea6f35c39b814b5260). 

```python 
# We fill the missing data according to a data filling model 
price.fillMissingData(model = "constant")
```

After that, we have to specify the agenda properties of the associated exchange place and link it properly with the data we are going to simulate. 

```python 
# If necessary, we can shift our data 
price.shiftMarketTime(timeshift = 0)
# We can define the time zone in which data have been scraped (UTC+...)
price.dataTimeZone   = 0
# We can define the timezone in which the market is located (UTC+...)
price.marketTimeZone = 0
# We can define market opening/closing hours in the format "HH:MM"
# Note : If the market never close, the opening hour is "00:00"
#        while the closing hour is "24:00".
price.marketOpeningHour = "00:00"
price.marketClosingHour = "24:00"
# If the market has a mid day break or others breaks during the day 
# write it in the format : "HH:MM-HH:MM"
price.marketLunch    = None
marketBreakList      = list()
# Days of the week the market is open - 0 : Monday -> 6 : Sunday 
price.daysOfWeek = [0, 1, 2, 3, 4]
```

And finally, according to the time details we provided, the algorithm will generate for every piece of time data an information about if the market is open of not. This 
happen in the function [setMarketState](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html#ac2d310c0f53aab16c269de1fb82f62df) 

```python 
# Function that defines if the market is open/close 
price.setMarketState() 
```

Now we have prepared the data dedicated to the simulation. 

However, in our trading strategy, we are not especially going to use such a low timeframe data, we may for example want to use H1 data. One way to get this new data is to re-sample it directly from the trading strategy. But it is numerically very costly and it destroys the algorithm trading environment experience of the user. That's why we propose a smarter solution. 

Before running any backtest, you prepare all the re-sampled data you will need in your strategies. For example in this tutorial we will need H1 data. To get this data, we create a copy of our initial EUR.USD data and we re-sample it to the H1 timeframe with the method: [resampleData](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE.html#aad5573e679cb8310e9e005caa1b6efd6). 

```python 
# From the price object, it is possible to define another exact same object 
# thanks to the deepcopy function
price_H1 = price.createCopy()
# Here this dataset object is resampled to be used in the simulation. 
# The resampling process is exactly the same as you can see 
# on trading platforms. 
price_H1.resampleData("01:00", name = "EUR.USD")
```

We get a second PRICE object called price_H1. Note that it is extremely important to keep the exact same name if we want to retrieve historical data in the standard way in the trading strategy file(s).  

We can finally agreggate our data in a [PRICE_TABLE](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1data_1_1PRICE__TABLE.html) object and synchronize it. 

```python 
# We generate our data table which will be involved in the simulation 
table = PRICE_TABLE([price, price_H1]) 
# In the case where we have more than 1 not resampled price, 
# the synchronize function will be necessary. 
table.synchronize()
```

Note that here the synchronization function is useless because it only synchronizes "base" data (used for simulation) and not re-sampled data. 

We are finally done with the data preparation ! It was the hardest part ! 

### [Symbols](#symbols)

Let's implement our [SYMBOL](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1symbol_1_1SYMBOL.html) object. It is simple as that : 

```python 
symbol = SYMBOL(symbolName              = "EUR.USD",
                contractSize            = 100000, 
                marginCurrency          = "USD",    # Can be any existing currency (only USD is working for instance)
                profitCalculationMethod = "Forex",  # "CFD", "Forex", "Stock", "CFD-Index"
                marginRequestMethod     = "Forex",  # "CFD", "Forex", "Stock", "CFD-Index"
                marginPercentage        = 100, 
                execution               = "Market", 
                minimalVolume           = 0.01, 
                maximalVolume           = 100.0, 
                volumeStep              = 0.01, 
                precision               = 5,        # Price precision (3 means 1 point = 0.001)
                exchangeType            = "Point",  # "Point", "Percentage"
                exchangeLong            = 6.88, 
                exchangeShort           = 0.63)
``` 

Here we provide the "identity card" of the symbol we are going to simulate. 
Note that all the parameters are not used by the simulator for instance. I let you refer to the detailed doc for more details. Note also that the name of the symbol has to exactly correspond to the name of a PRICE object; here "EUR.USD". 

### [Portfolios](#portfolios)

We can then initialize our [PORTFOLIO](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1portfolio_1_1PORTFOLIO.html) object with all the usefull parameters. See the doc for more details. 

```python 
# We initialize our portfolio 
p = PORTFOLIO(initialDeposit                  = 100000,                # The initial client deposit 
              leverage                        = 30,                    # The leverage value (margin = initialDeposit*leverage)
              currency                        = "USD",                 # The currency 
              positions                       = "long & short",        # "long", "short" or "long & short"
              marginCallTreeshold             = 100,                   # If marginLevel < marginCallTreeshold : Warning (no more trading allowed)
              marginMinimum                   = 50,                    # If marginLevel < marginMinimum : Automatically close all losing positions 
              minimumBalance                  = 50000,                 # If balance < minimumBalance : No more trading allowed 
              maximumProfit                   = 100000,                # If balance - inialDeposit > maximumProfit : No more trading allowed 
              maximumDrawDown                 = 70,                    # If drawDown < maximumDrawDown : No more trading allowed 
              maximumConsecutiveLoss          = 50000,                 # If valueLossSerie > maximumConsecutiveLoss : No more trading allowed 
              maximumConsecutiveGain          = 50000,                 # If valueGainSerie > maximumConsecutiveGain : No more trading allowed 
              maximumNumberOfConsecutiveGains = 30)

# We add the symbol identity we created inside the portfolio object 
p.addSymbol(symbol)
``` 
And we add the symbol we prepared inside the portfolio via the function [addSymbol()](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1portfolio_1_1PORTFOLIO.html#a14e16799e708a6bea1cce5959816c5a6). 

### [Simulation](#simulation)

We can finally start to initialize our [SIMULATION](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1simulation_1_1SIMULATION.html) object by using the following instruction : 

```python 
# We initialize the simulation object 
sim = SIMULATION([p], table)
```

The SIMULATION object takes in account a list of portfolio (here containing only one portfolio) and the TABLE_PRICE object. 

We set some important attributes of the simulation. 

```python 
sim.subLoopModel = "close only"
sim.maxHstDataSize = 2000
sim.startIndex = 2000
# sim.stopIndex  = 2010
sim.logEvery = 1000
```

Some details : 
* The [sub-loop](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1simulation_1_1SIMULATION.html#aacf7dc6d4d6b01e42bbd2851baf9c961) model represent the way the algorithm will present the sub-candle scale data to the trading strategy file(s). For example here we selected the "close only" sub-loop models meaning that only close price of candles will be presented to the strategies. Another model is "ohlc standard". This means that for every base candle, the algorithm will sequentially present open, high, low and finally close price to the strategy file(s). 
* The maxHstDataSize corresponds to a buffer size of how many candle historical data can be available to trading strategies. This value has to be adjusted according the needs of the strategies and have to be minimized to optimize the calculation time. 
* The start/stop indexes corresponds to the index of base data where simulation starts/stops. The start index has to be higher or equal than the maxHstDataSize if you want to have enough available historical data in your trading strategy file(s). Note that here the stop index hasn't been specified. By defaut, it has an unfinite like value meaning that the simulation will go up to the end of the dataset.
* logEvery variable corresponds to frequency at which the algo prints a percentage and activates the [show()](#global-structure) function in the strategy file(s). 

Now we have to connect our simulation object containing data, symbol(s) and portfolio(s) with strategy trading file(s). We have to provide the aboslute or relative path and the name of the file without the .py extension. We finally use the .importStartegy() function to import the strategies. 

```python 
# Relative or absolute path to the strategy file
# and strategy class importation  
sim.strategyPath = ["./"]
sim.strategyFile = ["strategyExample"]
sim.importStrategy()
```

We can finally run the simulation according our prefered [running mode](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1simulation_1_1SIMULATION.html#a9ae1e9bffddf17c3d2ed8c2966c6fdf0) 

```python 
# Run of the simulation 
sim.run(mode = "linear")
```

This is done for the simulation header file. As you can see, this file has a large potential of personalization and optimization. 

**Note:** In the header file, you also have to configure the simulation logging system. Check the [paragraph](#logging-system-configuration) below to get more informations. 

## [Strategy file](#strategy-file)

Let's build our strategy file. First let's create a python file with the same name as provided to the SIMULATION object (strategyExample.py in our example) and the same path as provided in the SIMULATION object (same folder as the simulation.py file in our example).  

### [Global structure](#global-structure)

All the strategy files have to follow the same following structure (where the client parameters correspond to a PORTFOLIO object) : 

```python 
class STRATEGY : 
    """ 
    The STYRATEGY class is imported in the SIMULATION class. 
    
    Only 3 functions are important and executed in the backtester : 
        - __init__ : To declare the strategy object 
        - run      : Function called to each timestep of the simulation 
        - show     : Function called to each log step of the simulation 
    
    One can add any function in this class. This will not pertubate the 
    simulation. 
    
    run and show functions take in account a "client" parameter which 
    corresponds to the "PORTFOLIO" object as declared in the simulation
    header file. 
    """ 
    
    def __init__(self) : 
        
        return 
    
    def run(self, client) : 
        
        return 
    
    def show(self, client) : 
        
        return 
```

As every class, thanks to the __init__ function you can define attributes directly after importation in the simulation.py file if you need. You can also use these attrributes to store data and avoid calculations. The run(client) function will be called at each timestep of the simulation, this is the place where all the trading decisions have to be taken. The show(client) function will be called according the SIMULATION.logEvery frequency and can be used to plot some figures, print important informations or write data for example. 

You can see that with a such structure you have a very large flexibility as you can add functions inside the STRATEGY class, oustside on the same file or even import other python module. You can do everything you want !  
 
### [Trading functions](#trading-functions)

Inside the run(client) function, you have access to a bunch of client.functions() accessible [here](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1portfolio_1_1PORTFOLIO.html). 

### [Example : SMA crossover strategy](#example)

Let's see an example with a SMA crossover strategy. The python file is available in the repository as an example strategy. 
The advantage of the Q26 Backtest System is that here we only need to focus on trading and not on dataprice management and bias avoiding. 

First we start with the __init__ function : 

```python 
    def __init__(self) : 
        # Name of the symbol to trade 
        # Note : The names of the symbol used in the strategy object
        #        have to be the the exact same as the ones defined in 
        #        the simulation header file 
        self.symbolName = "EUR.USD"
        self.timeframe = 60
        
        
        # Moving average parameters 
        self.SMA1 = None 
        self.SMA2 = None 
        
        self.SMA1_period = 10 
        self.SMA2_period = 30 
        
        # Trading status memory 
        self.activePosition = None
        self.placedOrder    = None 
        
        #  Other options 
        self.showInfo = True 
        
        return 
```

We initialize the following attributes : 
* The symbol name, which is exactly the same as the name we used in the data preparation step and the symbol creation 
* The timeframe (60 minutes = 1 hour). We previously prepared a H1 dataset, so we can pretend to a such timeframe. 
* SMA parameters (period) and SMA list buffers.
* Trading status memory : do I have an active position ? Which one ? What is the last placed and executed order ? 
* A show info variable. 

Then we can establish our crossover strategy in the run function : 

```python 
    def run(self, client) : 
        
        # We ask for the last existing price 
        # Note : In case of some sub-loop models, 
        # it is possible that the last price doesn't correspond 
        # to the close price. To avoid an ahead bias, take care to 
        # only use the current price information (askprice/bidprice)
        # to take decisions and note the askclose/bidclose prices. 
        self.lastPrice = client.getLastPrice(self.symbolName)
        self.dataSet1  = None 
        self.dataSet2  = None  
        
        # If the market if open 
        if self.lastPrice.get("market state") == "open" : 
            # We ask for 2 datasets in H1 period 
            self.dataSet1 = client.getHistoricalData(self.symbolName, self.SMA1_period, 0, self.timeframe, onlyOpen = True)
            self.dataSet2 = client.getHistoricalData(self.symbolName, self.SMA2_period, 0, self.timeframe, onlyOpen = True)
            # We calculate 2 SMA indicators 
            if len(self.dataSet1.get("askclose")) > 0 : 
                self.SMA1 = SMA(self.dataSet1.get("askclose"), period = self.SMA1_period, offset = 0)
            if len(self.dataSet2.get("askclose")) > 0 : 
                self.SMA2 = SMA(self.dataSet2.get("askclose"), period = self.SMA2_period, offset = 0)
            
            # If we have no active position 
            if self.activePosition is None : 
                
                # Long order 
                if self.SMA1.value[-1] > self.SMA2.value[-1] :
                    
                    if self.showInfo : print ("Placing long order")
                    
                    # We place an order. 
                    # Note, the placeOrder function returns a list 
                    # of three elements. If the order have been 
                    # correctly placed, the output is : [Executed order, lmt SL, lmt TG], 
                    # else, the output is : [False, False, False] 
                    orderList = client.placeOrder(self.symbolName,
                                                  action     = "long", 
                                                  orderType  = "MKT", 
                                                  volume     = 0.1, 
                                                  stoploss   = 0., 
                                                  takeprofit = 100.)
                    
                    if orderList[0] is not False : 
                        self.placedOrder = orderList[0] 
                        self.activePosition = "long"
                    
                # Short order 
                if self.SMA1.value[-1] < self.SMA2.value[-1] : 
                    
                    if self.showInfo : print ("Placing short order")
                    
                    orderList = client.placeOrder(self.symbolName,
                                                     action     = "short", 
                                                     orderType  = "MKT", 
                                                     volume     = 0.1, 
                                                     stoploss   = 100., 
                                                     takeprofit = 0.)
                    if orderList[0] is not False : 
                        self.placedOrder = orderList[0] 
                        self.activePosition = "short"
                    
            elif ((self.SMA1.value[-1] > self.SMA2.value[-1] and self.activePosition == "short") or 
                  (self.SMA1.value[-1] < self.SMA2.value[-1] and self.activePosition == "long")): 
                
                # To close a position, one has to provide the order that has been 
                # executed to open the position. 
                client.closePosition(self.symbolName, self.placedOrder)
                self.placedOrder    = None 
                self.activePosition = None
        
        return 
```

Exactly as if we were trading for real. And we can also show some information in the show function : 

```python 
    def show(self, client) : 
        
        if self.showInfo : 
            
            print ("===================================")
            print ("EXAMPLE TRADING STRATEGY LOG OUTPUT")
            print ("===================================")
            print ("Actual date:   ",str(self.lastPrice.get("date"))) 
            print ("Market status: ",self.lastPrice.get("market state"))
            print ("Actual price: ")
            print ("- Ask Open:  ",self.lastPrice.get("askopen"))
            print ("- Ask High:  ",self.lastPrice.get("askhigh"))
            print ("- Ask Low:   ",self.lastPrice.get("asklow"))
            print ("- Ask Close: ",self.lastPrice.get("askclose"))
            print ("- Ask Price: ",self.lastPrice.get("askprice"))
            print ("-----------------------------------")
            if self.dataSet1 is not None:
                print ("The last requested data :")
                print (pd.DataFrame(self.dataSet1))
                print ("The last known moving average values : ")
                print ("SMA(10): ",self.SMA1.value[-1])
                print ("SMA(30): ",self.SMA2.value[-1])
            
            
            try : 
                fig = plt.figure() 
                plt.plot(self.dataSet1.get("date")[-10:], self.SMA1.value[-10:], c = "blue", lw = 2, label = "SMA "+str(self.SMA1_period)) 
                plt.plot(self.dataSet2.get("date")[-10:], self.SMA2.value[-10:], c = "red", lw = 2, label = "SMA "+str(self.SMA2_period))
                plt.legend()
                plt.xticks(rotation=45)
                plt.ylabel("Price")
                plt.xlabel("Date")
                plt.show()
                plt.close(fig = fig)
            except : 
                pass 
        
        return 
```

Simple as that ! 


## [Logging system configuration](#logging-system-configuration)

### In the simulation header file

In order to follow as much accurately as possible your backtest events, the Alpha-Trading Backtest system integrates a logging system which 
is fully customizable and generate databases you can explore with your prefered tool. In the simulation header file, the logging system is 
managed by the method [set_database](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1simulation_1_1SIMULATION.html#a8a6b853ff13caa548899e1d6a6df2f8a) as following: 

```python
sim.set_database(client_id=0, 
                 name="sma_crossover_strategy_logfile", 
                 path="./", 
                 model="sqlite3", 
                 log = True, 
                 tables = tables)
```

The ```client_id``` parameter refers to the index position of the portfolio on which you want to set a database. Here we have only one portfolio so the default 
id is 0. The ```names``` and ```path``` parameters refers to the database name and its location in the user's system. The parameter ```model``` corresponds to 
the database language used. For instance sqlite3 is the only choice. The ```log``` parameter is a boolean refering to which or not the user want to print out a 
*default* log table in the database which contains informations about the [trading_log_actions](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1portfolio_1_1PORTFOLIO.html#a3ee65ef078f407d29d5145bb1479a54f) in the client instance. This means that the system will save 
any simulation information in the database. Finally the most important parameter is the parameter ```tables```. 

Here, ```tables``` refers to a list of custom tables you want to create in the database. By default this list is empty. If you want to add a custom table in your 
database, one needs to follow the structure given by the example here below: 

```python 
tables = [
    {"name"     : "simulation_variables", 
     "structure": {
         "date"                                       : "str", 
         "available_margin"                           : "float",
         "balance"                                    : "float", 
         "number_closed_positions"                    : "int",
         "number_currently_open_positions"            : "int",
         "number_pending_orders"                      : "int",
         "current_drawdown"                           : "float", 
         "current_maximum_number_of_consecutive_gains": "int", 
         "current_value_gain_serie"                   : "float", 
         "current_value_loss_serie"                   : "float", 
         "equity_balance"                             : "float", 
         "number_executed_orders"                     : "int", 
         "margin_level"                               : "float", 
         "used_margin"                                : "float"
     }}
]
```

Here the ```tables``` object contains only one custom table. Each table is represented by a python dictionary with two main keys: 
- The ```name``` of the table. It corresponds to the name of the table as it will be attributed in the database. 
- The ```structure``` of the table. The associated value of this parameter is also a dictionary where the keys refer to the column names 
and the associated value refers to the data type (in python syntax and string formatted). 

At the begining of the simulation, the tool will create the database and the default ```log``` table if you set the parameter 

```python
log = True
```

and each custom table you prepared. Now it's time to fill the custom tables in the strategy file!  

### In the strategy file 

Lets continue with our ```simulation_variables``` table example. During the simulation, elements can be added to custom table with the command 

```python 
simulation_variables = [
    str(self.lastPrice.get("date")),
    client.availableMargin, 
    client.balance, 
    len(client.closedPositions),
    len(client.openPositions), 
    len(client.pendingOrders),
    client.currentDrawDown, 
    client.currentMaximumNumberOfConsecutiveGains, 
    client.currentValueGainSerie, 
    client.currentValueLossSerie, 
    client.equity, 
    len(client.executedOrders), 
    client.marginLevel, 
    client.usedMargin
]

# Write some info into the database 
client.database.insert_element("simulation_variables", simulation_variables)
```

One just needs to prepare a list of the elements specified in the table structure in the header (and in the right order) and pass them to the database. That's it! 

Some important points: 
- Because this process can be extremely usefull to create meta-labeling this function can be written everywhere in your ```STRATEGY``` class. However you have to 
notice that if you do not prepare the exact same database in live trading, this will lead to an error. The client attributes I retrieve here are not necessarly 
available in live trading mode, this can also lead to an error if you keep this as is in live trading mode. Hence, I suggest that you add a variable ```simulation = True``` in your ```STRATEGY.__init__()``` method so that you can both use it in backtest and live mode.  
- Retrieve a lot of informations for future analyses is usefull, but take care to the amount of data you keep. The database can become rapidely very heavy and 
it can considerably slow down the simulation. In particular, for a fast simulation, in the header file I suggest you to set the parameter ```log = False``` or 
at least remove useless elements in the ```PORTFOLIO.trading_log_actions``` list. 

## [Results analysis](#results-analysis)

Once you have done your simulation, you can get the results. The available functions to print results from the simulation are stored in the classes [WRITER](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1writer_1_1WRITER.html) and [ANALYSIS](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1analysis_1_1ANALYSIS.html). You can directly apply them to the SIMULATION object as for example : 

```python 
# We print some basic simulation results 
results = sim.get_basic_results(client_id=0)
# We write the results in a csv file 
sim.writeClosedPositionsFile(index = 0)
# We plot the equity curve 
fig, ax = sim.showEquityCurve(index = [0])
```

The variable ```results``` refers to a basic statistics resume of your backtest: 

```shell
{'Average trade duration': '0:34:22.751505',
 'Cumulated gains': 1199.9999999999982,
 'Cumulated losses': -1357.7000000000276,
 'End date': '2020-01-31 16:58:00',
 'Initial Deposit': 100000,
 'Loosers average loss': -1.9340455840456232,
 'Loosers average loss percentage': -0.0019340455840456234,
 'Loosers ratio': 0.5308284787113476,
 'Max trade duration': '2 days, 7:14:00',
 'Maximum drawdown': 0.003060692238050209,
 'Min trade duration': '0:01:00',
 'Number of loosers': 702,
 'Number of loosers long': 356,
 'Number of loosers short': 346,
 'Number of transactions': 1163,
 'Number of winners': 461,
 'Number of winners long': 228,
 'Number of winners short': 233,
 'Number trades still open': 1,
 'P&L': -157.70000000003012,
 'P&L in percentage': -0.15770000000003012,
 'Profit factor': -0.88384768358251,
 'Start date': '2020-01-03 02:20:00',
 'Symbols': ['EUR.USD'],
 'Winners average gain': 2.6030368763557443,
 'Winners average gain percentage': 0.0026030368763557445,
 'Winners ratio': 0.46917152128865236}
```

But if you want to make a personal data analysis, you can directly get information from the [SIMULATION.portfolio[index]](/_alphatrading/html/classalphatrading_1_1simulation_1_1backtester_1_1portfolio_1_1PORTFOLIO.html) object. Here are stored all the transactions history and the equity curve. 

You can also retrieve data from the database you created. Later, some usefull tools will be implemented for this purpose. 

## Conclusion 

Thank you for following this hands-on. If you found any bug or any mistake, if you want to comment something or suggest an improvement, 
please do not hesitate to create an issue [here](https://github.com/LoannData/Q26_AlphaTrading/issues). 

Retrive the hands-on files here: 

[Hands-On files](https://github.com/LoannData/Q26_AlphaTrading/tree/master/examples/backtester/simple_backtest){: .btn .btn--success .btn--large}