# Quiz 083

<img width="max" alt="Screenshot 2024-10-01 at 1 49 37 PM" src="https://github.com/user-attachments/assets/5bb4da34-05c1-4e0b-a13d-bd4a32461b23">

## Trace Table

![image](https://github.com/user-attachments/assets/4f975acc-6e45-480f-bf8a-589e9937b0a5)


## Code

```py
def FUNC(N):
    if len(N) == 1:
        return N[0]
    else:
        mid = len(N) // 2
        L = FUNC(N[:mid])
        R = FUNC(N[mid:])
        return L + R


print(FUNC([3, 6, 7, 4]))

```

## Proof of work
<img width="max" alt="Screenshot 2024-10-01 at 1 51 20 PM" src="https://github.com/user-attachments/assets/e7b829ea-28de-4cde-a1a7-f2db0c03977c">
