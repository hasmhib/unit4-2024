# Quiz 080

<img width="max" alt="Screenshot 2024-10-01 at 1 37 43 PM" src="https://github.com/user-attachments/assets/23b7264b-9084-4a28-8f65-2d0a5c46a3a8">

## Trace Table



## Code

```py
def function(N: list[int]) -> int:
    if len(N) == 1:
        return N[0]
    else:
        M = function(N[1:])
        if N[0] > M:
            return N[0]
        else:
            return M


print(function(N=[4, 5, 8, 7]))
```


## Proof of work
<img width="max" alt="Screenshot 2024-10-01 at 1 42 28 PM" src="https://github.com/user-attachments/assets/c5d12d90-8310-4676-9616-a094ad65b584">
