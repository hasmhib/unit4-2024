# Quiz 087

<img width="max" alt="Screenshot 2024-10-02 at 1 39 58 PM" src="https://github.com/user-attachments/assets/80cad32e-4796-4293-a26c-8fcc6fd11b93">

## Code

```py
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


x = Item()
x.set('bob')
print(x.get()) 

t = Queue()
t.enqueue('bob')
print(t.dequeue())  

```

## Proof of work
<img width="max" alt="Screenshot 2024-10-02 at 1 42 31 PM" src="https://github.com/user-attachments/assets/e59aab71-7a25-4d60-96b6-704f0e123326">
