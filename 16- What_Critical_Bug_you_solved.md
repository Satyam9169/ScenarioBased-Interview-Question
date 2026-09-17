One of the most critical bugs I encountered in my React project was related to duplicate API calls and race conditions in a search functionality.

We had a product search page where users could type keywords and get results from the backend API. The issue was that when users typed quickly, multiple API requests were triggered simultaneously. Sometimes, an older API response arrived after a newer response and overwrote the latest search results.

For example, if a user searched for "mobile" and immediately changed it to "laptop", the API response for "mobile" could arrive later than "laptop". As a result, the UI displayed incorrect search results.

### How I Identified the Bug

Initially, users reported that search results were inconsistent and sometimes showed incorrect data.

I investigated the issue using:

1. Browser Developer Tools Network tab to monitor API requests and response timing.
2. Console logging request IDs and search keywords.
3. React DevTools to inspect component re-renders and state updates.
4. Reproducing the issue with slow network throttling.

After investigation, I found that every keystroke was triggering a new API call without cancelling previous requests or controlling the response order.

### How I Fixed It

I implemented a debounce mechanism to reduce unnecessary API calls and used AbortController to cancel previous requests when a new search request was initiated.

Additionally, I ensured that only the latest request was allowed to update the component state.

The improved flow was:

- User types in the search box.
- Wait for a short debounce period.
- Cancel the previous API request if it is still running.
- Send the latest API request.
- Update the UI only with the latest valid response.

This resolved the race condition and significantly reduced unnecessary API calls.

### Impact

After implementing the fix:

- Incorrect search results were eliminated.
- API calls were reduced significantly.
- The user experience became smoother.
- Backend load was reduced.
- The application became more reliable under slow network conditions.

### What I Learned

The biggest lesson I learned was that frontend applications are not only about rendering UI. We also need to handle asynchronous behavior carefully.

I learned the importance of:

- Managing race conditions in asynchronous API calls.
- Cancelling unnecessary network requests.
- Using debounce for search and user-input-driven APIs.
- Testing applications under slow network conditions.
- Monitoring real-world production issues instead of relying only on local development testing.

Since then, I always consider API cancellation, loading states, error handling, and stale responses when implementing asynchronous features.