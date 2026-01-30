+++
date = '2026-01-30T21:00:13+02:00'
draft = false
title = 'Running LLMs on a Raspberry Pi Zero 2W'
+++

In this tutorial I will show you how to run LLMs locally on a $15 computer, Raspberry Pi Zero 2W with llama.cpp.

### Versions used in this tutorial
- [llama.cpp](https://github.com/ggml-org/llama.cpp) version b7886-f3bc98890
- Raspberry Pi OS, Legacy Lite 64-bit, (Debian 12) 

## Pre-requisites

- Raspberry Pi Zero 2W
- Debian 12 64-bit, other versions might work as well
- ~2GB of free space

I highly recommend using zram as it in most cases improves the performance. You can find many instructions online, I recommend [this one](https://pimylifeup.com/raspberry-pi-zram/).

## Installing llama.cpp and dependencies

1. Install dependencies via the apt package manager
```bash
sudo apt install libcurl4-openssl-dev libssl-dev git cmake build-essential
```

2. Clone the llama.cpp repository
```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp
```

3. Build the llama.cpp

```bash
cmake -B build -DBUILD_SHARED_LIBS=OFF -DGGML_USE_MMAP=1
```
This will make the build directory with these specific flags. It'll build as static library and will use mmap for reducing memory usage.

```bash
cmake --build build --config Release
```

This will build the llama.cpp with release mode(longer build time due to optimizations, but results in faster performance).
Build as single threaded due to the limited memory.
Note: This will take some time. In my case it took around 3 hours. You can also cross-compile it on a much powerful machine, but we won't do it in this tutorial.


```bash
sudo cmake --install build --prefix /usr/local
```

This step is optional, but if you want to use the llama.cpp without adding it to the path.

## Running LLMs

Now when everything is ready, we can run the LLMs. You can mostly use <500M models with the pi.
Expect the models not to be ChatGPT-like quality, more for testing purposes.

I used LFM2-350M (at Q4_K_M quantization) for this testing.
I also tried to run LFM2 VL 450M(at Q8_0) and Qwen3 0.6B(at Q4_K_M), but they were too slow in my case.

It's necessary to limit the context length, otherwise it will run out of memory when trying to allocate the kv cache.
To do that, you can use the `--ctx-size <value>` flag. I'll recommend a context length between 2000-8000, but you can experiment with different values and use thay that work best for you.

To run local gguf models, use ```bash llama-cli --ctx-size <value> model.gguf```

### Equation example

Ran with ```bash llama-cli --ctx-size 8000 -hf LiquidAI/LFM2-350M-GGUF:Q4_K_M```

> Solve x: -3x = 3+17x-76

To solve the equation -3x = 3 + 17x - 76, follow these steps:

1. **Move all terms to one side of the equation**:
  $$\[-3x - 17x = 3 - 76\]$$

2. **Combine like terms on the left side**:
  $$\[-20x = -73\]$$

3. **Divide both sides by the coefficient of x (which is -20)**:
  $$\[x = \frac{-73}{-20}\]$$

4. **Simplify the fraction**:
  $$\[x = \frac{73}{20}\] $$

Therefore, the solution to the equation is x = 73/20.

[ Prompt: 2.1 t/s | Generation: 6.4 t/s ]

In this example I asked the LLM to solve the equation -3x = 3 + 17x - 76. It was surprisingly fast and got it right. But what I've observed is that more complex the question, the slower it gets. When trying asking the LLM more complex questions, it got the answers wrong.

### Code examples

Both ran with ```bash llama-cli --ctx-size 8000 -hf LiquidAI/LFM2-350M-GGUF:Q4_K_M```

> Write a python program that asks for two numbers, adds them and prints the sum

Here's a simple Python program that asks for two numbers, adds them and prints the sum:

```python
# Ask for the first number
num1 = float(input("Enter the first number: "))

# Ask for the second number
num2 = float(input("Enter the second number: "))

# Add the two numbers
total = num1 + num2

# Print the result
print("The sum of ", num1, " and ", num2, " is: ", total)
```

This program uses the `input()` function to get two numbers from the user, 
converts them to floats to handle both integers and decimals, and then sums them together
before printing out the result. The `print()` function then outputs the sum to the console.

[ Prompt: 10.5 t/s | Generation: 6.7 t/s ]

As expected, this simple program wasn't much of a challenge for the LLM.
Now let's try something a bit more complex.

> Write a program in C that asks for a number, then checks if it's prime and prints the result  
                
```c
#include <stdio.h>

// Function to check if a number is prime
int isPrime(int num) {
    if (num <= 1) {
        return 0; // Number less than or equal to 1 is not prime
    }
    for (int i = 2; i <= sqrt(num); i++) {
        if (num % i == 0) {
            return 0; // Number can be divided by some number less than itself
        }
    }
    return 1; // Number is prime
}

int main() {
    int num;

    // Prompt user for a number
    printf("Enter a number: ");
    scanf("%d", &num);

    // Check if prime
    if (isPrime(num)) {
        printf("%d is a prime number.\n", num);
    } else {
        printf("%d is not a prime number.\n", num);
    }

    return 0;
}
```

**Explanation:**

1. **Include Header File:**
  - `#include <stdio.h>` includes the standard input/output library for using `scanf` and `printf`.
 
2. **Prime Check Function (`isPrime`):**
  - This function takes an integer `num` as input.
  - It first checks if `num` is less than or equal to 1. Numbers less than or equal to 1 are not prime.
  - It then iterates from 2 to the square root of `num` (inclusive). If `num` is
divisible by any of these values, it's not prime, and the function returns 0.
  - If the loop completes without finding divisors, the function returns 1, indicating the number is prime.

3. **Main Function:**
  - Declares an integer variable `num` to store user input.
  - Prompts the user to enter a number using `printf`.
  - Uses `scanf` to read the input as an integer.
  - Calls the `isPrime` function to check if the number is prime.
  - Prints the result using `printf` based on the `isPrime` function's output.

**How it Works:**

The program first checks if the input number is less than or equal to 1 (not prime
  ). Then, it efficiently checks divisibility from 2 up to the square root of the
  number using a faster algorithm. This approach optimizes performance for larger
  numbers.

[ Prompt: 10.8 t/s | Generation: 6.5 t/s ]

As expected, this program was a bit more challenging for the LLM. It forgot to include the `math.h` header file to use the `sqrt` function, and `-lm` flag is missing.
To fix this, I added `#include <math.h>` and `-lm` to the command line(example: `gcc -lm main.c -o main`).

To sum these up, it's impressive to see this small LLM to produce somewhat working code.

## Conclusion

While you can run LLMs on a Raspberry Pi Zero 2W, it's not recommended for general use.
This is a great experiment to try LLMs on a small device and I'd recommend trying it out.
You can do this with other Pis too(and run larger models), the instructions are mostly the same.

Also the LFM2-350M impressed at coding, but this wasn't a LLM review.

If you have any questions or feedback, please let me know in the comments.