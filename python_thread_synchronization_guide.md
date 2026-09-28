# Python Thread Synchronization Guide

This guide covers the core synchronization primitives provided by Python's built-in `threading` module to control execution order, coordinate tasks, and protect shared data.

---

## 1. Lock (`threading.Lock`)
**Best Use Case:** Protecting shared data from corruption (Race Conditions). 

```python
import threading

shared_counter = 0
counter_lock = threading.Lock()

def increment_counter():
    global shared_counter
    for _ in range(100000):
        # Safely modify the shared variable
        with counter_lock:
            shared_counter += 1

threads = [threading.Thread(target=increment_counter) for _ in range(3)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Final Counter value: {shared_counter}") # Always exactly 300000
```

---

## 2. Event (`threading.Event`)
**Best Use Case:** One-way signaling. Perfect for making one thread wait for an explicit trigger or initialization phase from another thread.

```python
import threading
import time

setup_complete = threading.Event()

def initialize_system():
    print("Loading configuration files...")
    time.sleep(2)
    print("System initialized!")
    setup_complete.set() # Unblocks any threads waiting on this event

def process_requests():
    print("Worker waiting for system setup...")
    setup_complete.wait() # Pauses here until set() is called
    print("Worker: Now processing user requests!")

threading.Thread(target=process_requests).start()
threading.Thread(target=initialize_system).start()
```

---

## 3. Condition (`threading.Condition`)
**Best Use Case:** State-based signaling, typically used in Producer-Consumer patterns where threads wait for specific data states or logic to become true.

```python
import threading
import time

queue = []
cv = threading.Condition()

def consumer():
    with cv:
        while not queue: # Wait until the list has items
            print("Consumer: Queue empty, sleeping...")
            cv.wait() 
        print(f"Consumer: Popped {queue.pop(0)} from queue!")

def producer():
    time.sleep(1.5)
    with cv:
        queue.append("Task Data")
        print("Producer: Added task to queue.")
        cv.notify() # Wake up the waiting consumer thread

threading.Thread(target=consumer).start()
threading.Thread(target=producer).start()
```

---

## 4. Semaphore (`threading.Semaphore`)
**Best Use Case:** Throttling or limiting access to a fixed resource capacity (e.g., maximum concurrent database or API connections).

```python
import threading
import time

# Allow maximum 2 threads to download simultaneously
download_limiter = threading.Semaphore(2)

def download_file(thread_id):
    with download_limiter:
        print(f"Thread {thread_id}: Starting download...")
        time.sleep(2) # Simulating heavy network download
        print(f"Thread {thread_id}: Download complete.")

# Launch 4 threads, but only 2 will run concurrently
for i in range(4):
    threading.Thread(target=download_file, args=(i,)).start()
```

---

## 5. Barrier (`threading.Barrier`)
**Best Use Case:** Synchronizing a fixed number of threads at a checkpoint before they are all allowed to proceed together.

```python
import threading
import time

# Wait for exactly 3 players to connect before starting the match
match_barrier = threading.Barrier(3)

def player_connect(player_name):
    print(f"{player_name} is logging in...")
    time.sleep(1) 
    print(f"{player_name} is ready. Waiting for others...")
    
    match_barrier.wait() # Blocks until 3 threads hit this point
    
    print(f"Match Started for {player_name}!")

players = ["Alice", "Bob", "Charlie"]
for p in players:
    threading.Thread(target=player_connect, args=(p,)).start()
```

---

## Summary Cheat Sheet

| Primitive | Primary Use Case | Real-world Analogy |
| :--- | :--- | :--- |
| **Lock** | Protect variables/resources | A single-occupancy bathroom door key. |
| **Event** | One thread triggers another | A race starter firing a starting pistol. |
| **Condition** | Notify based on complex data states | A restaurant pager that buzzes when your specific table is ready. |
| **Semaphore** | Limit maximum concurrent threads | A nightclub bouncer letting only 50 people inside at a time. |
| **Barrier** | Wait for all workers to catch up | A tour guide waiting for the entire group to assemble before moving. |