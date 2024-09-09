# Quiz 077

<img width="924" alt="Screenshot 2024-09-09 at 10 13 45 PM" src="https://github.com/user-attachments/assets/19cc4097-2527-4f12-bb7d-07fb6eb581c8">

## Code

![image](https://github.com/user-attachments/assets/16c4aed5-8195-4277-bae8-039ac4ccac0a)


```py
def get_equation(k: int, n: int, p: int) -> list:
    out = []
    for i in range(1, k + n + 1):
        if (i & (1 << (p - 1))) != 0:
            out.append(i - 1)
    print(out)

get_equation(3, 4, 1)
get_equation(3, 4, 2)
get_equation(3, 4, 3)

```

## Proof of work
<img width="max" alt="Screenshot 2024-09-09 at 10 26 33 PM" src="https://github.com/user-attachments/assets/8d19541f-b5f4-4c05-b516-eb98a1c49d18">
