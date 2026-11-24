# Track `stocks`: forecasting tomorrow's Saudi Aramco share price

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/snow10100/aie113-projects/blob/main/tracks/stocks/notebook.ipynb)
&nbsp; [العربية](README.ar.md)

> **A teaching example, not investment advice.** This model must never be used to buy, sell
> or recommend any share.

## 1. What this model does

The model looks at the last few weeks of Saudi Aramco's share price and guesses the closing
price of the next trading day. Think of the weather: "tomorrow will be like today" is already
a very good forecast, and it is hard to do better. Our model tries to beat "tomorrow's price =
today's price" by spotting small patterns in the recent moves. The honest result: it does not
manage to. That is the most useful lesson of this track.

## 2. The data

- **What:** daily prices of Saudi Aramco (symbol `2222.SR`) on the Saudi Exchange (Tadawul),
  from Yahoo Finance: <https://finance.yahoo.com/quote/2222.SR/history/>. The notebook
  downloads it with the open-source library [`yfinance`](https://github.com/ranaroussi/yfinance)
  (Apache 2.0 licence). No account or key is needed.
- **Licence:** the prices belong to Yahoo and its data providers. Yahoo's terms allow personal
  use only, and `yfinance` is an unofficial tool meant for research and education. That is
  why the repository never stores price data: each person downloads it when the notebook runs.
- **How big:** 1,246 trading days, from 1 November 2021 to 29 October 2026 (the dates are
  fixed in the notebook so everyone gets the same numbers). 31 empty rows (zero volume, flat
  price: days where Yahoo recorded no trading) are dropped, leaving 1,215 days.
- **One row** is one trading day (Sunday to Thursday): the opening, highest, lowest and closing
  price in SAR, and the number of shares traded. We only use the **closing price**. Prices are
  corrected for the bonus shares of 2022 and 2023 (one free share for every ten), so old and
  new prices can be compared. They are not corrected for dividends: on the first day a share
  trades without the right to the next dividend, the price drops by about that amount. We do
  not use the volume, because Yahoo's
  volume numbers are clearly wrong from March to June 2023.

## 3. How the model learns

For every day the notebook computes eight simple numbers about the recent past, using only
what was known that evening: today's move, the moves of the two days before, the move over the
last week, how far the price is from its 5-day and its 20-day average, how jumpy the last ten
days were, and the day of the week.

The model is a **ridge regression**: it gives each of the eight numbers a weight, multiplies
each number by its weight and adds everything up to get "tomorrow's move in percent", which becomes a price. It learns
the weights from 950 days of history (November 2021 to October 2025) so that its guesses land
as close as possible to what really happened. "Ridge" means it is also held back from giving
big weights, so that it does not chase noise. How strongly it is held back is chosen by
testing on older years only (always learning on earlier days and checking on later ones). It
chose to be held back strongly, which already tells us there is little signal to find.

The **baseline** is the naive guess: tomorrow's closing price = today's closing price. Every
forecasting model must be compared with it.

## 4. How we know if it is any good

The last 12 months (2 November 2025 to 28 October 2026, 245 trading days) are kept apart and
never shown to the model while it learns. On those days we measure the **average error**: how
far, in SAR, the forecast was from the real closing price (MAE).

| | Average error (MAE) | Average error in % (MAPE) |
|---|---|---|
| Ridge model | 0.1732 SAR (17.3 halalas) | 0.67% |
| Naive guess "tomorrow = today" | 0.1712 SAR (17.1 halalas) | 0.66% |

- The model is **1.2% worse** than the naive guess. It does not beat it.
- On the 226 test days when the price moved, the model guessed the direction (up or down)
  right **51.3%** of the time. A coin flip gets about 50%.
- A typical day moves the price by about 17 halalas, while the model's inputs move its
  forecast by about 2 halalas at most. The model has learned that the recent past says very
  little about tomorrow.

What this means for a user: the forecast is "today's price, give or take a couple of halalas".
That is no better than reading today's price yourself. For a big, busy share like Aramco, the
news that will move tomorrow's price is not in yesterday's prices. A chart where the forecast
line hugs the real price can look impressive, but the naive guess hugs it just as well: always
compare with the baseline.

### Why did the model say that?

For a ridge model a forecast is an exact sum: a starting point (the average daily move in
the training years) plus one push per input. Each push is the input's weight times how far
today's value is from its usual level, and the notebook shows it in halalas. Example from the
notebook: after the close of Thursday 29 October 2026 (25.92 SAR), the model forecast 25.895
SAR for the next trading day, 2.5 halalas lower. The biggest push down came from the move two
days earlier (+1.87%, pushing −1.2 halalas: after a rise the model expects a small step back);
the biggest push up came from the price sitting above its 5-day average (+0.5 halalas). Over
all forecasts, today's move matters most (a rise today nudges tomorrow's forecast up, about
2 halalas for a typical move).

## 5. Limits and risks

- It does not know the news: company results, oil prices, interest rates, market holidays or
  dividend dates. It is wrong on exactly the days that matter most.
- It knows only one share and only the years it was trained on (up to October 2025).
- Free data has gaps and errors (the empty days, the broken 2023 volume); a real service would
  need a licensed data feed.
- It is no better than "tomorrow = today". It must **never** be used to trade, to give
  investment advice, to publish "price targets", or to make decisions about anyone's money.

## 6. What the notebook does, step by step

1. Installs what Colab lacks (an up-to-date `yfinance`).
2. Sets the share, the dates, the test period and the file paths as constants.
3. Downloads the daily prices from Yahoo Finance (with one retry).
4. Takes a quick look, drops the empty days, and plots the price.
5. Builds the eight inputs and the target (tomorrow's move).
6. Splits by time: about four years to learn, the last 12 months to test.
7. Trains the ridge model, choosing how strongly to hold it back on older years only.
8. Compares it with the naive guess on the test months, with a small chart.
9. Saves the model and a facts file into `artifacts/`.
10. Reloads the model and forecasts the next trading day from the latest prices.
11. Explains that forecast (which inputs pushed it up or down) and which inputs matter overall.

## 7. What the notebook leaves in `artifacts/`

- `model.joblib`: the trained scikit-learn pipeline (input scaling + ridge), about 2 KB.
- `model_meta.json`: the model version, the ticker, the eight input names, the chosen
  strength, the training and test date ranges and sizes, and the test numbers for the model
  and the naive guess.

`artifacts/` is ignored by git: each run creates it again.

## 8. Project ideas for this track

**The API.** A `POST /forecast` endpoint. The request names a ticker; this model only knows
`2222.SR`, so any other ticker gets a clear rejection. The service fetches the latest prices
and answers with the date of the latest price used, the last close, the model's forecast for
the next trading day, the naive forecast next to it, the change in halalas, the top two or
three reasons, the model version, and the "not investment advice" note. A `GET /health` and a
`GET /model-info` (the contents of `model_meta.json`) complete it.

**Model-behaviour tests worth writing.**
- Only the past counts: the forecast for a given day does not change if you delete all the
  days after it.
- The forecast always stays close to today's price (for example within 3%), whatever
  realistic prices you send.
- Multiplying every price by 2 doubles the forecast, because the inputs are relative moves.
- Thirty days of a perfectly flat price give a forecast within a halala or so of that price.

**A secret in this track.** Yahoo needs no key, but a real service would buy a licensed market
data feed, which comes with an API key. Your own `/forecast` endpoint could also be protected
by a key. Both belong in environment variables and GitHub secrets, never in the code.

**A React front end.** One page with the last 60 closing prices as a line, tomorrow's
forecast and the naive guess as two dots, a small bar chart of the reasons, and the
disclaimer always visible.

## 9. How to open it

- **Colab:** [open the notebook in Colab](https://colab.research.google.com/github/snow10100/aie113-projects/blob/main/tracks/stocks/notebook.ipynb)
  and choose *Runtime → Run all* (a CPU is enough; it runs in under a minute).
- **Local Jupyter** (Python 3.10 to 3.12), from this folder:

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt jupyter
jupyter notebook notebook.ipynb
```
