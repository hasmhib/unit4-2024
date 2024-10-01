# Quiz 084

<img width="max" alt="Screenshot 2024-10-01 at 1 52 13 PM" src="https://github.com/user-attachments/assets/a94c9c85-8c4b-41a6-991f-9ccece6a2886">

## Trace Table

![image](https://github.com/user-attachments/assets/dbbf0f79-6127-4c07-a55b-a9a4a411fa36)


## Code

```py
def FUNC(N):
    if N == 0:
        return 0
    if N == 1:
        return 1
    else:
        return FUNC(N - 1) + FUNC(N - 2)

print(FUNC(7))
```

## Proof of work
<img width="max" alt="Screenshot 2024-10-01 at 1 53 24 PM" src="https://github.com/user-attachments/assets/900b7d2d-e3f5-489f-b090-ae9b30c9e663">
