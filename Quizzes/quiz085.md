# Quiz 085

<img width="max" alt="Screenshot 2024-10-01 at 1 54 01 PM" src="https://github.com/user-attachments/assets/c48c9c5d-47df-4e4b-850d-d7371e715e7b">

## Trace Table

![image](https://github.com/user-attachments/assets/a3e71811-8773-4de0-9ad2-6288bd2655b3)


## Code

```py
def MERGE_SORT(N):
    if len(N) == 1:
        return N
    else:
        mid = len(N) // 2
        L = MERGE_SORT(N[:mid])
        R = MERGE_SORT(N[mid:])

        S, i, j = [], 0, 0
        while i < len(L) and j < len(R):
            if L[i] < R[j]:
                S.append(L[i])
                i += 1
            else:
                S.append(R[j])
                j += 1

        S.extend(L[i:])
        S.extend(R[j:])

        return S


print(MERGE_SORT([3, 6, 7, 4]))
```

## Proof of work
<img width="max" alt="Screenshot 2024-10-01 at 1 55 55 PM" src="https://github.com/user-attachments/assets/58704274-4e4b-4e91-ba1b-4f7b0ec3980c">

