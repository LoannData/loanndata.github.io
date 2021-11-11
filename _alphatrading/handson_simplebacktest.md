---
# permalink: /_alphatrading/handson_simplebacktest
title: "Handson - A simple backtest"
toc: true
toc_label: "Hands-on"
toc_sticky: true
---

## [Introduction](#introduction)

The [Q26 Backtest System](https://q26.io/?page_id=159) is dedicated to algorithmic traders developers who only want to dedicate them to build their trading strategies in a Python 3 environment. In this tutorial, I will show you how to prepare your simulation, strategies and data files in order to get a smart framework in which you can completely focus on your strategy development. 

The Q26 Backtest System needs at least 3 files to run : 
* The historical data file(s) 
* The trading strategy file(s) 
* A simulation header file 

Both, the historical data file(s), the trading strategy file(s) and the Q26 Backtest System are imported in the simulation header file. Thanks to this smart structure, it is possible to add a lot of automating in your backtesting experience to perform a lot more complex simulations such as strategy parameters optimization, backtest a large number of assets/strategies ... so a potential huge gain of time. The python environment being very famous in data sciences, you can have access to a lot modules to analyze your simulation results and improve your backtest experience. 

1. [Introduction](#introduction)

2. [Simulation file](#simulation-file)

    2.1 [Importations & paths](#importations--path)

    2.2 [Data preparation](#data-preparation)

    2.3 [Symbols](#symbols)

    2.4 [Portfolios](#portfolios)

    2.5 [Simulation](#simulation)

3. [Strategy file](#strategy-file)

    3.1 [Global structure](#global-structure)

    3.2 [Trading functions](#trading-functions)

    3.3 [Example : SMA crossover strategy](#example)

4. [Results analysis](#results-analysis)

## [Package installation](#package-installation) 

Let first install the Q26 QuanTester package. Open a terminal and type : 

```shell
pip install q26-quanTester
```

## [Simulation file](#simulation-file)

Let's by start building a simple simulation file header. At first you have to create a python file with the name of your choice (I will call it simulation.py to make it simple) and put it anywhere on your system. 

### [Importations & paths](#importations--path)

I suggest first to import some useful modules in order to manage any objects the Q26 Backtest System manipulates. 

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
# Q26 BacktestSystem class importations 
from quanTest.symbol     import SYMBOL
from quanTest.portfolio  import PORTFOLIO 
from quanTest.data       import PRICE 
from quanTest.data       import PRICE_TABLE
from quanTest.simulation import SIMULATION
```
Let's describe a bit the role of each imported class : 
* The SYMBOL class allows to provide a symbol identity to the simulator. For example, if you want to backtest your trading strategy on XAU.USD symbol, you will need to provide some information such as the contract size, the minimum margin percentage, the margin request method (~ type of product) ... 
* The PORTFOLIO class simulates your trading portfolio but also the trading rules. This means you can define your initial balance but also a leverage value and a bunch of others parameters which can play an important role in the simulation. Moreover, PORTFOLIO member functions simulate with precision your interactions with the market and with the broker. 
* The PRICE and PRICE_TABLE classes are used to store and prepare the historical data. They propose a lot of functionalities and allow to backtest number of assets simultaneously. 
* The SIMULATION class contains all the important parameters and functions to perform a smart simulation.   

To get more information about the attributes and the member functions of these classes, I suggest you to check the detailed code documentation [here](https://q26.io/data/q26_backtest_system/doc/html/annotated.html). 


### [Data preparation](#data-preparation)

Let's start with the data preparation. At first, we have to specify the relative or absolute path to our datafile. Note that for instance the Q26 Backtest System can only read .csv datafiles. So if needed, you can reformat your datafile before integrating it inside the system. But don't worry, the majority of the data providers allow to easily get .csv datafiles. 

```python 
# We define the path to the dataset
path  = "./"
path += "exampleDataFile.csv"
```

Then, let's create a [PRICE](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1data_1_1PRICE.html) object.  

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

To explain to the system how to read it, let's use the command : setColumnsTitle.

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
* Our historical data don't provide bid/ask information. But the Q26 Backtest System requires it. As made in the code, you can read ohlc columns two times, one for the bid and one for the ask, this will just lead to a 0 spread. 
* In some datasets, the time is presented in a unique column while in some others (like in this dataset), the time is splitted between days and hours. The setColumnsTitle has a parameter for that : splitDaysHours. If False, you have to provide the name of the time column in the parameter date. If True, you can let date to None and you have to provide the name of the columns corresponding to days and hours. 

Then, we can read the datafile. 

```python 
# We read the data
price.read(path)
```

When this has been done, we have to specify the timeframe of the data we are going to simulate. 

```python 
# We define the timeframe associated to the loaded data 
price.setBaseTimeframe(timeframe = dt.timedelta(minutes = 1))
```

And we have to prevent the missing data in our datafile by using the following function. 

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

And finally, according to the time details we provided, the algorithm will generate for every piece of time data an information about if the market is open of not. 

```python 
# Function that defines if the market is open/close 
price.setMarketState() 
```

Now we have prepared the data dedicated to the simulation. 

However, in our trading strategy, we are not especially going to use such a low timeframe data, we may for example want to use H1 data. One way to get this new data is to re-sample it directly from the trading strategy. But it is numerically very costly and it destroys the algorithm trading environment experience of the user. That's why we propose a smarter solution. 

Before running any backtest, you prepare all the re-sampled data you will need in your strategies. For example in this tutorial we will need H1 data. To get this data, we create a copy of our initial EUR.USD data and we re-sample it to the H1 timeframe. 

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

We can finally agreggate our data in a [PRICE_TABLE](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1data_1_1PRICE__TABLE.html) object and synchronize them. 

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

Let's implement our [SYMBOL](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1symbol_1_1SYMBOL.html) object. It is simple as that : 

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

We can then initialize our [PORTFOLIO](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1portfolio_1_1PORTFOLIO.html) object with all the usefull parameters. See the doc for more details. 

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
And we add the symbol we prepared inside the portfolio via the function addSymbol(). 

### [Simulation](#simulation)

We can finally start to initialize our [SIMULATION](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1simulation_1_1SIMULATION.html) object by using the following instruction : 

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
* The [sub-loop](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1simulation_1_1SIMULATION.html#aacf7dc6d4d6b01e42bbd2851baf9c961) model represent the way the algorithm will present the sub-candle scale data to the trading strategy file(s). For example here we selected the "close only" sub-loop models meaning that only close price of candles will be presented to the strategies. Another model is "ohlc standard". This means that for every base candle, the algorithm will sequentially present open, high, low and finally close price to the strategy file(s). 
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

We can finally run the simulation according our prefered [running mode](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1simulation_1_1SIMULATION.html#a4c9217c6c17c58f0e44453341d994905) 

```python 
# Run of the simulation 
sim.run(mode = "linear")
```

This is done for the simulation header file. As you can see, this file has a large potential of personalization and optimization. 

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

Inside the run(client) function, you have access to a bunch of client.functions() accessible [here](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1portfolio_1_1PORTFOLIO.html). 

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


## [Results analysis](#results-analysis)

Once you have done your simulation, you can get the results. The available functions to print results from the simulation are stored in the classes [WRITER](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1writer_1_1WRITER.html) and [ANALYSIS](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1analysis_1_1ANALYSIS.html). You can directly apply them to the SIMULATION object as for example : 

```python 
# We write the results in a csv file 
sim.writeClosedPositionsFile(index = 0)
# We plot the equity curve 
fig, ax = sim.showEquityCurve(index = [0])
```

But if you want to make a personal data analysis, you can directly get information from the [SIMULATION.portfolio[index]](https://q26.io/data/q26_backtest_system/doc/html/classquanTest_1_1portfolio_1_1PORTFOLIO.html) object. Here are stored all the transactions history and the equity curve. 
