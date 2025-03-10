---
title: Strategy Functions Reference
description: 'Develop, backtest, paper trade, and run with the same code.'
position: 13
version: 1.1
category: Blankly Strategy
---

As mentioned before. the `Blankly.Strategy` is an extremely versatile way to interact with the blankly package and subsequently exchanges. The Strategy has easy ways to add various types of events that subscribe to various streams, has backtests built in, and can easily be deployed to our platform via [Blankly Slate](https://blankly.finance). Here's a function reference for those that want to dive even deeper.

## Functions

### `add_price_event(callback: typing.Callable, symbol: str, resolution: typing.Union[str, float], init: typing.Callable = None, teardown: typing.Callable = None, synced: bool = False, variables: dict = None)`

Adds a price event to the strategy. This will pass a price as well as a `price_event` function with args `(price, symbol)`. Users can access their strategy information within `StrategyState`

#### Arguments

| Arg        | Description                                                  | Examples                                                     | Type         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| callback   | A callback function to add a price event for                 | `price_event`                                                | Callable     |
| symbol     | Fill this to inform the price_event which price to provide   | `'BTC-USD'` or `'XLM-USD'`                                   | str          |
| resolution | Resolution to send prices to the user function.              | `3600` or `'15s'`                                            | str or float |
| init       | Fill this with a callback function to allow a setup for the state variable. This is run per price event initialization. | Pass a function like `setup` with arguments that are `setup(currency_pair, state)` | Callable     |
| teardown   | A function to run when the strategy is stopped or interrupted. Example usages include liquidating     positions, writing or cleaning up data or anything else useful | `teardown(state)`                                            | Callable     |
| synced     | Whether to start price event in sync with the exchange resolution times (i.e. if it's 15m resolution then run at 12:15, 12:30, 12:45, and 1:00) | True or False                                                | bool         |
| variables  | A dictionary to initialize the variables dictionary in the state | `{'symbol': 'BTC-USD'}`                                      | dict         |

### Example Use Case

<alert>

Remember to set up your `settings.json` and `keys.json` in order for all your code to fully function properly. Check out our [getting started docs](/getting-started/installation)

</alert>

```python
from blankly import Strategy, StrategyState, Alpaca
import blankly


def init(symbol: str, state: StrategyState):
    variables = state.variables  # grab the variables dictionary
    # initialize any variables here
    variables['has_bought'] = True
    # get 50 data points at specific resolution of event
    variables['history'] = state.interface.history(symbol, 50, state.resolution)['close'].to_list()


def price_event(price, symbol, state: StrategyState):
    """ This buys and sells whenever the boolean is true or false """
    interface: blankly.Interface = state.interface
    state.variables['history'].append(price)  # add new price to history
    # buy the symbol using available cash
    if not state.variables['has_bought']:
        interface.market_order(symbol, 'buy', int(interface.cash/price))
        state.variables['has_bought'] = True
    else:
        interface.market_order(symbol, 'sell', int(interface.account[state.base_asset]['available']))
        state.variables['has_bought'] = False


a = Alpaca()
s = Strategy(a)
s.add_price_event(price_event, 'MSFT', resolution='15m', init=init)

s.start()
```

Check out our [RSI](/examples/rsi), [Golden Cross](/examples/golden-cross) examples as well for more references.

### `add_scheduled_event(callback: typing.Callable, resolution: typing.Union[str, float], init: typing.Callable = None, teardown: typing.Callable = None, synced: bool = False, variables: dict = None)`

This creates an event that will call your function at a regular interval with no extra API calls.

#### Arguments

| Arg        | Description                                                  | Examples                                                     | Type         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| callback   | A callback function to add a price event for                 | `price_event`                                                | Callable     |
| resolution | Resolution to send prices to the user function.              | `3600` or `'15s'`                                            | str or float |
| init       | Fill this with a callback function to allow a setup for the state variable. This is run per price event initialization. | Pass a function like `setup` with arguments that are `setup(currency_pair, state)` | Callable     |
| teardown   | A function to run when the strategy is stopped or interrupted. Example usages include liquidating     positions, writing or cleaning up data or anything else useful | `teardown(state)`                                            | Callable     |
| synced     | Whether to start price event in sync with the exchange resolution times (i.e. if it's 15m resolution then run at 12:15, 12:30, 12:45, and 1:00) | True or False                                                | bool         |
| variables  | A dictionary to initialize the variables dictionary in the state | `{'symbol': 'BTC-USD'}`                                      | dict         |

### `add_arbitrage_event(callback: typing.Callable, symbols: list, resolution: typing.Union[str, float], init: typing.Callable = None, teardown: typing.Callable = None, synced: bool = False, variables: dict = None)`

This will gather data from multiple symbols and send the prices to your event. When live this will create a thread pool and query each symbol in a different thread to optimize the speed.

#### Arguments

| Arg        | Description                                                  | Examples                                                     | Type         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| callback   | A callback function to add a price event for                 | `price_event`                                                | Callable     |
| symbols    | A list of symbols to grab the price from                     | `['BTC-USD', 'ETH-USD']`                                     | list         |
| resolution | Resolution to send prices to the user function.              | `3600` or `'15s'`                                            | str or float |
| init       | Fill this with a callback function to allow a setup for the state variable. This is run per price event initialization. | Pass a function like `setup` with arguments that are `setup(currency_pair, state)` | Callable     |
| teardown   | A function to run when the strategy is stopped or interrupted. Example usages include liquidating     positions, writing or cleaning up data or anything else useful | `teardown(state)`                                            | Callable     |
| synced     | Whether to start price event in sync with the exchange resolution times (i.e. if it's 15m resolution then run at 12:15, 12:30, 12:45, and 1:00) | True or False                                                | bool         |
| variables  | A dictionary to initialize the variables dictionary in the state | `{'symbol': 'BTC-USD'}`                                      | dict         |

### `add_orderbook_event(callback: typing.Callable, symbol: str, init: typing.Callable = None, teardown: typing.Callable = None)`

Add a orderbook events to the strategy. This will pass a price as well as a full orderbook with args `(price, symbol)`.

<alert type="warning">

Note: Currently Alpaca does not support level II market order data so the orderbook feed is limited and will not show a full orderbook like Coinbase Pro or Binance.

</alert>

#### Arguments

| Arg        | Description                                                  | Examples                                                     | Type         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| callback   | A callback function to add a price event for                 | `price_event`                                                | Callable     |
| symbol     | Fill this to inform the order book_event which price to provide | `'BTC-USD'` or `'XLM-USD'` or 'MSFT'                         | str          |
| resolution | Resolution to send prices to the user function.              | `3600` or `'15s'`                                            | str or float |
| init       | Fill this with a callback function to allow a setup for the state variable. | Pass a function like `setup` with arguments that are `setup(symbol, state)` | Callable     |
| teardown   | A function to run when the strategy is stopped or interrupted. Example usages include liquidating     positions, writing or cleaning up data or anything else useful | `teardown(state_object)`                                     | Callable     |

### `add_bar_event(callback: typing.Callable, symbol: str, resolution: str or float, init: typing.Callable = None, teardown: typing.Callable = None)`

Adds a bar (OHCLV data) event. This is particularly useful for oscillators and indicators that require OHCLV data continuously. 

<alert>

Bar Events are by definition synced with the exchange so that bucket intervals are aligned with exchange data for OHCLV

</alert>

#### Arguments

| Arg        | Description                                                  | Examples                                                     | Type         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| callback   | A callback function to add a price event for                 | `price_event`                                                | Callable     |
| symbol     | Fill this to inform the order book_event which price to provide | `'BTC-USD'` or `'XLM-USD'` or 'MSFT'                         | str          |
| resolution | Resolution to send prices to the user function.              | `3600` or `'15s'`                                            | str or float |
| init       | Fill this with a callback function to allow a setup for the state variable. | Pass a function like `setup` with arguments that are `setup(symbol, state)` | Callable     |
| teardown   | A function to run when the strategy is stopped or interrupted. Example usages include liquidating     positions, writing or cleaning up data or anything else useful | `teardown(state_object)`                                     | Callable     |

#### Example Use Case

```python
from blankly import Strategy, StrategyState, Interface
from blankly.indicators import aroon_oscillator
import blankly


def init(symbol: str, state: StrategyState):
    variables = state.variables  # grab the variables dictionary
    # initialize any variables here
    variables['has_bought'] = False
    # get 50 data points at specific resolution of event
    variables['history'] = state.interface.history(symbol, 50, state.resolution)


def some_bar_event(bar, symbol, state: StrategyState):
    '''This buys and sells whenever the boolean is true or false'''
    # bar = {'open': 1234.5, 'close': 1200.4, 'high': 1555', 'low': 1000, 'volume': 25000}
    interface: blankly.Interface = state.interface
    variables = state.variables  # grab the variables state
    variables['history'].append(bar)  # add new price to history
    oscillation = aroon_oscillator(variables['close'], variables['high'], variables['low'])
    # ... do something with the oscillation calculation


a = blankly.Alpaca()
s = Strategy(a)
s.add_price_event(some_bar_event, 'MSFT', resolution='15m', init=init)
```

### `time() -> float` 

Gets the current time depending on if the strategy is in a backtest mode or not. 

### Response

| Key  | Description                     | Type |
| ---- | ------------------------------- | ---- |
| time | The epoch time that is returned | int  |

### `backtest(initial_values: dict = None, to: str = None, start_date: str = None, end_date: str = None, save: bool = False, settings_path: str = None, callbacks: typing.Callable = None, **kwargs)`

This allows the user to backtest the strategy on a given time interval and with various starting values. We have made our backtesting as customizable as possible and also have built-in all [`blankly.metrics`](/metrics/metrics). Users can also create their own custom metrics as well. 

#### Arguments

| Arg            | Description                                                  | Examples                                                     | Type                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------- |
| initial_values | Optional dictionary of initial value sizes                   | `{ 'BTC': 3, 'USD': 5650}`                                   | dict                  |
| to             | Optionally declare an amount of time before now to backtest  | `'5y'` or `'10h'`                                            | str                   |
| start_date     | Optionally override argument "to" by specifying a start date | `'03/06/2018'`                                               | str                   |
| end_date       | Optionally end the backtest at a date                        | `'03/06/2018'`                                               | str                   |
| save           | Optionally save the price data references to the data required for the backtest as well as     overriden settings | `'True'` or `'False'`                                        | bool                  |
| settings_path  | Optional path to the backtest.json file.                     | `'./backtest.json'`                                          | str                   |
| callbacks      | Additional metrics that take in one parameter `backtest_data`. They then return their result back into the backtesting framework for a `BacktestResult` object. More info on the passed argument can be found below. | `custom_metric(dataframes): pass`                            | list[typing.Callable] |
| **kwargs       | Use these `**kwargs` to set any of the backtesting `settings` defined in [`backtest.json`](/usage/backtest.json). | `strategy.backtest(use_price='open')`. You can also specify `save=True` to write these directly to `backtest.json` for global reuse. | kwarg                 |



#### Custom Metrics Function

When implementing custom metrics, we pass a dictionary of values to you:

##### Example

```json
{
  'history':
            BTC   EUR  USD          time   Account Value (USD)
	0    0.000000   0  100.0  1.591391e+09           100.000000
	1    0.010332   0    0.0  1.591391e+09            99.919840
	..        ...  ..    ...           ...                  ...
	417  0.003566   0    0.0  1.627333e+09           132.932823
	418  0.003566   0    0.0  1.627420e+09           140.798602
	[419 rows x 5 columns], 

	'resampled_account_value':              
	             time       value
	0    1.591391e+09  100.000000
	1    1.591477e+09   99.919840
	..            ...         ...
	416  1.627333e+09  126.282068
	417  1.627420e+09  132.932823
	[418 rows x 2 columns], 

	'returns':              
	             time     value
	0    1.591391e+09       NaN
	1    1.591477e+09 -0.080160
	..            ...       ...
	416  1.627333e+09  4.036733
	417  1.627420e+09  6.650755
	[418 rows x 2 columns]
}
```

### Properties

| Arg                      | Description                                                  | Examples                                      | Type         |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- | ------------ |
| resampled_account_values | Access to the Strategy Interface                             | `backtest_data['resampled_account_values']`   | pd.Dataframe |
| history                  | The history of account values                                | `backtest_data['history']`                    | pd.Dataframe |
| returns                  | Dataframe of Returns, the value of the returns can be accessed by column `returns['value']` | `returns = backtest_data['returns']['value']` | pd.Dataframe |



Users can use this to define their own custom metrics. F.e., let's say we want to combine the Sharpe and Sortino Ratio:

```python
from blankly import Strategy, StrategyState, Interface, Alpaca
from blankly.metrics import sortino, sharpe


def custom_backtest_metric(backtest_data):
    returns = backtest_data['returns']['value']  # get all the returns
    return sortino(returns) + sharpe(returns)


a = Alpaca()
s = Strategy(a)
s.add_price_event(custom_backtest_metric, 'MSFT', resolution='15m')
s.backtest(initial_values={'USD': 10000}, to='2y', callbacks=[custom_backtest_metric])
```



#### Complete Backtest Example Use Case

Taking our first example, let's say we wanted to make a custom metric that combined the Sortino Ratio with the Sharpe Ratio

```python
from blankly import Strategy, StrategyState, Interface
from blankly.metrics import sortino, sharpe
import blankly


def init(symbol: str, state: StrategyState):
    variables = state.variables  # grab the variables dictionary
    # initialize any variables here
    variables['model_started'] = False
    # get 50 data points at specific resolution of event
    variables['history'] = state.interface.history(symbol, 50, state.resolution)['close'].tolist()


def price_event(price, symbol, state: StrategyState):
    interface: blankly.Interface = state.interface
    variables = state.variables
    variables['history'].append(price)  # add new price to history
    # Sell 1 of whatever the symbol was (in this case MSFT)
    interface.market_order(symbol, 'sell', 1)


def custom_sharpe_sortino(backtest_data):
    returns = backtest_data['returns']['value']  # get all the returns
    return sortino(returns) + sharpe(returns)


a = blankly.Alpaca()
s = Strategy(a)
s.add_price_event(price_event, 'MSFT', resolution='15m', init=init)
result = s.backtest(initial_values={'MSFT': 10000},
                    start_date='2015-09-25',
                    end_date='2016-08-25',
                    callbacks=[custom_sharpe_sortino])

print(result)
```

