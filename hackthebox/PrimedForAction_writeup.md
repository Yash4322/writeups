# HTB Challenge Writeup – *Primed for Action*

## Challenge Overview

This challenge presents a scenario where intelligence units intercepted a list of numbers. Among mostly irrelevant values, **exactly two numbers are prime**, and these primes are used to form a key.

### Objective:

* Identify the **two prime numbers** in the list
* Compute their **product**
* Return the result as the final answer

---

##  Accessing the Challenge

1. Connect to the HTB VPN using OpenVPN:

   ```bash
   sudo openvpn <your_vpn_file>.ovpn
   ```

2. Navigate to the provided IP address in your browser.

3. The platform presents a coding interface where input is provided and output is expected.

---

##  Sample Input

```
2 6 7 18 6
```

##  Expected Output

```
14
```

---

##  Approach

### Step 1: Understand the Pattern

* The input is a list of integers.
* Only **two numbers are prime**.
* The rest are noise (non-prime).

### Step 2: Identify Prime Numbers

We iterate through the list and check each number using a **primality test**:

* Numbers < 2 → Not prime
* Even numbers > 2 → Not prime
* Check divisibility up to √n

### Step 3: Compute the Product

* Multiply the two detected prime numbers.

---

##  Exploit / Solution Code

```python
def is_prime(n):
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    i = 3
    while i * i <= n:
        if n % i == 0:
            return False
        i += 2
    return True


nums = list(map(int, input().split()))
primes = [x for x in nums if is_prime(x)]

print(primes[0] * primes[1])
```

---

##  Execution

1. Paste the code into the challenge interface.
2. Run the program.
3. The output will be automatically validated by the platform.

---

##  Key Takeaways

* Basic number theory (prime detection) can appear in CTFs.
* Always filter meaningful data from noise.
* Efficient primality checks are important for performance.

---

##  Final Thoughts

This was a simple warm-up style challenge focusing on logic rather than exploitation. It reinforces the importance of:

* Pattern recognition
* Clean implementation
* Handling input/output correctly in constrained environments

---

✅ *Challenge Solved Successfully*
