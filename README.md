# hft-verilog-generator
## Intro
Convert json descriptions of quant algorithms to verilog HDL and backtest the hardware on price time series data. 
## Usage
### Requirements 
- icarus verilog compiler
- gtkwave
- python 3.6 or greater <br>
### Summary
Given .csv data `src/generator/gen_testbench.py` generates a raw input hex file and a verilog test bench used to input the data signals to the fpga hardware. Given a json formatted description of a rule based quantitative algorithm `src/generator/gen_verilog.py` generates a verilog module describing the behavior in hardware. When compiled with iverilog the verilog test bench will generate an output file of buy/sell/hold signals over time which can be analyzed by `src/analysis/profit_analyzer.py` to discover statistics about the algorithm such as total percent profit and average percent profit per trade (buy, sell).
### File Formats
CSV Data: <br>
```
field1, field2, ...
data1, data2, ...
...
```
JSON Algo Description: <br>
```
{
  "Signals": ["signal1", "signal2", ...],
  "Constants": {
      "constant1": float literal,
      "constant2": float literal
  },
  "Buy": {
      "Conditions": {
          "c1": ["signal/constant", "<>", "signal/constant"],
          "c2": ["signal/constant", "<>", "signal/constant"],
          ...
      },
      "Logic": "[boolean expression on conditions, true when algo should indicate buy]"
  },
  "Sell": {
      "Conditions": {
          "c1": ["signal/constant", "<>", "signal/constant"],
          "c2": ["signal/constant", "<>", "signal/constant"],
          ...
      },
      "Logic": "[boolean expression on conditions, true when algo should indicate sell]"
  }
}
```
The signals in the json file should match the fields in the .csv that will be used for backtesting. Refrain from using verilog keywords like module, time, begin, etc...
### FPU
All designs are powered by the FPU:
![fp_gt_schematic](https://user-images.githubusercontent.com/22607081/86496693-22f30880-bd44-11ea-8c3d-ed248f331ff9.png)

### Example 1 (Basic RSI based decision)
JSON:
```
{
    "Signals": ["RSI"],
    "Constants": {
        "low": 30,
        "high": 70
    },
    "Buy": {
        "Conditions": {
            "c1": ["RSI", "<", "low"]
        },
        "Logic": "c1"
    },
    "Sell": {
        "Conditions": {
            "c1": ["RSI", ">", "high"]
        },
        "Logic": "c1"
    }
}
```
Generated Schematic:
<img width="1021" alt="rsi_schematic" src="https://user-images.githubusercontent.com/22607081/86496714-31412480-bd44-11ea-9e1f-6a90c4921acc.png">
### Example 2 (Ichimoku Cloud based strategy)
JSON:
```
{
    "Signals": ["PRICE", "SSA", "SSB", "KS", "TS"],
    "Constants": {
        "low": 30,
        "high": 70
    },
    "Buy": {
        "Conditions": {
            "c1": ["PRICE", "<", "SSA"],
            "c2": ["PRICE", "<", "SSB"],
            "c3": ["SSA", ">", "SSB"],
            "c4": ["PRICE", ">", "KS"],
            "c5": ["TS", ">", "KS"]
        },
        "Logic": "! ( c1 | c2 ) | c3 | c4 | c5"
    },
    "Sell": {
        "Conditions": {
            "c1": ["PRICE", "<", "SSA"],
            "c2": ["PRICE", "<", "SSB"],
            "c3": ["SSA", "<", "SSB"],
            "c4": ["PRICE", "<", "KS"],
            "c5": ["TS", "<", "KS"]
        },
        "Logic": "( c1 & c2 ) | c3 | c4 | c5"
    }
}
```
![ichimoku_schematic](https://user-images.githubusercontent.com/22607081/86496704-2c7c7080-bd44-11ea-8925-79957956b1b5.png)

## Sample Results
## Credits
