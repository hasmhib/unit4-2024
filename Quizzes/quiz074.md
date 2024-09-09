# Quiz 074

<img width="max" alt="Screenshot 2024-08-30 at 10 03 23 PM" src="https://github.com/user-attachments/assets/70ae926d-3b37-4b49-a272-79c123990edd">

## Code

![image](https://github.com/user-attachments/assets/46847570-c8ed-410f-a04e-e90ea98a9a6a)

```.py
def parity_check(data: str) -> bool:
    parity_bit = int(data[-1])
    data_bits = data[:-1]

    count_ones = data_bits.count('1')

    is_even_ones = (count_ones % 2 == 0)

    if (is_even_ones and parity_bit == 1) or (not is_even_ones and parity_bit == 0):
        return False
    else:
        return True


input = ["1001110010110011100110010110011100101", "011101111011101111011101111001111"]

outputs_parity = [parity_check(data) for data in input]
print(outputs_parity)
```

## Proof of work
<img width="max" alt="Screenshot 2024-09-09 at 8 50 10 PM" src="https://github.com/user-attachments/assets/ab10a666-4c36-4e8c-a33c-9eb08c064add">
