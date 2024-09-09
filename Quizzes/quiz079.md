# Quiz 079


## Code

![image](https://github.com/user-attachments/assets/adcdfa8b-422b-490b-8267-9da6cd876c0c)


```py
def get_equation(k: int, n: int, p: int) -> list:
    out = []
    for i in range(1, k + n + 1):
        if (i & (1 << (p - 1))) != 0:
            out.append(i - 1)
    return out


def position_parity(k):
    position = []
    for i in range(k):
        position.append(2 ** i - 1)
    return position


def get_msg(n, k, msg):
    len_msg = k + n
    out = [-1] * len_msg
    msg_index = 0
    parity_pos = position_parity(k)

    for i in range(len_msg):
        if i not in parity_pos:
            out[i] = msg[msg_index]
            msg_index += 1

    return out


def build_data(n, k, msg):
    out = get_msg(n, k, msg)
    parity_pos = position_parity(k)

    for p in parity_pos:
        total = 0
        equation_pos = get_equation(k, n, p + 1)
        for i in range(1, len(equation_pos)):
            if out[equation_pos[i]] != -1:
                total += out[equation_pos[i]]

        if total % 2 == 0:
            out[p] = 0
        else:
            out[p] = 1


    print(out)


n = 4
k = 3
msg = [1, 0, 1, 1]

build_data(n, k, msg)
```

## Proof of work
<img width="max" alt="Screenshot 2024-09-09 at 11 09 19 PM" src="https://github.com/user-attachments/assets/7d37de13-4520-4797-b174-8d5cc07118a3">

