<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Backtesting Investment Strategies (US Stocks)</title>
    <style>
        body { font-family: Arial, sans-serif; }
        h1, h2, h3 { color: #333; }
        pre { background-color: #f4f4f4; padding: 10px; border-left: 4px solid #ddd; }
        code { background-color: #f4f4f4; padding: 2px 4px; border-radius: 4px; }
        ul { list-style-type: disc; padding-left: 20px; }
        .note { background-color: #e7f0ff; border-left: 4px solid #007bff; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>Backtesting Investment Strategies (US Stocks)</h1>

<h2>Overview</h2>
    <p>
        This project focuses on backtesting different investment strategies using historical data of the Dow Jones Industrial Average (DJIA). The analysis covers several strategies, including a simple momentum strategy, contrarian strategy, and strategies based on simple moving averages (SMA). The goal is to evaluate the performance of these strategies compared to a basic buy-and-hold approach.
    </p>

<h2>Requirements</h2>
    <p>
        - Python (preferably 3.6 or higher)<br>
        - Required libraries:
        <ul>
            <li><code>pandas</code></li>
            <li><code>numpy</code></li>
            <li><code>matplotlib</code></li>
            <li><code>seaborn</code></li>
        </ul>
        You can install the required packages using pip:
        <pre><code>pip install pandas numpy matplotlib seaborn</code></pre>
    </p>

<h2>Data</h2>
    <p>
        The data used in this analysis is stored in <code>dji.csv</code>, which contains historical closing prices of the DJIA index. The data should include at least the following columns:
        <ul>
            <li><code>Date</code>: The date of the record.</li>
            <li><code>Close</code>: The closing price of the DJIA on that date.</li>
        </ul>
    </p>

<h2>Code Explanation</h2>

<h3>Importing the Data</h3>
    <pre><code>
import pandas as pd

# Load and inspect data
data = pd.read_csv("dji.csv", parse_dates=["Date"], index_col="Date")
df = data.loc["2010-01-01":"2020-03-31", "Close"].to_frame()
    </code></pre>

<h3>Data Visualization & Returns</h3>
    <pre><code>
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns

plt.style.use("seaborn")
df.plot(figsize=(20,12), fontsize=15)
plt.legend(fontsize=20)
plt.show()

df["Return"] = df.pct_change()
df.dropna(inplace=True)
df.plot(figsize=(20,12), secondary_y="Return", mark_right=True, fontsize=15)
plt.show()
    </code></pre>

<h3>Backtesting a Simple Momentum Strategy</h3>
    <pre><code>
df["Position"] = np.sign(df["Return"])
df["Strategy_Ret"] = df["Position"].shift() * df["Return"]
df["Strategy"] = df["Strategy_Ret"].add(1, fill_value=0).cumprod() * df.iloc[0, 0]

df[["Close", "Strategy"]].plot(figsize=(15,10), fontsize=15)
plt.title("Simple Momentum Strategy", fontsize=20)
plt.legend(fontsize=15)
plt.show()
    </code></pre>

<h3>Backtesting a Simple Contrarian Strategy</h3>
    <pre><code>
df["Position"] = -np.sign(df["Return"])
df["Strategy_Ret"] = df["Position"].shift() * df["Return"]
df["Strategy"] = df["Strategy_Ret"].add(1, fill_value=0).cumprod() * df.iloc[0, 0]

df[["Close", "Strategy"]].plot(figsize=(15,10), fontsize=15)
plt.legend(fontsize=15)
plt.title("Simple Contrarian Strategy", fontsize=20)
plt.show()
    </code></pre>

<h3>More Complex Strategies & Backtesting vs. Fitting</h3>
    <pre><code>
df["Position"] = np.where(df["Return"] > 0.01, -1, 1)
df["Strategy_Ret"] = df["Position"].shift() * df["Return"]
df["Strategy"] = df["Strategy_Ret"].add(1, fill_value=0).cumprod() * df.iloc[0, 0]

df[["Close", "Strategy"]].plot(figsize=(15,10), fontsize=15)
plt.legend(fontsize=15)
plt.title("More Complex Strategy", fontsize=20)
plt.show()
    </code></pre>

<h3>Simple Moving Averages (SMA)</h3>
    <pre><code>
df["SMA50"] = df["Close"].rolling(window=50).mean()
df["SMA200"] = df["Close"].rolling(window=200).mean()

df[["Close", "SMA50"]].plot(figsize=(15,10), fontsize=15)
plt.legend(fontsize=15)
plt.show()

df[["SMA50", "SMA200"]].plot(figsize=(15,10), fontsize=15)
plt.legend(fontsize=15)
plt.show()
    </code></pre>

<h3>SMA Crossover Strategy</h3>
    <pre><code>
df["Position"] = np.sign(df["SMA50"] - df["SMA200"])
df["Strategy_Ret"] = df["Position"].shift() * df["Return"]
df["Strategy"] = df["Strategy_Ret"].add(1, fill_value=0).cumprod() * df.iloc[0, 0]

df[["Close", "Strategy"]].plot(figsize=(15,10), fontsize=15)
plt.legend(fontsize=15)
plt.title("SMA Crossover Strategy", fontsize=20)
plt.show()
    </code></pre>

<h3>Backtesting the Perfect Strategy</h3>
    <pre><code>
df["Position"] = np.sign(df["Return"])
df["Strategy_Ret"] = df["Position"] * df["Return"]
df["Strategy"] = df["Strategy_Ret"].add(1, fill_value=0).cumprod() * df.iloc[0, 0]

df[["Close", "Strategy"]].plot(figsize=(15,10), fontsize=15, logy=True)
plt.legend(fontsize=15)
plt.title("The Perfect Strategy", fontsize=20)
plt.show()
    </code></pre>

<h3>Function for Summary Statistics</h3>
    <pre><code>
def summary_ann(returns):
    summary = returns.agg(["mean", "std"]).T
    summary["Return"] = summary["mean"] * 252
    summary["Risk"] = summary["std"] * np.sqrt(252)
    summary.drop(columns=["mean", "std"], inplace=True)
    return summary

summary_ann(df[["Return", "Strategy_Ret"]])
    </code></pre>

<h2>Notes</h2>
    <div class="note">
        <ul>
            <li><strong>Backtesting vs. Fitting:</strong> Strategies should not be overly fitted to historical data. Forward testing is necessary to validate performance.</li>
            <li><strong>Transaction Costs:</strong> Costs associated with changing positions should be included in the analysis.</li>
            <li><strong>Tax Effects:</strong> Be aware of potential tax implications from frequent trading.</li>
        </ul>
    </div>
</body>
</html>
