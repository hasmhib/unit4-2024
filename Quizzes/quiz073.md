# Quiz 073

<img width="max" alt="Screenshot 2024-08-30 at 10 02 39 PM" src="https://github.com/user-attachments/assets/3dad70e7-e4d6-4053-9d14-7601431d2424">

## Code

![image](https://github.com/user-attachments/assets/4589fedd-324b-4a0a-a927-bc6113830fcf)

```.py
def error_check(data: str) -> bool:
    n = len(data) // 3
    original, copy1, copy2 = data[:n], data[n:2 * n], data[2 * n:]

    if original == copy1 and original == copy2 and copy1 == copy2:
        return False
    else:
        return True


data = ["100111001011001110010110011100101"]

outputs = error_check(data)
print(outputs)
```

## Proof of work
<img width="max" alt="Screenshot 2024-09-09 at 8 39 29 PM" src="https://github.com/user-attachments/assets/8fced978-ba83-4558-a5d9-362e3c9cc8b7">
