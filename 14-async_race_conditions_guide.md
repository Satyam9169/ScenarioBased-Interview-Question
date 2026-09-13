# Handling Asynchronous API Calls & Race Conditions in React
### A Senior Frontend Engineer's Guide to Robust Data Fetching & Architecture

Understanding asynchronous behavior, preventing race conditions, and mastering production-grade data fetching strategies for senior frontend interviews.

---

## 1. Asynchronous Behavior vs. Race Condition

API A being called at 1 second and API B at 3 seconds does not guarantee API A's response will arrive first.
* Network latency, server processing time, database queries, and network congestion determine response order.

### Scenario
* **Time 1s**: API A called
* **Time 3s**: API B called
* **Time 6s**: API B response received
* **Time 8s**: API A response received

The actual execution timeline:
```text
API A: [Request: 1s] -------------------------> [Response: 8s]
API B:        [Request: 3s] ------------> [Response: 6s]
```

This is natural asynchronous behavior, not necessarily a race condition.

### What is a Race Condition?
A race condition occurs when multiple asynchronous operations update **shared state**, and the final result depends unpredictably on which operation finishes first.

```javascript
const [data, setData] = useState([]);

useEffect(() => {
  fetch('/api/A')
    .then(res => res.json())
    .then(data => setData(data));

  fetch('/api/B')
    .then(res => res.json())
    .then(data => setData(data));
}, []);
```

* **If API B responds first**: API B Response (`setData(B)`) $\rightarrow$ API A Response (`setData(A)`). Final UI contains **A's data**.
* **If API A responds first**: API A Response (`setData(A)`) $\rightarrow$ API B Response (`setData(B)`). Final UI contains **B's data**.

The result is unpredictable based on response timing. That is a race condition.

---

## 2. Solutions for Senior Developers

The solution depends entirely on the business requirements and data dependencies.

---

### Approach 1: If APIs are Independent & Both Required $\rightarrow$ `Promise.all()`
If API A and API B are independent and both responses are required before rendering:

```javascript
const fetchData = async () => {
  try {
    const [responseA, responseB] = await Promise.all([
      fetch('/api/A'),
      fetch('/api/B')
    ]);
    const dataA = await responseA.json();
    const dataB = await responseB.json();
    console.log(dataA, dataB);
  } catch (error) {
    console.error(error);
  }
};
```

* **Why?** `Promise.all()` waits for all promises to resolve. If API A takes 8s and API B takes 6s, `Promise.all()` completes at 8s. The order of completion does not matter.
* **Interview Answer:** *"If APIs are independent and I need both responses before updating the UI, I use `Promise.all()`. It resolves only when all promises complete, preventing inconsistent intermediate state."*

---

### Approach 2: If API B Depends on API A $\rightarrow$ Sequential Execution
Suppose API B requires data from API A (e.g., API A fetches User ID, API B fetches User Orders using that User ID). API B must not execute before A completes.

```javascript
const fetchData = async () => {
  try {
    const responseA = await fetch('/api/A');
    const dataA = await responseA.json();

    const responseB = await fetch(`/api/B/${dataA.id}`);
    const dataB = await responseB.json();
    console.log(dataA, dataB);
  } catch (error) {
    console.error(error);
  }
};
```

* **Execution Flow:** API A Called $\rightarrow$ Response A $\rightarrow$ API B Called $\rightarrow$ Response B. This guarantees dependency order.
* **Important:** Do not use sequential execution unnecessarily. If APIs are independent, sequential calls increase total loading time.

---

### Approach 3: Independent APIs with Separate State Variables
This is the most common real-world scenario. Instead of storing both responses in one shared state variable (`setData(...)`), decouple them into distinct state variables.

```javascript
const [dataA, setDataA] = useState(null);
const [dataB, setDataB] = useState(null);

useEffect(() => {
  const fetchA = async () => {
    const response = await fetch('/api/A');
    const data = await response.json();
    setDataA(data);
  };

  const fetchB = async () => {
    const response = await fetch('/api/B');
    const data = await response.json();
    setDataB(data);
  };

  fetchA();
  fetchB();
}, []);
```

* **Result:** Response order doesn't matter. API B response updates `setDataB()` and API A response updates `setDataA()` completely independently.
* **Senior Developer Principle:** *Avoid sharing mutable state between independent asynchronous operations.*

---

### Approach 4: Using `Promise.allSettled()` for Error Resilience
Sometimes one API may fail, but you still want the remaining successful responses rendered (e.g., independent dashboard widgets).

```javascript
const fetchData = async () => {
  const results = await Promise.allSettled([
    fetch('/api/A'),
    fetch('/api/B')
  ]);

  const [resultA, resultB] = results;

  if (resultA.status === 'fulfilled') {
    console.log('API A Success');
  }
  if (resultB.status === 'fulfilled') {
    console.log('API B Success');
  }
};
```

| Method | Behavior |
| :--- | :--- |
| `Promise.all()` | Fails immediately if any single promise rejects (*fail-fast*). |
| `Promise.allSettled()` | Waits for all promises to settle regardless of fulfillment or rejection. |
| `Promise.race()` | Settles as soon as the first promise settles (resolves or rejects). |

---

### Approach 5: Rapid User Input / Search Changes $\rightarrow$ `AbortController`
When a user types quickly (`R` $\rightarrow$ `Re` $\rightarrow$ `React`), earlier search requests may finish slower than newer ones, overwriting fresh data with stale results.

```javascript
useEffect(() => {
  const controller = new AbortController();

  const fetchData = async () => {
    try {
      const response = await fetch(`/api/search?q=${search}`, {
        signal: controller.signal
      });
      const data = await response.json();
      setResults(data);
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error(error);
      }
    }
  };

  fetchData();

  return () => {
    controller.abort();
  };
}, [search]);
```

* **What Happens?** Typing `R` starts a request. Typing `Re` aborts the `R` request. Typing `React` aborts the `Re` request. Only the latest active request is permitted to update the component state.

---

### Approach 6: Request ID / "Latest Request Wins" Strategy
When cancellation is not possible (e.g., non-abortable third-party client SDKs or WebSockets), use a monotonic request counter via `useRef`.

```javascript
const requestIdRef = useRef(0);

const fetchData = async () => {
  const requestId = ++requestIdRef.current;
  const response = await fetch('/api/data');
  const data = await response.json();

  if (requestId === requestIdRef.current) {
    setData(data);
  }
};
```

* **How it works:** Request 1 starts ($ID = 1$). Request 2 starts ($ID = 2$). If Request 2 finishes first, $ID\ (2) === Latest\ ID\ (2)$, so state updates. When Request 1 finishes later, $ID\ (1) \neq Latest\ ID\ (2)$, so the stale response is safely discarded.

---

### Approach 7: Modern Data Fetching via React Query / RTK Query
In production applications, manual synchronization across complex components is avoided by using dedicated server-state caching libraries.

```javascript
const { data: dataA } = useQuery({
  queryKey: ['apiA'],
  queryFn: fetchApiA
});

const { data: dataB } = useQuery({
  queryKey: ['apiB'],
  queryFn: fetchApiB
});
```

* **Benefits:** Automatic caching, deduplication, loading/error states, window focus refetching, query cancellation, and stale-data management significantly reduce race conditions out of the box.

---

## 3. Real Senior-Level Decision Matrix

| Scenario | Best Solution |
| :--- | :--- |
| APIs independent, both required | `Promise.all()` |
| APIs independent, partial success acceptable | `Promise.allSettled()` |
| API B depends on API A | Sequential `await` |
| Independent UI components / data | Separate state variables |
| Rapid search / input changes | `AbortController` |
| Old request must never overwrite latest (non-abortable) | Request ID / "Latest Wins" Pattern |
| Complex production data fetching | React Query / RTK Query |

---

## 4. Best Interview Answer (Professional Version)

> *"The order in which APIs are invoked doesn't guarantee the order in which responses arrive because network latency and server processing time are unpredictable.*
>
> *I first identify whether the APIs are independent or dependent. If they are independent and both responses are required, I use `Promise.all()` to synchronize them. If partial failure is acceptable, I prefer `Promise.allSettled()`.*
>
> *If API B depends on API A, I execute them sequentially using `async`/`await`. For scenarios like autocomplete or search where multiple requests can be triggered rapidly, I use `AbortController` to cancel stale requests or implement a request ID mechanism so that only the latest response updates the UI.*
>
> *Additionally, I avoid shared mutable state between independent API calls and prefer React Query or RTK Query for managing complex asynchronous data fetching in production applications."*

### Crucial Follow-up Question

**Interviewer:** *"Does `Promise.all()` solve race conditions?"*

**Correct Answer:** *"Not always. `Promise.all()` only waits for multiple promises to complete within a single batch. It does **not** automatically prevent race conditions when multiple batches are triggered repeatedly over time (e.g., rapid user searches or re-renders). For that, we still need `AbortController` or Request ID tracking."*
