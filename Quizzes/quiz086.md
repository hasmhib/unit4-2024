# Quiz 086

<img width="max" alt="Screenshot 2024-10-02 at 1 30 06 PM" src="https://github.com/user-attachments/assets/f6546d90-7f8e-4a16-9186-f217aca32034">

## Code

### Class Queue
```.py
class Item:
    def __init__(self):
        self.data = None

    def get(self):
        return self.data

    def set(self, value):
        self.data = value

class Queue:
    def __init__(self):
        self.data = []

    def isEmpty(self):
        return len(self.data) == 0

    def enqueue(self, value):
        item = Item()
        item.set(value)
        self.data.append(item)

    def dequeue(self):
        output = self.data[0].get()
        self.data = self.data[1:]
        return output

```

### frequency_queue
```py
def frequency_queue(scores):
    min_score = 1
    max_score = 7
    frequency = [0] * (max_score - min_score + 1)

    for score in scores:
        if min_score <= score <= max_score:
            frequency[score - min_score] += 1

    out_queue = Queue()
    for freq in frequency:
        out_queue.enqueue(freq)

    return out_queue


S = [4, 5, 5, 6, 3]
print(frequency_queue(S))

S = [1, 2, 3, 4, 5, 6, 7]
print(frequency_queue(S))

S = [1, 1, 1, 1, 1, 1, 7]
print(frequency_queue(S))
```

## Proof of work
<img width="max" alt="Screenshot 2024-10-02 at 1 36 40 PM" src="https://github.com/user-attachments/assets/a776bd63-4f87-4127-8679-69f6acdd0eaa">
