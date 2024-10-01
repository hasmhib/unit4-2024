# Quiz 081

<img width="max" alt="Screenshot 2024-10-01 at 1 46 11 PM" src="https://github.com/user-attachments/assets/56afb8bf-6b30-49dc-9706-43525efbcde2">

## Trace Table

![image](https://github.com/user-attachments/assets/75a9e1ef-8c7a-47ac-9bda-45cffaed48ac)


## Code

```py
def swap_letter(s: str, i: int, j: int) -> str:
    if i == j:
        return s
    s_list = list(s)
    s_list[i], s_list[j] = s_list[j], s_list[i]
    return ''.join(s_list)

def PERM(s: str, k: int):
    if k == len(s):
        return [s]
    else:
        out = []
        for i in range(k, len(s)):
            t = swap_letter(s, k, i)
            out.extend(PERM(t, k + 1))
        return out

N = 'AB'
print(PERM(N, 0))
```

## Proof of work
<img width="max" alt="Screenshot 2024-10-01 at 1 45 51 PM" src="https://github.com/user-attachments/assets/6ac73f56-83b3-4c7d-a673-8e079c046206">

