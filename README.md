# Neural Network Price Prediction — MQL4 Script

A MetaTrader 4 script that implements a **neural network price prediction framework** by loading a binary model file from the MT4 Files sandbox via `FileOpen(FILE_BIN | FILE_READ)`, populating a 10-element input feature vector from the last 10 closing prices, applying a configurable equal-weight linear inference computation as a scaffold for real model weights, comparing the predicted price against the current close via `MathAbs(predictedPrice − actualPrice) >= AlertThreshold`, and firing alerts when the deviation between prediction and reality exceeds the threshold — providing a complete integration architecture for external ML model inference inside MT4.

---

## Overview

True neural network inference inside MT4 is constrained by MQL4's lack of native tensor operations and deep learning libraries. The architecture this script implements — loading weights from a binary file, constructing an input feature vector from recent closes, applying a weighted sum inference — is the correct integration pattern for deploying externally trained models inside MT4. In the current implementation, the inference step uses equal weighting (`inputs[i] × 0.1`) as a functional placeholder that demonstrates the full data flow without requiring a trained model file. To deploy a real model, a developer would replace the weighting loop with a proper matrix multiplication reading actual trained weights from the binary model file, implementing whatever network architecture (feedforward, LSTM hidden state approximation, etc.) was used during training. The alert fires when the absolute difference between the model's output and the actual current close exceeds `AlertThreshold` — signalling that the model is predicting a meaningful price divergence from current levels.

---

## Features

- **Binary model file loading** — `FileOpen(ModelFile, FILE_BIN | FILE_READ)` opens the model from the MT4 sandbox; `handle < 0` guard aborts `PredictPrice()` with a descriptive log and returns `0.0` — no prediction fires on a missing model
- **10-bar input feature vector** — `double inputs[10]` populated via `iClose(symbol, timeframe, i)` for `i = 0` to `9`; represents the last 10 closing prices as the network's raw input features
- **Inference scaffold** — `for (int i = 0; i < 10; i++) output += inputs[i] × 0.1` computes a weighted sum with equal weights as a placeholder; replace with actual trained weight application reading from `handle` via `FileReadDouble()` per weight
- **`AlertThreshold` deviation gate** — `MathAbs(predictedPrice − actualPrice) >= AlertThreshold` fires when model diverges from current price by the configured minimum distance; default `0.0020` = 20 pips on a 4-digit EURUSD broker
- **Alert message includes both prices** — `AlertPrediction()` formats with `"Predicted Price: %.5f, Actual Price: %.5f"` for direct comparison
- **`FileClose(handle)`** called unconditionally after inference regardless of weight-reading success
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `actualPrice = iClose(TradeSymbol, Timeframe, 0)` fetched
2. `PredictPrice()` opens `ModelFile`, populates `inputs[10]` from recent closes, computes weighted sum, closes file, returns prediction
3. `MathAbs(predictedPrice − actualPrice) >= AlertThreshold` → **Significant Price Difference Detected**
4. `AlertPrediction()` dispatches via all enabled channels with both predicted and actual prices

---

## Extending to a Real Model

Replace the inference loop in `PredictPrice()` with actual weight application:

```mql4
// Example: Read weights from binary file and apply to inputs
double weights[10];
for (int i = 0; i < 10; i++) weights[i] = FileReadDouble(handle);
double output = 0.0;
for (int i = 0; i < 10; i++) output += inputs[i] * weights[i];
```

Train your model externally (Python/TensorFlow/PyTorch), export weights to a binary file, and place it in the MT4 `MQL4/Files/` sandbox as `ModelFile`.

---

## Input Parameters

| Parameter          | Type            | Default               | Description                                                         |
|--------------------|-----------------|------------------------|---------------------------------------------------------------------|
| `TradeSymbol`      | string          | `EURUSD`               | Symbol for analysis                                                 |
| `Timeframe`        | ENUM_TIMEFRAMES | `PERIOD_M15`           | Timeframe for price prediction                                      |
| `AlertThreshold`   | double          | `0.0020`               | Minimum `|predicted − actual|` to trigger alert (in price units)    |
| `ModelFile`        | string          | `neural_model.bin`     | Path to the binary model weights file in MT4 Files sandbox          |
| `EnableAlerts`     | bool            | `true`                 | Fire an on-screen/sound alert                                       |
| `EnableEmail`      | bool            | `false`                | Send an email notification                                          |
| `EnablePush`       | bool            | `false`                | Send a mobile push notification                                     |

---

## Alert Message Format

```
Significant Price Difference Detected detected on EURUSD (Timeframe: PERIOD_M15)
Predicted Price: 1.08640, Actual Price: 1.08412
```

---

## Installation

1. Copy `Neural_Network_Price_Prediction_001.mq4` to `MQL4/Scripts/`
2. Compile in MetaEditor (F7)
3. Place your model binary at `MQL4/Files/neural_model.bin` (or set `ModelFile` accordingly)
4. Drag onto any chart from Navigator → Scripts; configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)
- Binary model file present in MT4 Files sandbox

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
