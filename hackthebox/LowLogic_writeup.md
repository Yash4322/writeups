# HTB Challenge Writeup: Low Logic

## Challenge Overview

The "Low Logic" challenge from Hack The Box presents a transistor-level circuit along with an `input.csv` file. The objective is to reverse-engineer the logic implemented by the circuit and compute the corresponding outputs for each row of inputs. The final result, when interpreted correctly, reveals the flag.

---

## Files Provided

* `chip.jpeg` – Image of the transistor-based circuit
* `input.csv` – CSV file containing multiple rows of binary inputs (`in0, in1, in2, in3`)

---

## Step 1: Understanding the Circuit

The circuit is constructed using BJTs (Bipolar Junction Transistors) and resistors. Instead of analyzing it at the electrical level, the correct approach is to abstract the circuit into digital logic.

### Key Observations

* Transistors in **series** behave like an **AND gate**
* Transistors in **parallel** behave like an **OR gate**

By grouping the transistors into functional blocks and analyzing the connections, the circuit simplifies to the following Boolean expression:

```
OUT = (IN0 AND IN1) OR (IN2 AND IN3)
```

---

## Step 2: Processing the Input Data

The provided `input.csv` contains multiple rows of binary values. Each row represents a set of inputs to the circuit.

Example format:

```
in0,in1,in2,in3
1,0,0,1
1,1,0,0
0,1,1,0
...
```

For each row:

1. Compute `(in0 AND in1)`
2. Compute `(in2 AND in3)`
3. OR the results
4. Append the output bit to a result string

---

## Step 3: Automating the Solution

To efficiently process all rows, a Python script can be used:

```python
import csv

with open("input.csv") as f:
    reader = csv.DictReader(f)
    result = ""

    for row in reader:
        out = (int(row['in0']) & int(row['in1'])) | (int(row['in2']) & int(row['in3']))
        result += str(out)

print(result)
```

Running this script produces a long binary string.

---

## Step 4: Converting Binary to ASCII

The resulting binary string represents ASCII-encoded text. To decode it:

```python
binary = "PASTE_YOUR_OUTPUT_HERE"

n = int(binary, 2)
flag = n.to_bytes((n.bit_length() + 7) // 8, 'big').decode()

print(flag)
```

---

## Final Flag

```
HTB{4_G00d_Cm05_3x4mpl3}
```

---

## Conclusion

This challenge demonstrates how transistor-level circuits can be abstracted into Boolean logic. Instead of analyzing electrical behavior in detail, recognizing structural patterns (series → AND, parallel → OR) allows rapid simplification.

The workflow is:

1. Reverse the circuit into a Boolean expression
2. Apply the logic to structured input data
3. Convert the resulting binary output into readable text

This approach is broadly applicable to hardware-based CTF challenges.
