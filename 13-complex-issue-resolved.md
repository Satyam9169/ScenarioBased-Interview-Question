# Complex Issue Resolved -- POS Application (React)

## Scenario: Product Search Performance Optimization

### Situation

In our POS (Point of Sale) application, the product catalog contained
more than **30,000 products**. Cashiers used the search feature while
creating bills, but the application was making an API request on **every
keystroke**.

This resulted in: - Slow search results - High network traffic -
Increased backend load - Poor user experience during billing

------------------------------------------------------------------------

## Task

My responsibility was to optimize the product search without changing
the existing functionality or user experience. The goal was to make the
search faster and reduce unnecessary API requests.

------------------------------------------------------------------------

## Action

### 1. Identified the Root Cause

-   Used **Chrome DevTools (Network tab)** to inspect API requests.
-   Found that each key press triggered a new API call.

### 2. Implemented Debounced Search

Instead of calling the API immediately, I added a **300--500 ms
debounce** so the request was sent only after the user paused typing.

**Benefits:** - Reduced unnecessary API calls - Lower backend load -
Faster user experience

### 3. Optimized Rendering

To avoid unnecessary component re-renders, I used: - `React.memo` -
`useMemo` - Stable `key` props for product lists

### 4. Used Server-side Pagination

The backend already supported pagination, so we requested only the
required products instead of fetching the entire catalog.

### 5. Improved User Experience

Added: - Loading indicator while searching - Empty-state message when no
products matched - Error handling for failed API requests

------------------------------------------------------------------------

## Result

-   Significantly reduced the number of API requests.
-   Faster product search and smoother billing experience.
-   Lower backend load and improved application responsiveness.
-   Better overall user experience for cashiers during peak business
    hours.

------------------------------------------------------------------------

# Technologies Used

-   React.js
-   Axios
-   JavaScript (ES6+)
-   React.memo
-   useMemo
-   Debouncing
-   REST APIs
-   Chrome DevTools

------------------------------------------------------------------------

# Interview Answer (STAR)

**Situation:**\
In our POS application, the product catalog had more than 30,000
products. Every keystroke in the search box triggered an API call,
making the search slow and increasing server load.

**Task:**\
I was responsible for improving the search performance while maintaining
a smooth user experience.

**Action:**\
I analyzed the issue using Chrome DevTools, implemented a debounced
search (300--500 ms), optimized rendering with `React.memo` and
`useMemo`, and leveraged the backend's paginated search API so only
relevant results were fetched.

**Result:**\
The number of API requests dropped significantly, product search became
much faster, backend load was reduced, and cashiers were able to
complete billing more efficiently.

------------------------------------------------------------------------

## Key Learnings

-   Analyze performance issues before optimizing.
-   Debouncing is ideal for search inputs.
-   Render optimization is as important as network optimization.
-   Pagination improves scalability for large datasets.
-   Measure improvements using browser developer tools.
