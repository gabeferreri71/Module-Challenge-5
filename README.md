# Module-Challenge-5: Financial Planning with APIs and Simulations

## Part I: Create a Financial Planner for Emergencies
### Evaluate the Cryptocurrency Wallet by Using the Requests Library

1. Here, we are given the BTC and ETH coin amounts, create a monthly income variable set to 12000, and are given the BTC and ETH urls. 

2. We will do the following for both BTC and ETH. First, I created btc_call and assigned it to btc_url (not necessary but simplifies the code). Using the .json and .get functions, we then create btc_response set to requests.get(btc_call).json() to get a response object, then make the object readable using json.dumps with indent and sort_keys parameters with print(json.dumps(btc_response, indent=2, sort_keys=True)).

For ETH, it's the same process, but using eth_call, eth_url, and eth_response.

3. We now navigate the json objects for price. In the case of BTC, we make a btc_price variable set to ["data"]["1"]["quotes"]["USD"]["price"] in order to navigate the object. Note, for ETH, we use eth_price and the ID is "1027," which would replace the "1" as shown above.

4. To determine the value of the crypto in the wallet, we create btc_value and eth_value variables, assigning each =btc_coins * btc_price and =eth_coins * eth_price. I also rounded both results to two decimal places (value in USD format $0.00). We then add the resulting btc_value and eth_value to get the total_crypto_wallet, which is printed with using the round function to  get USD price format. 

### Evaluate the Stock and Bond Holdings Using Alpaca SDK

1-2. First, we are given spy_shares and agg_shares for later. We now need to request api information for the alpaca main and secret keys. To do this we use:

alpaca_api_key = os.getenv("alpaca_api") 
alpaca_secret_key = os.getenv("alpaca_secret")

Which calls the keys from the .env file we are required to make for this module. To make sure the keys are loaded but not seen, we use display(type()), where the resulting keys should result in "str" is done correctly. 

Next, we create an alpaca rest object using: 

alpaca = tradeapi.REST(
    alpaca_api_key,
    alpaca_secret_key,
    api_version="v2") 

Where we use the main and secret keys along with the version "v2." Note that tradeapi is from our imports and .REST creates the object with the mentioned parameters.

3. Next, we want to make a dataframe of the shares ticker and shares number. First, we created a list named shares_data for the spy_shares and agg_shares. We create a tickers variable for both SPY and AGG next. We now create a small dataframe, portfolio_df, assigned to pd.DataFrame(shares_data, index = tickers) followed by print(portfolio_df) to check the output. We next set our timeframe to "1D," and created our start_date and end_date variables. We are using the same day in this case (I used 01-27-22 from when I worked on the module) and utilize the Timestamp function from Pandas with the parameters of the date and the timezone "tz" as shown:

start_date = pd.Timestamp("2022-01-27", tz = "America/New_York").isoformat() 
end_date = pd.Timestamp("2022-01-27", tz = "America/New_York").isoformat() 

4. To get current closing prices, we use the get_barset() function with .df property with tickers, dataframe, start = start_date and end = end_date, assigned to a variable called portfolio_prices_df and reviewed using the .head function.

5. For agg and spy close prices, we create close_prices variables to each, assigned to float(portfolio_prices_df["AGG"]["close"]) where for SPY we would have "SPY" in this case. This close value needs to be a float for each, hence the use of float(). Both agg_close_price and spy_close_price are printed to see the result.

6. To calculate in USD the shares value for each, we will multiply the close price by the specific share number. Using the AGG example, this is done with:

agg_value = agg_close_price * portfolio_df.loc["AGG"]["shares"]  
round(agg_value, 2)

Where we create agg_value, use agg_close_price from before, and multiply that by the .loc function used on the portfolio_df data frame to find the number of shares. For USD format, the round function was used for two decimals in the output. The process is used for the soy_value. 

For total stocks and bonds, the resulting agg_value and spy_value are added together and assigned to total_stocks_bonds.

For the total portfolio, total_portfolio variable is created and is the sum of total_stocks_bonds and total_crypto_wallet. 

### Evaluate the Emergency Fund

1. First, we want to make a list of the total crypto wallet and the total stocks/bonds. To do this, we used:

savings_data = {
    "Portfolio Wallets": [total_crypto_wallet, total_stocks_bonds]
} 

savings_data

2. We now want to take the list info and make it into a dataframe with wallet ticker labels in $USD format:

wallet_tickers = ["Crypto Value ($USD)", "Stock/Bonds Value ($USD)"]
savings_df = pd.DataFrame(savings_data, index = wallet_tickers) 

round(savings_df,2) 

3. With the savings_df dataframe created, we now want to make a pie chart with parameters 'y = 'Portfolio Wallets'' and 'title = 'Portfolio Composition'' using plot.pie(), resulting in the accompanying pie chart.

4. Lastly, we create an emergency_fund_value varaible equal to monthly_income * 3. We now create three if statements (combos of if, elif, and else) to determine if the total_portfolio is >, == (verify), or < the emergency_fund_value as shown below:

if total_portfolio > emergency_fund_value:
    print("Congrats! You have enough money in this fund.")
elif total_portfolio == emergency_fund_value:
    print("Congrats on reaching this financial goal! You're portfolio matches your emergency fund!")
else: 
    print(f"Your emergency fund goal can be reached with ${(emergency_fund_value - total_portfolio):0.2f} more dollars")

## Part II: Create a Financial Planner For Retirement 
### Create the Monte Carlo Simulation 

1. We're analyzing a 3-year period in this part. Similar to before, we will use the pd.Timestamp() for start and end dates, with varibles being start_date_years and end_date_years. unlike before in start_date_years, the date will be set to 2019-01-27 being three years from the date we're using. We also create a limit_rows variable set to the max of 1000. 

Once the pd.Timestamp() functions are run, we create a dataframe prices_years_df using the .get_barset() function with alpaca. Similar to the single day, we have interior parameters of tickers, timeframe, start = start_date_years, end = end_date_years, and limit = limit_rows. .df is added outside the function. We then use the .head() and .tail() to see the resulting dataframe. Note the dataframe output order, because it will be important in the next step (the output has SPY on the right and AGG on the left)

2. We now will create variable MC_years for the simulation. Using MCSimulation we get:

MC_years = MCSimulation(
  portfolio_data = prices_years_df,
  weights = [.40,.60],
  num_simulation = 500,
  num_trading_days = 252*30
)

Where portfolio_data is assigned to prices_years_df, we weight 0.4 for AGG and 0.6 for SPY (note order from before), we run 500 simulations as given, and for 30 years, num_trading_days is equal to 252 * 30 for each day.

We then review the simulation input data by using MC_years.portfolio_data.head().

For cumulative returns over 30 years, we will input MC_years.calc_cumulative_return() where you'll get returned "Running Monte Carlo simulation number " for 500 samples. A table with rows = num_trading_days and 500 columns will be produced. 

We now want to plot this simulation assigned in a variable simulation_plot. This can be done using MC_years.plot_simulation(). 

3. For probability distribution, we will create distribution_plot and assign it to MC_years.plot_distribution(), and get a resulting bar graph of results.

4. To generate the summary statistics of the simulation, we create variable weight_table and assign it to MC_years.summarize_cumulative_return(), followed by a print statement of weight_table


