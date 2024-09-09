# Quiz 075

<img width="max" alt="Screenshot 2024-09-02 at 11 57 10 PM" src="https://github.com/user-attachments/assets/56cce0f2-ddc4-4e7d-9f3f-c2c6c10e136a">

## Code

![image](https://github.com/user-attachments/assets/496ce86e-53ad-419c-bb9a-2f34b3a60a04)

```.py
import matplotlib.pyplot as plt

def hamming_code(n: int) -> int:
    k = 0
    while 2 ** k <= k + n + 1:
        k += 1
    return k

x = []
y = []


for n in range(1, 1000):
    k = hamming_code(n)
    x.append(n)
    y.append(k)

plt.figure(figsize=(10, 6))
plt.plot(x, y)

plt.show()
```

## Proof of work
<img width="max" alt="Screenshot 2024-09-02 at 11 58 21 PM" src="https://github.com/user-attachments/assets/fe422889-1af2-482a-97d9-d568594b3745">
