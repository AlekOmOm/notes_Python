# 15. Project Example: Pandas Candlestick Analysis Dashboard 📈

[<- Back: Project Example: NumPy Image Filter App](./14-ex-Numpy-Streamlit-Image-Filter-App.md) | [Next: (Conclusion or Next Steps) ->]

## Table of Contents

- [Project Concept](#project-concept)
- [Requirements](#requirements)
- [Why this Project Showcases Pandas](#why-this-project-showcases-pandas)
- [Assumptions about the API](#assumptions-about-the-api)
- [Installation](#installation)
- [Project Code](#project-code)
- [Running the App](#running-the-app)
- [Code Explanation](#code-explanation)
- [Possible Extensions](#possible-extensions)

## Project Concept

Create an interactive Streamlit dashboard that fetches 15-minute Bitcoin (BTCUSDT) candlestick data for a user-selected day from a specified API. The application will use Pandas to process this time-series data, calculate key statistics and moving averages, and visualize the price action and indicators using Plotly for interactive charting.

## Requirements

1.  **Framework:** Use Streamlit for the web application interface.
2.  **Core Library:** Use Pandas extensively for data handling, manipulation, time-series analysis, and calculations.
3.  **Data Source:** Fetch data via HTTP GET request from the user-provided candlestick server API endpoint.
4.  **User Input:** Allow users to select a specific date for which to fetch and analyze data. Allow users to specify periods for calculating Simple Moving Averages (SMAs).
5.  **Functionality:**
    *   Fetch OHLCV (Open, High, Low, Close, Volume) data for BTCUSDT for the selected day.
    *   Display basic daily statistics (Day's Open, High, Low, Close, Total Volume, Price Change %).
    *   Calculate and display configurable Simple Moving Averages (e.g., 8-period, 21-period SMAs based on the 15-min candles).
    *   Display an interactive candlestick chart of the 15-minute data for the day.
    *   Overlay the calculated SMAs on the candlestick chart.
6.  **Visualization:** Use Plotly for creating interactive candlestick charts with overlays.
7.  **Dependencies:** Use `requests` library for API calls. NumPy might be used implicitly by Pandas but direct usage should be minimized.
8.  **Keep it Simple:** Focus on demonstrating Pandas' time-series handling, DataFrame manipulation, and rolling calculations. Aim for concise code.

## Why this Project Showcases Pandas

*   **Data Acquisition & Structuring:** Reading data from an external source (API JSON response) directly into a DataFrame.
*   **Data Cleaning:** Handling potential missing values (if any) and ensuring correct data types (numeric for prices/volume, datetime for time).
*   **Time Series Indexing:** Setting and utilizing a `DatetimeIndex` for powerful time-based selection and analysis (fundamental to financial data).
*   **Columnar Operations:** Easily accessing and performing calculations on specific columns (Open, High, Low, Close, Volume).
*   **Rolling Calculations:** Demonstrating the `.rolling().mean()` functionality for calculating SMAs efficiently.
*   **Aggregation & Description:** Calculating daily summary statistics from the 15-minute intervals (e.g., `min()`, `max()`, `sum()`, first/last values).
*   **Data Transformation:** Creating new columns for indicators like SMAs.
*   **Visualization Integration:** Although using Plotly, Pandas prepares the data in the ideal format for charting libraries.

## Assumptions about the API

Since the exact API details are not provided, we will make the following assumptions. **You will need to modify the `API_ENDPOINT` and potentially the data parsing logic in the code to match your specific server.**

1.  **Endpoint Structure:** There's an API endpoint like `http://your-candlestick-server.com/api/candles/BTCUSDT/15m/{YYYY-MM-DD}` where `{YYYY-MM-DD}` is the date string.
2.  **Method:** The request method is GET.
3.  **Response Format:** The API returns a JSON response, likely a list of dictionaries or a dictionary containing a list, where each element represents a 15-minute candle.
4.  **Data Fields:** Each candle object in the JSON response contains fields like:
    *   `timestamp` (Unix timestamp in milliseconds or seconds, or an ISO string)
    *   `open` (string or number)
    *   `high` (string or number)
    *   `low` (string or number)
    *   `close` (string or number)
    *   `volume` (string or number)

## Installation

You'll need Streamlit, Pandas, Requests, and Plotly.

```bash
pip install streamlit pandas requests plotly
```

## Project Code

Save the following code as a Python file (e.g., `btc_analyzer_app.py`).

```python
import streamlit as st
import pandas as pd
import requests
import plotly.graph_objects as go
from datetime import datetime, date, timedelta

# --- Configuration & API ---

# !!! IMPORTANT: UPDATE THIS URL TO YOUR ACTUAL API ENDPOINT !!!
# Example assumes date is appended like YYYY-MM-DD
API_ENDPOINT_TEMPLATE = "http://127.0.0.1:8000/api/candles/BTCUSDT/15m/{date_str}"
# API_ENDPOINT_TEMPLATE = "http://your-candlestick-server.com/api/candles/BTCUSDT/15m/{date_str}" # Example


# --- Data Fetching and Processing Function ---

@st.cache_data(ttl=900) # Cache data for 15 minutes (900 seconds)
def fetch_data(selected_date):
    """Fetches and processes 15m BTCUSDT candle data for the selected date."""
    date_str = selected_date.strftime('%Y-%m-%d')
    api_url = API_ENDPOINT_TEMPLATE.format(date_str=date_str)
    try:
        response = requests.get(api_url, timeout=10) # Added timeout
        response.raise_for_status() # Raise an exception for bad status codes (4xx or 5xx)
        data = response.json()

        if not data:
            st.warning(f"No data returned from API for {date_str}. Is the server running?")
            return None

        # --- Convert to Pandas DataFrame ---
        # This part depends HEAVILY on your API's JSON structure. Adjust accordingly.
        # Assumption: JSON is a list of candle dictionaries.
        df = pd.DataFrame(data)

        # --- Data Cleaning and Type Conversion (Crucial with Pandas) ---
        # Rename columns if necessary (e.g., if API uses 't', 'o', 'h', 'l', 'c', 'v')
        # df.rename(columns={'t': 'timestamp', 'o': 'open', ...}, inplace=True)

        # 1. Convert Timestamp
        # Try converting from milliseconds, then seconds, then ISO format
        try:
            # Assume timestamp is in milliseconds
            df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
        except (ValueError, TypeError):
             try:
                 # Assume timestamp is in seconds
                 df['timestamp'] = pd.to_datetime(df['timestamp'], unit='s')
             except (ValueError, TypeError):
                  # Assume timestamp is ISO format string
                 df['timestamp'] = pd.to_datetime(df['timestamp'])

        # 2. Convert OHLCV columns to numeric, coercing errors
        ohlcv_cols = ['open', 'high', 'low', 'close', 'volume']
        for col in ohlcv_cols:
            df[col] = pd.to_numeric(df[col], errors='coerce')

        # 3. Drop rows with NaN values that might result from coercion
        df.dropna(subset=ohlcv_cols, inplace=True)

        # 4. Set Timestamp as Index (Essential for time-series operations)
        df.set_index('timestamp', inplace=True)

        # Ensure data is sorted by time
        df.sort_index(inplace=True)

        # Filter strictly for the selected day (handles potential time zone issues better)
        start_dt = pd.Timestamp(selected_date).tz_localize(df.index.tz) # Match timezone if exists
        end_dt = start_dt + pd.Timedelta(days=1)
        df = df[(df.index >= start_dt) & (df.index < end_dt)]

        if df.empty:
             st.warning(f"Dataframe is empty after processing for {date_str}. Check API response and processing logic.")
             return None

        return df

    except requests.exceptions.RequestException as e:
        st.error(f"API Request Failed: {e}")
        return None
    except (KeyError, ValueError, TypeError) as e:
        st.error(f"Data Processing Error: {e}. Check API response format and DataFrame processing steps.")
        st.json(data) # Show the raw data that caused the error
        return None
    except Exception as e:
        st.error(f"An unexpected error occurred: {e}")
        return None

def calculate_sma(df, period):
    """Calculates the Simple Moving Average for the 'close' price."""
    if 'close' not in df.columns:
        return None
    return df['close'].rolling(window=period).mean()

def create_candlestick_chart(df, sma_short_data=None, sma_long_data=None, sma_short_period=None, sma_long_period=None):
    """Creates an interactive Plotly candlestick chart."""
    fig = go.Figure()

    # Candlestick Trace
    fig.add_trace(go.Candlestick(x=df.index,
                                 open=df['open'],
                                 high=df['high'],
                                 low=df['low'],
                                 close=df['close'],
                                 name='Candles'))

    # Short SMA Trace
    if sma_short_data is not None and sma_short_period:
        fig.add_trace(go.Scatter(x=df.index, y=sma_short_data,
                                 mode='lines', name=f'SMA {sma_short_period}',
                                 line=dict(color='orange', width=1)))

    # Long SMA Trace
    if sma_long_data is not None and sma_long_period:
        fig.add_trace(go.Scatter(x=df.index, y=sma_long_data,
                                 mode='lines', name=f'SMA {sma_long_period}',
                                 line=dict(color='purple', width=1)))

    fig.update_layout(
        title=f"BTC/USDT 15-Minute Candlesticks ({df.index.date[0].strftime('%Y-%m-%d')})",
        xaxis_title="Time",
        yaxis_title="Price (USDT)",
        xaxis_rangeslider_visible=False, # Hide the range slider
        height=600, # Adjust height
        legend_title_text='Legend'
    )
    return fig

# --- Streamlit App Layout ---

st.set_page_config(layout="wide")
st.title("📈 Pandas Candlestick Analyzer")
st.write("Analyze 15-minute BTC/USDT data fetched from your API.")

# Sidebar for Inputs
st.sidebar.header("Inputs")
# Date selection - default to yesterday
yesterday = date.today() - timedelta(days=1)
selected_date = st.sidebar.date_input("Select Date", yesterday)

# SMA Period Inputs
st.sidebar.subheader("Moving Averages")
sma_short_period = st.sidebar.number_input("Short SMA Period", min_value=1, max_value=50, value=8, step=1)
sma_long_period = st.sidebar.number_input("Long SMA Period", min_value=2, max_value=200, value=21, step=1)

# --- Main Area ---

# Fetch data when date changes
df_data = fetch_data(selected_date)

if df_data is not None and not df_data.empty:
    st.header(f"Analysis for {selected_date.strftime('%Y-%m-%d')}")

    # --- Calculate Daily Statistics using Pandas ---
    day_open = df_data['open'].iloc[0]
    day_high = df_data['high'].max()
    day_low = df_data['low'].min()
    day_close = df_data['close'].iloc[-1]
    day_volume = df_data['volume'].sum()
    price_change = ((day_close - day_open) / day_open) * 100 if day_open != 0 else 0

    # Display Stats in Columns
    col1, col2, col3, col4, col5 = st.columns(5)
    col1.metric("Day Open", f"${day_open:,.2f}")
    col2.metric("Day High", f"${day_high:,.2f}")
    col3.metric("Day Low", f"${day_low:,.2f}")
    col4.metric("Day Close", f"${day_close:,.2f}", f"{price_change:.2f}%")
    col5.metric("Total Volume", f"{day_volume:,.0f}")

    # --- Calculate SMAs ---
    sma_short = calculate_sma(df_data, sma_short_period)
    sma_long = calculate_sma(df_data, sma_long_period)

    # --- Create and Display Chart ---
    st.subheader("Candlestick Chart with Moving Averages")
    fig = create_candlestick_chart(df_data, sma_short, sma_long, sma_short_period, sma_long_period)
    st.plotly_chart(fig, use_container_width=True)

    # --- Display Raw Data (Optional) ---
    if st.checkbox("Show Raw Data"):
        st.subheader("Raw 15-Minute Data")
        st.dataframe(df_data)

elif df_data is None:
    # Error messages are handled within fetch_data
    st.info("Enter API details and select a date to fetch data.")
else: # df_data is empty after processing
    st.warning(f"No valid data could be processed for {selected_date.strftime('%Y-%m-%d')}.")

```

## Running the App

1.  **Modify the Code:** **Crucially, update the `API_ENDPOINT_TEMPLATE` variable** near the top of the script with the correct URL for your candlestick server. You might also need to adjust the DataFrame creation and cleaning steps inside `fetch_data` based on your API's exact JSON response structure and field names.
2.  Save the code as `btc_analyzer_app.py`.
3.  Open your terminal or command prompt.
4.  Navigate to the directory where you saved the file.
5.  Ensure your candlestick server API is running and accessible.
6.  Run the command:

    ```bash
    streamlit run btc_analyzer_app.py
    ```

7.  Streamlit will open the application in your web browser. Select a date and view the analysis.

## Code Explanation

1.  **Configuration:** Sets the `API_ENDPOINT_TEMPLATE`. **This MUST be changed.**
2.  **`fetch_data(selected_date)` Function:**
    *   Uses `@st.cache_data` to avoid re-fetching data from the API unnecessarily if the date hasn't changed (cache expires after 15 minutes).
    *   Formats the date and constructs the specific API URL.
    *   Uses `requests.get()` to fetch data. Includes basic error handling (`raise_for_status()`, `try...except`).
    *   **Pandas DataFrame Creation:** `pd.DataFrame(data)` converts the JSON list into a DataFrame. **(This is highly dependent on API output)**.
    *   **Pandas Data Cleaning:** Converts `timestamp` to datetime objects (`pd.to_datetime`) and OHLCV columns to numeric (`pd.to_numeric`, `errors='coerce'`). Handles potential NaNs (`dropna`).
    *   **Pandas Time Series Index:** Sets the converted timestamp column as the DataFrame's index (`df.set_index()`). This is vital for time-series operations.
    *   Sorts the index (`sort_index`) and filters data strictly for the selected day.
    *   Returns the processed DataFrame or `None` if errors occur.
3.  **`calculate_sma(df, period)` Function:** Takes a DataFrame and period, uses Pandas' `.rolling(window=...).mean()` on the 'close' column to calculate the SMA.
4.  **`create_candlestick_chart(...)` Function:** Uses Plotly (`graph_objects`) to create the chart. Adds a `go.Candlestick` trace for OHLC and `go.Scatter` traces for the SMAs passed to it. Configures layout basics.
5.  **Streamlit UI:**
    *   Standard setup (`st.title`, sidebar widgets).
    *   `st.date_input` provides a calendar for date selection.
    *   `st.number_input` allows users to change SMA periods.
    *   The main part of the script calls `fetch_data`.
    *   If data is valid:
        *   **Pandas Aggregations:** Calculates daily stats by accessing the first/last elements (`.iloc`, `.iloc[-1]`) and using `.max()`, `.min()`, `.sum()` on the relevant DataFrame columns.
        *   Displays statistics using `st.metric`.
        *   Calls `calculate_sma` for both periods.
        *   Calls `create_candlestick_chart` to generate the plot.
        *   Displays the interactive Plotly chart using `st.plotly_chart`.
        *   Optionally displays the raw DataFrame using `st.dataframe` controlled by `st.checkbox`.
    *   Displays appropriate messages if data fetching fails or processing results in an empty DataFrame.

## Possible Extensions

*   **More Indicators:** Add other technical indicators using Pandas (e.g., Exponential Moving Average - EMA using `.ewm().mean()`, Bollinger Bands, RSI).
*   **Volume Visualization:** Add a separate subplot below the candlestick chart to show the volume using a bar chart (`go.Bar`).
*   **Date Range Selection:** Allow users to select a date range instead of just a single day.
*   **Resampling:** Add options to resample the 15-minute data to other timeframes (e.g., 1-hour, 4-hour, daily) using Pandas' `df.resample().agg()` and display charts for those timeframes.
*   **Error Handling:** Implement more detailed error messages based on specific API status codes or data parsing issues.
*   **Comparison:** Allow selecting two different dates and display their charts or stats side-by-side.
*   **Data Persistence:** Save fetched data locally (e.g., CSV, Parquet) to build a local cache and reduce API calls.

---

[<- Back: Project Example: NumPy Image Filter App](./14-ex-Numpy-Streamlit-Image-Filter-App.md) | [Next: (Conclusion or Next Steps) ->]
