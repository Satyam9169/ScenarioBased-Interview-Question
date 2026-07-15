📦 React-Scenario-Based-Interview-Questions
│
├── README.md
├── 01-Infinite-Scrolling.md               
├── 02-Auto-Logout-After-Inactivity.md
├── 03-Internationalization-i18n.md
├── 04-Large-File-Upload.md
├── 05-React-SEO-Google-Indexing.md
├── 06-Stale-Closures-useEffect.md
├── 07-Bundle-Optimization.md
├── 08-Role-Based-Routing.md
├── 09-Page-Transitions-Animations.md
├── 10-API-Throttling-Debouncing.md
├── 11-Handling-100K-Users.md
├── 12-LinkedIn-Infinite-Feed.md
├── 13-Solved-compelx-issue-.md
└── assets/


# 🚀 Scenario 1: Building an Infinite Scrolling Component for Large API Datasets (React)

## 📌 Interview Question

**How would you build an infinite scrolling component for large API datasets in React?**

---

# 📖 Overview

When building an infinite scrolling component in React, the objective is to **load data page by page** as the user scrolls, instead of downloading the entire dataset at once.

This technique improves:

- 🚀 Performance
- ⚡ Initial page load time
- 💾 Memory usage
- 😊 User experience

Infinite scrolling is commonly used in applications like:

- LinkedIn Feed
- Instagram
- Facebook
- Twitter/X
- YouTube
- Amazon Product Listings

---

# 🎯 Approach

A typical implementation follows these steps:

1. Fetch the first page of data.
2. Display the fetched items.
3. Observe the last rendered item.
4. When the last item enters the viewport, fetch the next page.
5. Append the new data to the existing list.
6. Repeat until there is no more data.

---

# 📁 Folder Structure

```text
src/
│── App.jsx
│── InfiniteScroll.jsx
│── api.js
```

---

# 🌐 API Layer

## api.js

```javascript
import axios from "axios";

const API = "https://jsonplaceholder.typicode.com/posts";

export const fetchPosts = async (page, limit) => {
  const response = await axios.get(API, {
    params: {
      _page: page,
      _limit: limit,
    },
  });

  return response.data;
};
```

---

# ⚛️ Infinite Scroll Component

## InfiniteScroll.jsx

```javascript
import { useEffect, useState, useRef, useCallback } from "react";
import { fetchPosts } from "./api";

const LIMIT = 10;

function InfiniteScroll() {
  const [posts, setPosts] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const observer = useRef();

  useEffect(() => {
    loadPosts();
  }, [page]);

  const loadPosts = async () => {
    if (loading) return;

    setLoading(true);

    const data = await fetchPosts(page, LIMIT);

    if (data.length === 0) {
      setHasMore(false);
    } else {
      setPosts((prev) => [...prev, ...data]);
    }

    setLoading(false);
  };

  const lastElementRef = useCallback(
    (node) => {
      if (loading) return;

      if (observer.current) observer.current.disconnect();

      observer.current = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting && hasMore) {
          setPage((prev) => prev + 1);
        }
      });

      if (node) observer.current.observe(node);
    },
    [loading, hasMore]
  );

  return (
    <div style={{ width: "600px", margin: "20px auto" }}>
      <h2>Infinite Scroll</h2>

      {posts.map((post, index) => {
        if (posts.length === index + 1) {
          return (
            <div
              ref={lastElementRef}
              key={post.id}
              style={{
                border: "1px solid gray",
                margin: "10px",
                padding: "15px",
              }}
            >
              <h3>{post.title}</h3>
              <p>{post.body}</p>
            </div>
          );
        }

        return (
          <div
            key={post.id}
            style={{
              border: "1px solid gray",
              margin: "10px",
              padding: "15px",
            }}
          >
            <h3>{post.title}</h3>
            <p>{post.body}</p>
          </div>
        );
      })}

      {loading && <h3>Loading...</h3>}

      {!hasMore && <h3>No More Data</h3>}
    </div>
  );
}

export default InfiniteScroll;
```

---

# 🏠 App Component

## App.jsx

```javascript
import InfiniteScroll from "./InfiniteScroll";

function App() {
  return <InfiniteScroll />;
}

export default App;
```

---

# 🔄 Application Flow

```text
Page Loads
     │
     ▼
Fetch Page 1
     │
     ▼
Render Posts
     │
     ▼
Observe Last Item
     │
     ▼
Last Item Visible?
     │
    Yes
     │
     ▼
IntersectionObserver Fires
     │
     ▼
Increase Page Number
     │
     ▼
Fetch Next Page
     │
     ▼
Append New Data
     │
     ▼
Repeat Until No More Data
```

---

# 🧠 Understanding the State

| State | Purpose |
|--------|----------|
| posts | Stores all fetched posts |
| page | Current page number |
| loading | Prevents duplicate API calls |
| hasMore | Indicates whether more data is available |
| observer | Stores the IntersectionObserver instance |

---

# 🔍 Why Use IntersectionObserver Instead of Scroll Events?

| Scroll Event | IntersectionObserver |
|--------------|----------------------|
| Fires continuously while scrolling | Fires only when the observed element enters the viewport |
| Requires manual scroll position calculations | Browser handles visibility detection |
| Higher CPU usage | More efficient |
| Can cause performance issues | Optimized by the browser |
| Needs throttling/debouncing | No manual optimization required |

---

# 🚀 Best Practices for Large Datasets

## ✅ 1. Use Pagination

Never fetch all records at once.

```http
GET /posts?page=2&limit=20
```

---

## ✅ 2. Prevent Duplicate Requests

Use a loading flag.

```javascript
if (loading) return;
```

---

## ✅ 3. Stop When No More Data Exists

```javascript
if (data.length === 0) {
    setHasMore(false);
}
```

---

## ✅ 4. Handle Errors

```javascript
try {
   // API Call
} catch (error) {
   console.log(error);
}
```

Show an error message and optionally provide a retry option.

---

## ✅ 5. Virtualize Large Lists

Rendering thousands of DOM nodes slows the browser.

Use:

- react-window
- react-virtualized

Only visible items are rendered.

---

## ✅ 6. Cache Loaded Data

Avoid unnecessary API requests using:

- React Query (TanStack Query)
- SWR
- Redux Toolkit Query

Benefits:

- Faster navigation
- Reduced server load
- Better user experience

---

## ✅ 7. Clean Up the Observer

Always disconnect old observers.

```javascript
observer.current.disconnect();
```

This prevents memory leaks.

---

## ✅ 8. Show Loading State

```jsx
{loading && <h3>Loading...</h3>}
```

---

## ✅ 9. Display End-of-Data Message

```jsx
{!hasMore && <h3>No More Data</h3>}
```

---

# ⚡ Performance Optimizations

- Use pagination
- Lazy load images (`loading="lazy"`)
- Use `React.memo()` for list items
- Use `useCallback()` for observer callbacks
- Use virtualization (`react-window`)
- Cache API responses
- Minimize unnecessary re-renders
- Abort stale API requests with `AbortController`

---

# ❌ Common Mistakes

- Fetching the entire dataset at once
- Triggering multiple API requests simultaneously
- Forgetting the loading state
- Replacing existing data instead of appending
- Not handling the end of available data
- Not disconnecting the observer
- Rendering thousands of DOM nodes without virtualization

---

# 🎤 Interview Explanation (1-Minute Answer)

> For large API datasets, I implement infinite scrolling using the browser's **IntersectionObserver API**. I initially fetch the first page of data and render it. I attach an observer to the last rendered item, and when it enters the viewport, I request the next page from the server and append the results to the existing list. I maintain **loading** and **hasMore** state to prevent duplicate requests and stop fetching when all data has been loaded. For very large datasets, I combine infinite scrolling with virtualization using **react-window** to render only the visible items, keeping memory usage low and ensuring smooth scrolling.

---

# ❓ Frequently Asked Interview Questions

### Q1. Why is IntersectionObserver preferred over the scroll event?

**Answer:**

- Better performance
- Lower CPU usage
- Browser-optimized
- No manual scroll calculations
- Prevents excessive event firing

---

### Q2. How do you avoid duplicate API requests?

**Answer:**

- Maintain a loading state
- Skip new requests while loading
- Track `hasMore`
- Abort stale requests if necessary

---

### Q3. What if the API contains millions of records?

**Answer:**

- Server-side pagination
- Infinite scrolling
- Virtualization (`react-window`)
- Lazy loading
- Cached API responses

---

### Q4. Why shouldn't you fetch all records at once?

**Answer:**

Because it increases:

- Initial page load time
- Memory consumption
- Network bandwidth usage
- Browser rendering cost
- Overall application latency

---

### Q5. What libraries can improve infinite scrolling?

- react-window
- react-virtualized
- react-intersection-observer
- TanStack Query (React Query)
- SWR
- Axios

---

# 📚 Key Takeaways

- ✅ Fetch data page by page
- ✅ Use the IntersectionObserver API
- ✅ Append new data instead of replacing existing data
- ✅ Maintain `loading` and `hasMore` state
- ✅ Prevent duplicate requests
- ✅ Handle loading, error, and end-of-data states
- ✅ Disconnect observers to avoid memory leaks
- ✅ Use virtualization (`react-window`) for massive datasets
- ✅ Cache API responses for better performance
- ✅ Build scalable, performant infinite scrolling experiences

---

# 🏆 Conclusion

Infinite scrolling is an effective pattern for displaying large datasets while keeping applications responsive and efficient. By combining **pagination**, **IntersectionObserver**, **proper state management**, and **virtualization**, you can build a scalable solution that delivers a smooth user experience even with millions of records.