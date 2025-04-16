# 11. Concurrency and Parallelism ⚡

[<- Back: Advanced Python Concepts](./10-advanced-concepts.md) | [Next: Testing and Documentation ->](./12-testing.md)

## Table of Contents

- [Understanding Concurrency Concepts](#understanding-concurrency-concepts)
- [Threading](#threading)
- [Multiprocessing](#multiprocessing)
- [Asynchronous Programming](#asynchronous-programming)
- [Choosing the Right Approach](#choosing-the-right-approach)
- [Real-world Applications](#real-world-applications)

## Understanding Concurrency Concepts

### Key Terminology

- **Concurrency**: Dealing with multiple things at once (structure)
- **Parallelism**: Doing multiple things at once (execution)
- **Synchronous**: Operations that block until completed
- **Asynchronous**: Operations that don't block the calling thread
- **I/O-bound**: Programs that spend time waiting for external resources (network, disk)
- **CPU-bound**: Programs that spend time performing calculations

### Python's Limitations: The Global Interpreter Lock (GIL)

```python
# The GIL prevents multiple native threads from executing Python bytecode simultaneously
# This means that threads can't achieve true parallelism on CPU-bound tasks
# However, the GIL is released during I/O operations, making threading useful for I/O-bound tasks
# For CPU-bound tasks, multiprocessing is recommended (uses separate processes with their own GIL)
```

### Concurrency vs Parallelism Visualization

```
Concurrency (managing multiple tasks):                 Parallelism (executing tasks simultaneously):
  
Single-core concurrency:                               Multi-core parallelism:
 ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                       Core 1: ┌──────────┐ ┌──────────┐
 │Task1│ │Task2│ │Task1│ │Task2│                               │  Task 1   │ │  Task 3   │
 └─────┘ └─────┘ └─────┘ └─────┘                               └──────────┘ └──────────┘
   Time →                                              Core 2: ┌──────────┐ ┌──────────┐
                                                               │  Task 2   │ │  Task 4   │
                                                               └──────────┘ └──────────┘
                                                         Time →
```

## Threading

Python's `threading` module enables concurrent execution via threads. Ideal for I/O-bound tasks.

### Basic Threading

```python
import threading
import time

def task(name, delay):
    print(f"{name} started")
    time.sleep(delay)  # Simulate I/O operation
    print(f"{name} completed")

# Create threads
thread1 = threading.Thread(target=task, args=("Thread-1", 2))
thread2 = threading.Thread(target=task, args=("Thread-2", 1))

# Start threads
thread1.start()
thread2.start()

# Wait for threads to complete
thread1.join()
thread2.join()

print("All threads completed")
```

### Thread Pooling with concurrent.futures

```python
import concurrent.futures
import requests
import time

def fetch_url(url):
    """Fetch a URL and return the response text"""
    start = time.time()
    response = requests.get(url)
    data = response.text
    elapsed = time.time() - start
    print(f"{url} fetched in {elapsed:.2f}s")
    return data

urls = [
    "https://www.python.org",
    "https://www.github.com",
    "https://www.wikipedia.org",
    "https://www.stackoverflow.com",
    "https://www.reddit.com"
]

# Using ThreadPoolExecutor to fetch URLs concurrently
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    # Submit all tasks and get Future objects
    future_to_url = {executor.submit(fetch_url, url): url for url in urls}
    
    # Process results as they complete
    for future in concurrent.futures.as_completed(future_to_url):
        url = future_to_url[future]
        try:
            data = future.result()
            print(f"{url} size: {len(data)}")
        except Exception as e:
            print(f"{url} generated an exception: {e}")
```

### Thread Synchronization

```python
import threading
import time
import random

class BankAccount:
    def __init__(self, balance):
        self.balance = balance
        self.lock = threading.Lock()
    
    def withdraw(self, amount):
        with self.lock:  # Thread-safe operation using a lock
            if self.balance >= amount:
                time.sleep(0.1)  # Simulate processing time
                self.balance -= amount
                return True
            return False
    
    def deposit(self, amount):
        with self.lock:  # Thread-safe operation using a lock
            time.sleep(0.1)  # Simulate processing time
            self.balance += amount
            return True

# Shared account
account = BankAccount(1000)

def make_withdrawals():
    for _ in range(5):
        amount = random.randint(100, 300)
        if account.withdraw(amount):
            print(f"Withdrew ${amount}, balance: ${account.balance}")
        else:
            print(f"Failed to withdraw ${amount}, balance: ${account.balance}")
        time.sleep(random.random())

def make_deposits():
    for _ in range(5):
        amount = random.randint(100, 300)
        account.deposit(amount)
        print(f"Deposited ${amount}, balance: ${account.balance}")
        time.sleep(random.random())

# Create and start threads
threads = [
    threading.Thread(target=make_withdrawals),
    threading.Thread(target=make_deposits)
]

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()

print(f"Final balance: ${account.balance}")
```

### Thread Communication with Queue

```python
import threading
import queue
import time
import random

# Create a thread-safe queue
task_queue = queue.Queue()
result_queue = queue.Queue()

def producer():
    """Generate work items and put them in the queue"""
    for i in range(10):
        item = f"Task {i}"
        task_queue.put(item)
        print(f"Produced: {item}")
        time.sleep(random.random())
    # Signal no more items
    task_queue.put(None)

def consumer():
    """Process items from the queue"""
    while True:
        item = task_queue.get()
        if item is None:
            # Signal to next consumer that no more items are coming
            task_queue.put(None)
            break
            
        # Process the item
        print(f"Processing: {item}")
        time.sleep(random.random() * 2)
        result = f"Result for {item}"
        result_queue.put(result)
        
        # Mark the task as done
        task_queue.task_done()

# Create and start threads
producer_thread = threading.Thread(target=producer)
consumer_threads = [threading.Thread(target=consumer) for _ in range(2)]

producer_thread.start()
for t in consumer_threads:
    t.start()

# Wait for all threads to complete
producer_thread.join()
for t in consumer_threads:
    t.join()

# Print results
print("\nResults:")
while not result_queue.empty():
    print(result_queue.get())
```

## Multiprocessing

Python's `multiprocessing` module enables true parallelism by using separate processes. Ideal for CPU-bound tasks.

### Basic Multiprocessing

```python
import multiprocessing
import time

def cpu_intensive_task(number):
    """A CPU-intensive task that calculates the sum of squares"""
    result = 0
    for i in range(number):
        result += i * i
    return result

if __name__ == "__main__":  # Required for Windows compatibility
    numbers = [10000000, 20000000, 30000000, 40000000]
    
    # Sequential execution
    start_time = time.time()
    results = [cpu_intensive_task(num) for num in numbers]
    sequential_time = time.time() - start_time
    print(f"Sequential execution time: {sequential_time:.2f} seconds")
    
    # Parallel execution
    start_time = time.time()
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(cpu_intensive_task, numbers)
    parallel_time = time.time() - start_time
    print(f"Parallel execution time: {parallel_time:.2f} seconds")
    print(f"Speedup: {sequential_time/parallel_time:.2f}x")
```

### Process Pool with concurrent.futures

```python
import concurrent.futures
import math
import time

def calculate_prime_factors(n):
    """Find all prime factors of a number"""
    factors = []
    d = 2
    while n > 1:
        while n % d == 0:
            factors.append(d)
            n //= d
        d += 1
        if d*d > n:
            if n > 1:
                factors.append(n)
            break
    return factors

# Large numbers to factorize
numbers = [
    112272535095293,
    112582705942171,
    112272535095293,
    115280095190773,
    115797848077099,
    1099726899285419
]

# Sequential processing
start = time.time()
results = [calculate_prime_factors(n) for n in numbers]
end = time.time()
print(f"Sequential execution time: {end - start:.2f} seconds")

# Parallel processing
start = time.time()
with concurrent.futures.ProcessPoolExecutor() as executor:
    results = list(executor.map(calculate_prime_factors, numbers))
end = time.time()
print(f"Parallel execution time: {end - start:.2f} seconds")
```

### Shared Memory between Processes

```python
import multiprocessing
import time

def update_counter(counter, lock):
    for _ in range(1000):
        with lock:
            counter.value += 1

if __name__ == "__main__":
    # Create a shared counter and lock
    counter = multiprocessing.Value('i', 0)  # 'i' stands for integer
    lock = multiprocessing.Lock()
    
    # Create and start processes
    processes = []
    for _ in range(4):
        p = multiprocessing.Process(target=update_counter, args=(counter, lock))
        processes.append(p)
        p.start()
    
    # Wait for all processes to finish
    for p in processes:
        p.join()
    
    print(f"Final counter value: {counter.value}")
```

### Inter-Process Communication with Pipes and Queues

```python
import multiprocessing
import time
import random

def producer(queue):
    """Produce items and put them in the queue"""
    for i in range(10):
        item = f"Item {i}"
        queue.put(item)
        print(f"Producer: {item}")
        time.sleep(random.random())
    # Signal no more items
    queue.put(None)

def consumer(queue):
    """Consume items from the queue"""
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consumer: {item}")
        time.sleep(random.random() * 2)

if __name__ == "__main__":
    # Create a multiprocessing queue
    queue = multiprocessing.Queue()
    
    # Create producer and consumer processes
    producer_process = multiprocessing.Process(target=producer, args=(queue,))
    consumer_process = multiprocessing.Process(target=consumer, args=(queue,))
    
    # Start processes
    producer_process.start()
    consumer_process.start()
    
    # Wait for processes to finish
    producer_process.join()
    consumer_process.join()
    
    print("All done!")
```

## Asynchronous Programming

Python's `asyncio` library provides tools for writing concurrent code using the `async`/`await` syntax. Ideal for I/O-bound tasks.

### Basic asyncio

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)  # Non-blocking sleep
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")
    
    # These will run concurrently
    await asyncio.gather(
        say_after(1, 'hello'),
        say_after(2, 'world')
    )
    
    print(f"finished at {time.strftime('%X')}")

# Run the asyncio program
asyncio.run(main())
```

### Asynchronous Web Requests

```python
import asyncio
import aiohttp  # External library for asynchronous HTTP requests
import time

async def fetch(session, url):
    """Fetch a URL asynchronously"""
    start = time.time()
    async with session.get(url) as response:
        data = await response.text()
        elapsed = time.time() - start
        print(f"{url} fetched in {elapsed:.2f}s")
        return data

async def main():
    urls = [
        "https://www.python.org",
        "https://www.github.com",
        "https://www.wikipedia.org",
        "https://www.stackoverflow.com",
        "https://www.reddit.com"
    ]
    
    start_time = time.time()
    
    # Create a session for all requests
    async with aiohttp.ClientSession() as session:
        # Create tasks for all URLs
        tasks = [fetch(session, url) for url in urls]
        
        # Wait for all tasks to complete
        results = await asyncio.gather(*tasks)
        
        # Print results
        for url, data in zip(urls, results):
            print(f"{url} size: {len(data)}")
    
    print(f"Total time: {time.time() - start_time:.2f}s")

# Run the asyncio program
asyncio.run(main())
```

### Asynchronous Queues

```python
import asyncio
import random

async def producer(queue):
    """Produce items and put them in the queue"""
    for i in range(10):
        item = f"Item {i}"
        await queue.put(item)
        print(f"Produced: {item}")
        await asyncio.sleep(random.random())
    
    # Signal no more items
    await queue.put(None)

async def consumer(queue, name):
    """Consume items from the queue"""
    while True:
        item = await queue.get()
        if item is None:
            # Signal to other consumers that no more items are coming
            await queue.put(None)
            break
        
        print(f"Consumer {name} got: {item}")
        await asyncio.sleep(random.random() * 2)
        queue.task_done()

async def main():
    # Create an asyncio queue
    queue = asyncio.Queue()
    
    # Create tasks
    producer_task = asyncio.create_task(producer(queue))
    consumer_tasks = [
        asyncio.create_task(consumer(queue, i))
        for i in range(2)
    ]
    
    # Wait for producer to finish
    await producer_task
    
    # Wait for all consumers to finish
    await asyncio.gather(*consumer_tasks)
    
    print("All done!")

# Run the asyncio program
asyncio.run(main())
```

### Combining asyncio with Threading or Multiprocessing

```python
import asyncio
import concurrent.futures
import time

def cpu_bound_task(n):
    """A CPU-bound task that calculates the sum of squares"""
    result = 0
    for i in range(n):
        result += i * i
    return result

async def main():
    # Create a list of numbers to process
    numbers = [10000000, 20000000, 30000000, 40000000]
    
    print("Running CPU-bound tasks...")
    
    # Using ProcessPoolExecutor for CPU-bound tasks
    with concurrent.futures.ProcessPoolExecutor() as executor:
        # Create a list of futures
        loop = asyncio.get_running_loop()
        futures = [
            loop.run_in_executor(executor, cpu_bound_task, num)
            for num in numbers
        ]
        
        # Wait for all tasks to complete
        results = await asyncio.gather(*futures)
        
        for num, result in zip(numbers, results):
            print(f"Result for {num}: {result}")
    
    print("All CPU-bound tasks completed")

# Run the asyncio program
asyncio.run(main())
```

## Choosing the Right Approach

### Decision Tree for Concurrency

```
Question: What kind of task are you performing?
│
├── I/O-bound tasks (waiting for external resources)
│   │
│   ├── Simple use case → Use concurrent.futures.ThreadPoolExecutor
│   │
│   ├── Complex use case with many connections → Use asyncio
│   │
│   └── Need to modify shared state → Use threading with locks
│
└── CPU-bound tasks (heavy computations)
    │
    ├── Simple use case → Use concurrent.futures.ProcessPoolExecutor
    │
    └── Complex use case → Use multiprocessing directly
```

### Performance Comparison

```python
import time
import threading
import multiprocessing
import asyncio
import concurrent.futures
import requests
import aiohttp

# Task definitions (I/O-bound and CPU-bound)
def io_bound_task(url):
    """Fetch a URL (I/O-bound)"""
    response = requests.get(url)
    return len(response.content)

async def async_io_bound_task(url, session):
    """Fetch a URL asynchronously (I/O-bound)"""
    async with session.get(url) as response:
        content = await response.read()
        return len(content)

def cpu_bound_task(n):
    """Calculate the sum of squares (CPU-bound)"""
    return sum(i * i for i in range(n))

# Test functions
def test_sequential(urls, numbers):
    """Run tasks sequentially"""
    start = time.time()
    
    # I/O-bound tasks
    io_results = [io_bound_task(url) for url in urls]
    
    # CPU-bound tasks
    cpu_results = [cpu_bound_task(n) for n in numbers]
    
    elapsed = time.time() - start
    return elapsed, io_results, cpu_results

def test_threading(urls, numbers):
    """Run tasks using threading"""
    start = time.time()
    
    with concurrent.futures.ThreadPoolExecutor() as executor:
        io_results = list(executor.map(io_bound_task, urls))
        cpu_results = list(executor.map(cpu_bound_task, numbers))
    
    elapsed = time.time() - start
    return elapsed, io_results, cpu_results

def test_multiprocessing(urls, numbers):
    """Run tasks using multiprocessing"""
    start = time.time()
    
    with concurrent.futures.ProcessPoolExecutor() as executor:
        io_results = list(executor.map(io_bound_task, urls))
        cpu_results = list(executor.map(cpu_bound_task, numbers))
    
    elapsed = time.time() - start
    return elapsed, io_results, cpu_results

async def test_asyncio(urls, numbers):
    """Run tasks using asyncio"""
    start = time.time()
    
    # I/O-bound tasks
    async with aiohttp.ClientSession() as session:
        io_tasks = [async_io_bound_task(url, session) for url in urls]
        io_results = await asyncio.gather(*io_tasks)
    
    # CPU-bound tasks (still sequential in asyncio)
    cpu_results = [cpu_bound_task(n) for n in numbers]
    
    elapsed = time.time() - start
    return elapsed, io_results, cpu_results

async def test_asyncio_with_process_pool(urls, numbers):
    """Run tasks using asyncio with a process pool for CPU-bound tasks"""
    start = time.time()
    
    # I/O-bound tasks
    async with aiohttp.ClientSession() as session:
        io_tasks = [async_io_bound_task(url, session) for url in urls]
        io_results = await asyncio.gather(*io_tasks)
    
    # CPU-bound tasks with process pool
    loop = asyncio.get_running_loop()
    with concurrent.futures.ProcessPoolExecutor() as executor:
        cpu_tasks = [
            loop.run_in_executor(executor, cpu_bound_task, n)
            for n in numbers
        ]
        cpu_results = await asyncio.gather(*cpu_tasks)
    
    elapsed = time.time() - start
    return elapsed, io_results, cpu_results
```

## Real-world Applications

### Web Scraping with asyncio

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import time
import logging

# Setup logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

async def fetch_page(session, url):
    """Fetch a web page asynchronously"""
    try:
        async with session.get(url, timeout=10) as response:
            if response.status == 200:
                logger.info(f"Fetched {url}")
                return await response.text()
            logger.warning(f"Error fetching {url}: HTTP {response.status}")
            return None
    except Exception as e:
        logger.error(f"Exception fetching {url}: {str(e)}")
        return None

async def extract_links(session, url, base_url):
    """Extract all links from a page"""
    html = await fetch_page(session, url)
    if not html:
        return []
    
    soup = BeautifulSoup(html, 'html.parser')
    links = []
    
    for a_tag in soup.find_all('a', href=True):
        href = a_tag['href']
        if href.startswith('/'):
            # Convert relative URL to absolute
            href = base_url + href
        if href.startswith(base_url):
            links.append(href)
    
    return links

async def crawl(start_url, max_pages=10):
    """Crawl a website asynchronously"""
    base_url = '/'.join(start_url.split('/')[:3])  # Extract protocol + domain
    visited = set()
    to_visit = {start_url}
    results = {}
    
    async with aiohttp.ClientSession() as session:
        while to_visit and len(visited) < max_pages:
            # Get up to 5 URLs to fetch concurrently
            current_batch = list(to_visit)[:5]
            to_visit = to_visit - set(current_batch)
            
            # Fetch all pages in the current batch
            tasks = [fetch_page(session, url) for url in current_batch]
            pages = await asyncio.gather(*tasks)
            
            # Process the results
            for url, html in zip(current_batch, pages):
                visited.add(url)
                if html:
                    results[url] = len(html)
                    
                    # Find new links to visit
                    if len(visited) < max_pages:
                        new_links = await extract_links(session, url, base_url)
                        to_visit.update(set(new_links) - visited)
            
            logger.info(f"Visited {len(visited)} pages, {len(to_visit)} remaining")
    
    return results

async def main():
    start_time = time.time()
    results = await crawl("https://www.python.org", max_pages=20)
    elapsed = time.time() - start_time
    
    logger.info(f"Crawled {len(results)} pages in {elapsed:.2f} seconds")
    
    # Print summary of results
    for url, size in sorted(results.items(), key=lambda x: x[1], reverse=True)[:5]:
        logger.info(f"{url}: {size} bytes")

# Run the crawler
if __name__ == "__main__":
    asyncio.run(main())
```

### Parallel Data Processing

```python
import multiprocessing
import pandas as pd
import numpy as np
import time
from functools import partial

def process_chunk(df_chunk, complex_operation=None):
    """Process a chunk of a DataFrame"""
    if complex_operation:
        return complex_operation(df_chunk)
    
    # Default processing: calculate statistics
    result = {
        'count': len(df_chunk),
        'mean': df_chunk.mean().to_dict(),
        'std': df_chunk.std().to_dict(),
        'min': df_chunk.min().to_dict(),
        'max': df_chunk.max().to_dict()
    }
    return result

def parallel_process_dataframe(df, operation=None, num_processes=None):
    """Process a DataFrame in parallel using multiple processes"""
    if num_processes is None:
        num_processes = multiprocessing.cpu_count()
    
    # Split DataFrame into chunks
    chunk_size = len(df) // num_processes
    chunks = [df.iloc[i:i+chunk_size] for i in range(0, len(df), chunk_size)]
    
    # Process chunks in parallel
    with multiprocessing.Pool(processes=num_processes) as pool:
        process_func = partial(process_chunk, complex_operation=operation)
        results = pool.map(process_func, chunks)
    
    return results

# Example usage
if __name__ == "__main__":
    # Create a sample DataFrame
    size = 1000000
    df = pd.DataFrame({
        'A': np.random.randn(size),
        'B': np.random.randn(size),
        'C': np.random.choice(['X', 'Y', 'Z'], size),
        'D': np.random.randint(0, 100, size)
    })
    
    # Define a complex operation
    def complex_operation(chunk):
        # Group by category and calculate statistics
        grouped = chunk.groupby('C').agg({
            'A': ['mean', 'std', 'min', 'max'],
            'B': ['mean', 'std', 'min', 'max'],
            'D': ['mean', 'std', 'min', 'max', 'sum']
        })
        
        # Additional calculations
        chunk['A_squared'] = chunk['A'] ** 2
        chunk['A_B_product'] = chunk['A'] * chunk['B']
        
        return {
            'grouped_stats': grouped.to_dict(),
            'corr': chunk.corr().to_dict(),
            'quantiles': chunk.quantile([0.25, 0.5, 0.75]).to_dict()
        }
    
    # Sequential processing
    start = time.time()
    sequential_result = process_chunk(df, complex_operation)
    sequential_time = time.time() - start
    print(f"Sequential processing time: {sequential_time:.2f} seconds")
    
    # Parallel processing
    start = time.time()
    parallel_results = parallel_process_dataframe(df, complex_operation)
    parallel_time = time.time() - start
    print(f"Parallel processing time: {parallel_time:.2f} seconds")
    print(f"Speedup: {sequential_time/parallel_time:.2f}x")
```

---

[<- Back: Advanced Python Concepts](./10-advanced-concepts.md) | [Next: Testing and Documentation ->](./12-testing.md)
