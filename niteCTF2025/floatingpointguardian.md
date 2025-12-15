# Floating Point Guardian
- the challenge provides the C source code for an ai gatekeeper program. the program asks 15 questions and expects 15 floating point answers, processes the answers through a hardcoded neural network, and computes a final value called `MASTER PROBABILITY`
- if this probability matches a fixed target value within a small tolerance, the program prints the flag
- so we need to find a set of 15 values that cause the network output to fall inside the allowed range
- all weights, biases, and activation functions are given
from the constants:
```
#define TARGET_PROBABILITY 0.7331337420
#define EPSILON 0.00001
```
the success condition is:
```
fabs(probability - TARGET_PROBABILITY) < EPSILON
```
## Network Structure
- we have 15 inputs
```
 → Hidden layer 1 (8 neurons, tanh)
 → Hidden layer 2 (6 neurons, tanh)
 → Output layer (1 neuron, sigmoid)
```
- there is no training, randomness or state. if we give the same inputs, the output will always be the same
## Input transformations
- each input is transformed before entering the first layer, depending on its index:
```
i % 4 == 0 → custom XOR-based activation

i % 4 == 1 → tanh(x)

i % 4 == 2 → cos(x)

i % 4 == 3 → sinh(x / 10)
```
- the XOR activation is the most interesting part:
```
(long)(x * 1000000) ^ key
```
- this converts a floating point value into an integer, XORs it with a constant, and converts it back. That operation breaks smoothness and makes the network output extremely sensitive to small changes in input.
- Key Observation
  - inputs are not validated in any way
  - floating-point values are compared directly against a target
  - the network is deterministic
  - the XOR step introduces discontinuities
- because of this, the problem is not about correct answers or realistic values.
- it is a numerical constraint problem, we need to push the output into a narrow floating-point window by choosing appropriate inputs.

## Strategy
- since all math is known and deterministic, the cleanest approach is to:
- reimplement the exact forward pass in Python
- generate candidate input vectors
- evaluate how close the output is to the target
- repeat until the error falls below the allowed epsilon
- this avoids guessing and keeps the process controlled and measurable.
## Solver Behavior
the python solver mirrors the C code exactly:
- same weights
- same activations
- same floating point behavior
it keeps track of the best error seen so far and prints progress periodically, so convergence is visible instead of opaque.
over time, the error consistently decreases as better candidates are found.
```
import math
import random
import time

INPUT_SIZE = 15
HIDDEN1_SIZE = 8
HIDDEN2_SIZE = 6

TARGET = 0.7331337420
EPS = 5e-6   # to match server tolerance

XOR_KEYS = [
    0x42, 0x13, 0x37, 0x99, 0x21, 0x88, 0x45, 0x67,
    0x12, 0x34, 0x56, 0x78, 0x9A, 0xBC, 0xDE
]

W1 = [
    [0.523, -0.891, 0.234, 0.667, -0.445, 0.789, -0.123, 0.456],
    [-0.334, 0.778, -0.556, 0.223, 0.889, -0.667, 0.445, -0.221],
    [0.667, -0.234, 0.891, -0.445, 0.123, 0.556, -0.789, 0.334],
    [-0.778, 0.445, -0.223, 0.889, -0.556, 0.234, 0.667, -0.891],
    [0.123, -0.667, 0.889, -0.334, 0.556, -0.778, 0.445, 0.223],
    [-0.891, 0.556, -0.445, 0.778, -0.223, 0.334, -0.667, 0.889],
    [0.445, -0.123, 0.667, -0.889, 0.334, -0.556, 0.778, -0.234],
    [-0.556, 0.889, -0.334, 0.445, -0.778, 0.667, -0.223, 0.123],
    [0.778, -0.445, 0.556, -0.667, 0.223, -0.889, 0.334, -0.445],
    [-0.223, 0.667, -0.778, 0.334, -0.445, 0.556, -0.889, 0.778],
    [0.889, -0.334, 0.445, -0.556, 0.667, -0.223, 0.123, -0.667],
    [-0.445, 0.223, -0.889, 0.778, -0.334, 0.445, -0.556, 0.889],
    [0.334, -0.778, 0.223, -0.445, 0.889, -0.667, 0.556, -0.123],
    [-0.667, 0.889, -0.445, 0.223, -0.556, 0.778, -0.334, 0.667],
    [0.556, -0.223, 0.778, -0.889, 0.445, -0.334, 0.889, -0.556]
]

B1 = [0.1, -0.2, 0.3, -0.15, 0.25, -0.35, 0.18, -0.28]

W2 = [
    [0.712, -0.534, 0.823, -0.445, 0.667, -0.389],
    [-0.623, 0.889, -0.456, 0.734, -0.567, 0.445],
    [0.534, -0.712, 0.389, -0.823, 0.456, -0.667],
    [-0.889, 0.456, -0.734, 0.567, -0.623, 0.823],
    [0.445, -0.667, 0.823, -0.389, 0.712, -0.534],
    [-0.734, 0.623, -0.567, 0.889, -0.456, 0.389],
    [0.667, -0.389, 0.534, -0.712, 0.623, -0.823],
    [-0.456, 0.823, -0.667, 0.445, -0.889, 0.734]
]

B2 = [0.05, -0.12, 0.18, -0.08, 0.22, -0.16]

W3 = [[0.923], [-0.812], [0.745], [-0.634], [0.856], [-0.723]]
B3 = [0.42]


def xor_activate(x, key):
    v = int(x * 1_000_000)
    v ^= key
    return v / 1_000_000.0


def forward(x):
    h1 = [0.0] * HIDDEN1_SIZE
    h2 = [0.0] * HIDDEN2_SIZE

    for j in range(HIDDEN1_SIZE):
        for i in range(INPUT_SIZE):
            if i % 4 == 0:
                a = xor_activate(x[i], XOR_KEYS[i])
            elif i % 4 == 1:
                a = math.tanh(x[i])
            elif i % 4 == 2:
                a = math.cos(x[i])
            else:
                a = math.sinh(x[i] / 10.0)
            h1[j] += a * W1[i][j]
        h1[j] = math.tanh(h1[j] + B1[j])

    for j in range(HIDDEN2_SIZE):
        for i in range(HIDDEN1_SIZE):
            h2[j] += h1[i] * W2[i][j]
        h2[j] = math.tanh(h2[j] + B2[j])

    out = sum(h2[i] * W3[i][0] for i in range(HIDDEN2_SIZE)) + B3[0]
    return 1.0 / (1.0 + math.exp(-out))


def solve():
    best_err = 1.0
    iters = 0
    start = time.time()

    while True:
        iters += 1
        x = [random.uniform(-1.5, 1.5) for _ in range(INPUT_SIZE)]
        y = forward(x)
        err = abs(y - TARGET)

        if err < best_err:
            best_err = err

        # sanity print every ~2 seconds
        if iters % 5000 == 0:
            elapsed = time.time() - start
            print(f"[{elapsed:6.1f}s] iterations={iters:,}  best_err={best_err:.8f}")

        if err < EPS:
            print("\nsolution found\n")
            print(f"probability = {y:.10f}\n")
            for i, v in enumerate(x):
                print(f"Q{i+1}: {v:.10f}")
            return


if __name__ == "__main__":
    solve()

```

```
shaunak@Shaunaks-MacBook-Pro Downloads % python3 mastersolve.py 
[   0.1s] iterations=5,000  best_err=0.00032686
[   0.3s] iterations=10,000  best_err=0.00032686
[   0.4s] iterations=15,000  best_err=0.00032686
[   0.6s] iterations=20,000  best_err=0.00021979
[   0.7s] iterations=25,000  best_err=0.00021979
[   0.8s] iterations=30,000  best_err=0.00021979
[   1.0s] iterations=35,000  best_err=0.00021979
[   1.1s] iterations=40,000  best_err=0.00021979
[   1.3s] iterations=45,000  best_err=0.00021979
[   1.4s] iterations=50,000  best_err=0.00021979
[   1.5s] iterations=55,000  best_err=0.00021979
[   1.7s] iterations=60,000  best_err=0.00021979
[   1.8s] iterations=65,000  best_err=0.00021979
[   2.0s] iterations=70,000  best_err=0.00021979
[   2.1s] iterations=75,000  best_err=0.00021979
[   2.3s] iterations=80,000  best_err=0.00021979
[   2.4s] iterations=85,000  best_err=0.00021979
[   2.5s] iterations=90,000  best_err=0.00021979
[   2.7s] iterations=95,000  best_err=0.00021979
[   2.8s] iterations=100,000  best_err=0.00021979
[   3.0s] iterations=105,000  best_err=0.00011296
[   3.1s] iterations=110,000  best_err=0.00011296
[   3.2s] iterations=115,000  best_err=0.00011296
[   3.4s] iterations=120,000  best_err=0.00011296
[   3.5s] iterations=125,000  best_err=0.00011296
[   3.7s] iterations=130,000  best_err=0.00011296
[   3.8s] iterations=135,000  best_err=0.00011296
[   3.9s] iterations=140,000  best_err=0.00011296
[   4.1s] iterations=145,000  best_err=0.00011296
[   4.2s] iterations=150,000  best_err=0.00011296
[   4.4s] iterations=155,000  best_err=0.00011296
[   4.5s] iterations=160,000  best_err=0.00011296
[   4.6s] iterations=165,000  best_err=0.00011296
[   4.8s] iterations=170,000  best_err=0.00011296
[   4.9s] iterations=175,000  best_err=0.00009598
[   5.1s] iterations=180,000  best_err=0.00009598
[   5.2s] iterations=185,000  best_err=0.00009598
[   5.4s] iterations=190,000  best_err=0.00009598
[   5.5s] iterations=195,000  best_err=0.00009598
[   5.6s] iterations=200,000  best_err=0.00009598
[   5.8s] iterations=205,000  best_err=0.00009598
[   5.9s] iterations=210,000  best_err=0.00009598
[   6.1s] iterations=215,000  best_err=0.00009598
[   6.2s] iterations=220,000  best_err=0.00009598
[   6.3s] iterations=225,000  best_err=0.00009598
[   6.5s] iterations=230,000  best_err=0.00009598
[   6.6s] iterations=235,000  best_err=0.00009598
[   6.8s] iterations=240,000  best_err=0.00009598
[   6.9s] iterations=245,000  best_err=0.00009598
[   7.0s] iterations=250,000  best_err=0.00009598
[   7.2s] iterations=255,000  best_err=0.00009598
[   7.3s] iterations=260,000  best_err=0.00009598
[   7.5s] iterations=265,000  best_err=0.00009598
[   7.6s] iterations=270,000  best_err=0.00009598
[   7.7s] iterations=275,000  best_err=0.00009598
[   7.9s] iterations=280,000  best_err=0.00009598
[   8.0s] iterations=285,000  best_err=0.00009598
[   8.2s] iterations=290,000  best_err=0.00009598
[   8.3s] iterations=295,000  best_err=0.00009598
[   8.5s] iterations=300,000  best_err=0.00009598
[   8.6s] iterations=305,000  best_err=0.00009598
[   8.7s] iterations=310,000  best_err=0.00009598
[   8.9s] iterations=315,000  best_err=0.00009598
[   9.0s] iterations=320,000  best_err=0.00009598
[   9.2s] iterations=325,000  best_err=0.00009598
[   9.3s] iterations=330,000  best_err=0.00009598
[   9.4s] iterations=335,000  best_err=0.00009598
[   9.6s] iterations=340,000  best_err=0.00009598
[   9.7s] iterations=345,000  best_err=0.00009598
[   9.9s] iterations=350,000  best_err=0.00009598
[  10.0s] iterations=355,000  best_err=0.00009598
[  10.2s] iterations=360,000  best_err=0.00009598
[  10.3s] iterations=365,000  best_err=0.00003417
[  10.4s] iterations=370,000  best_err=0.00003417
[  10.6s] iterations=375,000  best_err=0.00003417
[  10.7s] iterations=380,000  best_err=0.00003417
[  10.9s] iterations=385,000  best_err=0.00003417
[  11.0s] iterations=390,000  best_err=0.00003417
[  11.1s] iterations=395,000  best_err=0.00003417
[  11.3s] iterations=400,000  best_err=0.00003417
[  11.4s] iterations=405,000  best_err=0.00003417
[  11.6s] iterations=410,000  best_err=0.00003417
[  11.7s] iterations=415,000  best_err=0.00003417
[  11.8s] iterations=420,000  best_err=0.00003417
[  12.0s] iterations=425,000  best_err=0.00003417
[  12.1s] iterations=430,000  best_err=0.00003417
[  12.3s] iterations=435,000  best_err=0.00003417
[  12.4s] iterations=440,000  best_err=0.00003417
[  12.6s] iterations=445,000  best_err=0.00003417
[  12.7s] iterations=450,000  best_err=0.00003417
[  12.8s] iterations=455,000  best_err=0.00003417
[  13.0s] iterations=460,000  best_err=0.00003417
[  13.1s] iterations=465,000  best_err=0.00003417
[  13.3s] iterations=470,000  best_err=0.00003417
[  13.4s] iterations=475,000  best_err=0.00003417
[  13.5s] iterations=480,000  best_err=0.00003417
[  13.7s] iterations=485,000  best_err=0.00003417
[  13.8s] iterations=490,000  best_err=0.00003417
[  14.0s] iterations=495,000  best_err=0.00003417
[  14.1s] iterations=500,000  best_err=0.00003417
[  14.2s] iterations=505,000  best_err=0.00003417
[  14.4s] iterations=510,000  best_err=0.00003417
[  14.5s] iterations=515,000  best_err=0.00003417
[  14.7s] iterations=520,000  best_err=0.00003417
[  14.8s] iterations=525,000  best_err=0.00003417
[  15.0s] iterations=530,000  best_err=0.00003417
[  15.1s] iterations=535,000  best_err=0.00003417
[  15.2s] iterations=540,000  best_err=0.00003417
[  15.4s] iterations=545,000  best_err=0.00003417
[  15.5s] iterations=550,000  best_err=0.00003417
[  15.7s] iterations=555,000  best_err=0.00003417
[  15.8s] iterations=560,000  best_err=0.00003417
[  15.9s] iterations=565,000  best_err=0.00003417
[  16.1s] iterations=570,000  best_err=0.00003417
[  16.2s] iterations=575,000  best_err=0.00003417
[  16.4s] iterations=580,000  best_err=0.00003417
[  16.5s] iterations=585,000  best_err=0.00003417
[  16.6s] iterations=590,000  best_err=0.00003417
[  16.8s] iterations=595,000  best_err=0.00003417
[  16.9s] iterations=600,000  best_err=0.00003417
[  17.1s] iterations=605,000  best_err=0.00003417
[  17.2s] iterations=610,000  best_err=0.00003417
[  17.4s] iterations=615,000  best_err=0.00003417
[  17.5s] iterations=620,000  best_err=0.00003417
[  17.6s] iterations=625,000  best_err=0.00003417
[  17.8s] iterations=630,000  best_err=0.00003417
[  17.9s] iterations=635,000  best_err=0.00003417
[  18.1s] iterations=640,000  best_err=0.00003417
[  18.2s] iterations=645,000  best_err=0.00003417
[  18.3s] iterations=650,000  best_err=0.00003417
[  18.5s] iterations=655,000  best_err=0.00003417
[  18.6s] iterations=660,000  best_err=0.00003417
[  18.8s] iterations=665,000  best_err=0.00003417
[  18.9s] iterations=670,000  best_err=0.00003417
[  19.0s] iterations=675,000  best_err=0.00003417
[  19.2s] iterations=680,000  best_err=0.00003417
[  19.3s] iterations=685,000  best_err=0.00003417
[  19.5s] iterations=690,000  best_err=0.00003417
[  19.6s] iterations=695,000  best_err=0.00003417
[  19.7s] iterations=700,000  best_err=0.00003417
[  19.9s] iterations=705,000  best_err=0.00003417
[  20.0s] iterations=710,000  best_err=0.00003417
[  20.2s] iterations=715,000  best_err=0.00003417
[  20.3s] iterations=720,000  best_err=0.00003417
[  20.5s] iterations=725,000  best_err=0.00003417
[  20.6s] iterations=730,000  best_err=0.00001748
[  20.7s] iterations=735,000  best_err=0.00000562
[  20.9s] iterations=740,000  best_err=0.00000562
[  21.0s] iterations=745,000  best_err=0.00000562
[  21.2s] iterations=750,000  best_err=0.00000562
[  21.3s] iterations=755,000  best_err=0.00000562
[  21.4s] iterations=760,000  best_err=0.00000562
[  21.6s] iterations=765,000  best_err=0.00000562
[  21.7s] iterations=770,000  best_err=0.00000562
[  21.9s] iterations=775,000  best_err=0.00000562
[  22.0s] iterations=780,000  best_err=0.00000562
[  22.1s] iterations=785,000  best_err=0.00000562
[  22.3s] iterations=790,000  best_err=0.00000562
[  22.4s] iterations=795,000  best_err=0.00000562
[  22.6s] iterations=800,000  best_err=0.00000562
[  22.7s] iterations=805,000  best_err=0.00000562
[  22.9s] iterations=810,000  best_err=0.00000562
[  23.0s] iterations=815,000  best_err=0.00000562
[  23.1s] iterations=820,000  best_err=0.00000562
[  23.3s] iterations=825,000  best_err=0.00000562
[  23.4s] iterations=830,000  best_err=0.00000562
[  23.6s] iterations=835,000  best_err=0.00000562
[  23.7s] iterations=840,000  best_err=0.00000562
[  23.8s] iterations=845,000  best_err=0.00000562
[  24.0s] iterations=850,000  best_err=0.00000562
[  24.1s] iterations=855,000  best_err=0.00000562
[  24.3s] iterations=860,000  best_err=0.00000562
[  24.4s] iterations=865,000  best_err=0.00000562
[  24.6s] iterations=870,000  best_err=0.00000562
[  24.7s] iterations=875,000  best_err=0.00000562

solution found

probability = 0.7331355316

Q1: -1.2346058965
Q2: -0.2444725259
Q3: 1.4461183791
Q4: 1.2347870345
Q5: -0.1407737252
Q6: -0.0106305865
Q7: -0.5897033189
Q8: -0.7407747669
Q9: -1.1468904378
Q10: 0.8206268121
Q11: -0.4358976325
Q12: -1.3481025140
Q13: -0.1117186061
Q14: 1.3050089467
Q15: -1.1743599102
shaunak@Shaunaks-MacBook-Pro Downloads % 
```
```
┌──(shaggy㉿kali)-[~]
└─$ ncat --ssl floating.chals.nitectf25.live 1337
I am the AI Gatekeeper.
Enter your details so I know you are my Master.
Answer these questions with EXACT precision...

[Q1]  What is your height in centimeters? -1.2346058965
-1.2346058965
[Q2]  What is your weight in kilograms? -0.2444725259
-0.2444725259
[Q3]  What is your age in years? 1.4461183791
1.4461183791
[Q4]  What is your heart rate (bpm)? 1.2347870345
1.2347870345
[Q5]  How many hours do you sleep per night? -0.1407737252
-0.1407737252
[Q6]  What is your body temperature in Celsius? -0.0106305865
-0.0106305865
[Q7]  How many steps do you walk per day? -0.5897033189
-0.5897033189
[Q8]  What is your systolic blood pressure? -0.7407747669
-0.7407747669
[Q9]  How many calories do you consume daily? -1.1468904378
-1.1468904378
[Q10] What is your BMI (Body Mass Index)? 0.8206268121
0.8206268121
[Q11] How many liters of water do you drink daily? -0.4358976325
-0.4358976325
[Q12] What is your resting metabolic rate (kcal/day)? -1.3481025140
-1.3481025140
[Q13] How many hours do you exercise per week? -0.1117186061
-0.1117186061
[Q14] What is your blood glucose level (mg/dL)? 1.3050089467
1.3050089467
[Q15] Rate this CTF challenge out of 10: -1.1743599102
-1.1743599102


Processing through neural network layers...
========================================
MASTER PROBABILITY: 0.7331355315
========================================
You are the master! Here is your flag:
nite{br0_i5_n0t_g0nn4_b3_t4K1n6_any1s_j0bs_34x}

```
