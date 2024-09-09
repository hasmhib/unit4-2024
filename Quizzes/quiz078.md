# Quiz 078
<img width="max" alt="Screenshot 2024-09-09 at 10 29 27 PM" src="https://github.com/user-attachments/assets/c5869e5b-bfaf-434e-bca2-68db90e3bbb8">

## Code
![image](https://github.com/user-attachments/assets/c30722ba-0b35-46ae-b707-3145bc844aae)

```py
def position_parity(k):
    position = []
    for i in range(k):
        position.append(2**i - 1)
    return position

def get_msg(n, k, msg):
    len_msg = k + n
    out = []
    n = 0
    position = position_parity(k)

    for i in range(len_msg):
        out.append(-1)

    for i in range(len_msg):
        if i not in position:
            out[i] = msg[n]
            n += 1

    print(out)

get_msg(4, 3, [1, 0, 1, 1])
```

## Proof of work
<img width="max" alt="Screenshot 2024-09-09 at 10 46 20 PM" src="https://github.com/user-attachments/assets/47ab2f2d-69e4-4208-abaf-c6728abdede6">

