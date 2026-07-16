# 🤖 Understanding Python's `subprocess`
## Bridging the Gap Between Python and the Terminal

Before we mix Python and CDO, it is highly important to understand exactly what the `subprocess` library is doing. 

---

## 🏗️ 1. What is `subprocess`?

Imagine you have an assistant. You are busy writing Python code in your Jupyter Notebook. Suddenly, you need to run a CDO command in the Terminal (Anaconda Prompt). 

Instead of opening the Terminal yourself and typing the command, you hand a sticky note to your assistant and say: *"Go open the terminal, type exactly what is on this note, hit Enter, and tell me if it worked."*

**`subprocess` is that assistant.** 

It is a built-in Python library that allows your Python script to spawn a "child process" (a hidden terminal), execute a terminal command, and wait for the result.

---

## 🛠️ 2. The Syntax: `subprocess.run()`

The most common and modern way to use this library is the `subprocess.run()` function.

### The Basic Example (Running a simple command)

If you wanted to run the `echo "Hello World"` command in the terminal, you would do this in Python:

```python
import subprocess

# We pass the command as a LIST of strings. 
# ["command", "argument1", "argument2"]
subprocess.run(["echo", "Hello World"])
```

### 🌍 The Climate Example (Running CDO)

If you want to run this CDO command in the terminal:
`cdo yearmean input.nc output.nc`

You translate it into Python like this:
```python
import subprocess

# Break the command apart at the spaces, and put it in a list!
command = ["cdo", "yearmean", "input.nc", "output.nc"]

# Run it!
subprocess.run(command)
```
When Python hits this line, it will freeze for a moment, open a hidden terminal, run the CDO calculation, save `output.nc` to your computer, and then Python will resume to the next line of your code.

---

## 🎛️ 3. Important Parameters

By default, `subprocess.run()` is "blind". If the terminal throws an error, your Python code might not notice, which can cause massive headaches later. We use a few key parameters to make it safe.

### A. `check=True` (The Safety Net)
If you run `cdo yearmean input.nc output.nc` but `input.nc` doesn't exist, CDO will fail. If you don't use `check=True`, Python will just shrug, assume everything went fine, and move to the next line (which will crash because `output.nc` wasn't created).

By adding `check=True`, Python will intentionally crash and show you the exact error if the CDO command fails.

```python
subprocess.run(["cdo", "yearmean", "input.nc", "output.nc"], check=True)
```

### B. `capture_output=True` and `text=True` (Reading the Terminal)
Sometimes you want to see exactly what CDO printed in the terminal. For example, if you run `cdo info input.nc`, you want Python to read that information.

```python
result = subprocess.run(["cdo", "sinfo", "input.nc"], capture_output=True, text=True)

# Now you can print exactly what the terminal would have shown!
print(result.stdout)
```

---

## 🛡️ 4. The "Best Practice" Template for Climate Data

When writing production climate scripts, we wrap `subprocess.run` inside a `try/except` block. This prevents our Jupyter notebook from displaying a massive, ugly red error trace if something goes wrong.

Here is the exact template we use when combining Python and CDO:

```python
import subprocess
import os

input_file = "tasmax_data.nc"
output_file = "txx_annual.nc"

# Step 1: Check if we already did the math! No need to run CDO twice.
if not os.path.exists(output_file):
    print(f"Running CDO on {input_file}...")
    
    try:
        # Step 2: Run the terminal command safely
        subprocess.run(["cdo", "eca_txx", input_file, output_file], check=True)
        print("Success! CDO finished calculating.")
        
    except FileNotFoundError:
        # This happens if Python cannot find the 'cdo' program installed on your PC
        print("Error: CDO is not installed or not in your system PATH.")
        
    except subprocess.CalledProcessError as e:
        # This happens if CDO crashes (e.g., input file is missing or corrupted)
        print(f"CDO crashed! Make sure {input_file} is in the same folder.")
        
else:
    print(f"{output_file} already exists. We can skip the CDO calculation!")
```

## 🎯 Summary
1. `subprocess` allows Python to run terminal commands for you.
2. You break your terminal command into a Python list: `["cdo", "operator", "input", "output"]`.
3. You always use `check=True` so Python stops if the terminal command fails.
4. It is the ultimate tool for automating workflows, allowing you to use CDO for heavy math and Python for plotting, all inside a single script.
