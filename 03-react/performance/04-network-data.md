# 🌐 Network & Data Fetching Performance

> Tối ưu cách **tải, cache, và đồng bộ dữ liệu** — giảm waterfall, tránh over-fetching, tăng perceived speed.

---

## Mục Lục

1. [Vấn Đề Phổ Biến Với Data Fetching](#1-vấn-đề-phổ-biến-với-data-fetching)
2. [Request Waterfall — Nhận Dạng & Fix](#2-request-waterfall--nhận-dạng--fix)
3. [Caching Strategies](#3-caching-strategies)
4. [React Query — Data Fetching Done Right](#4-react-query--data-fetching-done-right)
5. [Optimistic Updates](#5-optimistic-updates)
6. [Prefetching & Preloading Data](#6-prefetching--preloading-data)
7. [Infinite Scroll vs Pagination](#7-infinite-scroll-vs-pagination)
8. [Real-time Data — WebSocket & SSE](#8-real-time-data--websocket--sse)

---

## 1. Vấn Đề Phổ Biến Với Data Fetching

### Các Anti-patterns Hay Gặp

| Anti-pattern | Hậu quả | Fix |
|-------------|---------|-----|
| Fetch trong `useEffect` không có cache | Refetch mỗi lần mount | React Query / SWR |
| Request waterfall | Trang load chậm, nhiều round trips | Parallel fetch, data loader |
| Over-fetching | Tải dữ liệu không cần dùng | GraphQL, field selection |
| Under-fetching | Nhiều request cho 1 tính năng | API composition, BFF |
| No loading state | UX xấu, layout shift | Skeleton UI |
| No error handling | Silent failures | Error boundaries |
| Race condition | Dữ liệu cũ ghi đè mới | AbortController, cancel |

---

## 2. Request Waterfall — Nhận Dạng & Fix

### Vấn Đề — Sequential Requests

```
Waterfall (BAD):
Timeline: ──────────────────────────────────────────►
  fetchUser:        ████████░░░░░░░░░░░░░░░░  (500ms)
  fetchUserPosts:           ████████░░░░░░░░  (500ms, đợi user xong)
  fetchComments:                    ████████  (500ms, đợi posts xong)
  Total: 1500ms ❌

Parallel (GOOD):
Timeline: ──────────────────────────────────────────►
  fetchUser:        ████████  (500ms)
  fetchUserPosts:   ████████  (500ms, chạy song song)
  fetchComments:    ████████  (500ms, chạy song song)
  Total: 500ms ✅
```

### ❌ Waterfall Pattern

```jsx
// ❌ Mỗi request đợi cái trước xong
function ProfilePage({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [followers, setFollowers] = useState([]);

  useEffect(() => {
    fetchUser(userId).then(u => {
      setUser(u);
      // ❌ Đợi user xong mới fetch posts
      fetchUserPosts(u.id).then(p => {
        setPosts(p);
        // ❌ Đợi posts xong mới fetch comments
        fetchFollowers(u.id).then(setFollowers);
      });
    });
  }, [userId]);
}
```

### ✅ Parallel Fetch

```jsx
// ✅ Fetch song song với Promise.all
function ProfilePage({ userId }) {
  const [data, setData] = useState({ user: null, posts: [], followers: [] });
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();

    Promise.all([
      fetchUser(userId, controller.signal),
      fetchUserPosts(userId, controller.signal),
      fetchFollowers(userId, controller.signal),
    ])
      .then(([user, posts, followers]) => {
        setData({ user, posts, followers });
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') setLoading(false);
      });

    return () => controller.abort();
  }, [userId]);

  if (loading) return <ProfileSkeleton />;
  return <Profile {...data} />;
}
```

### ✅ React Query — Parallel Queries

```jsx
import { useQueries } from '@tanstack/react-query';

function ProfilePage({ userId }) {
  const results = useQueries({
    queries: [
      { queryKey: ['user', userId], queryFn: () => fetchUser(userId) },
      { queryKey: ['posts', userId], queryFn: () => fetchUserPosts(userId) },
      { queryKey: ['followers', userId], queryFn: () => fetchFollowers(userId) },
    ],
  });

  const [userQuery, postsQuery, followersQuery] = results;
  const isLoading = results.some(r => r.isLoading);
  const isError = results.some(r => r.isError);

  if (isLoading) return <ProfileSkeleton />;
  if (isError) return <ErrorMessage />;

  return (
    <Profile
      user={userQuery.data}
      posts={postsQuery.data}
      followers={followersQuery.data}
    />
  );
}
```

---

## 3. Caching Strategies

### Stale-While-Revalidate (SWR) Pattern

```
Request 1 (cache miss): Fetch → Store cache → Return data
Request 2 (cache hit):  Return cache ngay (fast!) → Revalidate in background
Request 3 (background done): Update cache → Re-render nếu data mới
```

```jsx
// Implement SWR thủ công (để hiểu cơ chế)
const cache = new Map();

function useSWR(key, fetcher) {
  const [state, setState] = useState({
    data: cache.get(key) || null,
    isLoading: !cache.has(key),
    error: null,
  });

  useEffect(() => {
    let cancelled = false;

    fetcher().then(data => {
      if (cancelled) return;
      cache.set(key, data); // Lưu cache
      setState({ data, isLoading: false, error: null });
    }).catch(error => {
      if (cancelled) return;
      setState(prev => ({ ...prev, isLoading: false, error }));
    });

    return () => { cancelled = true; };
  }, [key]);

  return state;
}
```

### Cache Invalidation với React Query

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
    staleTime: 5 * 60 * 1000,   // Data fresh trong 5 phút
    gcTime: 10 * 60 * 1000,     // Giữ cache 10 phút (sau khi component unmount)
    refetchOnWindowFocus: false, // Không refetch khi tab được focus lại
  });
}

function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createProduct,
    onSuccess: (newProduct) => {
      // ✅ Invalidate cache → trigger refetch
      queryClient.invalidateQueries({ queryKey: ['products'] });

      // ✅ Hoặc update cache trực tiếp (không cần refetch)
      queryClient.setQueryData(['products'], (old) => [...old, newProduct]);
    },
  });
}
```

### `staleTime` vs `gcTime` (cacheTime)

```
Timeline sau khi data fetch:

                   staleTime (5 min)
                   │
Fresh    ──────────┤ Stale (data cũ, nhưng vẫn dùng được)
                   │
                   │         gcTime (10 min, sau unmount)
                   │         │
[Component unmount]          ├─────────────────────────
                             │ Cache còn đây (background)
                             │
                             [Cache bị xóa] → phải fetch lại
```

---

## 4. React Query — Data Fetching Done Right

### Setup

```jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,      // 1 phút
      retry: 1,                  // Retry 1 lần khi lỗi
      refetchOnWindowFocus: true,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Router />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### useQuery — Đầy Đủ Options

```jsx
function ProductList({ category }) {
  const {
    data,
    isLoading,
    isFetching,   // true khi đang background refetch
    isError,
    error,
    refetch,
    dataUpdatedAt,
  } = useQuery({
    queryKey: ['products', category],  // Cache key — thay đổi khi category thay đổi
    queryFn: () => fetchProducts({ category }),
    enabled: !!category,               // Chỉ fetch khi có category
    staleTime: 2 * 60 * 1000,
    select: (data) => data.items,      // Transform data trước khi return
    placeholderData: (previousData) => previousData, // Hiển thị data cũ khi loading
  });

  return (
    <div>
      {isFetching && <div className="refresh-indicator">Đang cập nhật...</div>}
      {isLoading ? <Skeleton /> : <ProductGrid products={data} />}
    </div>
  );
}
```

### useMutation — Create/Update/Delete

```jsx
function AddProductForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newProduct) => api.post('/products', newProduct),

    onMutate: async (newProduct) => {
      // Cancel đang-chạy queries để tránh conflict
      await queryClient.cancelQueries({ queryKey: ['products'] });

      // Snapshot data cũ để rollback nếu lỗi
      const snapshot = queryClient.getQueryData(['products']);

      // Optimistic update
      queryClient.setQueryData(['products'], (old) => [
        ...old,
        { ...newProduct, id: Date.now(), status: 'pending' }
      ]);

      return { snapshot }; // Trả về để dùng trong onError
    },

    onError: (err, newProduct, context) => {
      // Rollback về data cũ nếu lỗi
      queryClient.setQueryData(['products'], context.snapshot);
    },

    onSettled: () => {
      // Invalidate để fetch data thực từ server
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });

  const handleSubmit = (formData) => {
    mutation.mutate(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* ... */}
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Đang thêm...' : 'Thêm sản phẩm'}
      </button>
      {mutation.isError && <p>Lỗi: {mutation.error.message}</p>}
    </form>
  );
}
```

---

## 5. Optimistic Updates

### Nguyên Tắc

```
User action → Cập nhật UI ngay (optimistic) → Gọi API
                                              ↓
                                    Thành công → Confirm với data thật
                                    Thất bại → Revert về state cũ
```

### Với `useOptimistic` (React 19)

```jsx
function TodoList({ todos, onUpdate }) {
  const [optimisticTodos, addOptimistic] = useOptimistic(
    todos,
    (state, { type, id, text }) => {
      switch (type) {
        case 'ADD':
          return [...state, { id, text, completed: false, pending: true }];
        case 'TOGGLE':
          return state.map(t => t.id === id ? { ...t, completed: !t.completed } : t);
        case 'DELETE':
          return state.filter(t => t.id !== id);
        default:
          return state;
      }
    }
  );

  async function toggleTodo(id) {
    addOptimistic({ type: 'TOGGLE', id }); // UI cập nhật ngay
    await api.toggleTodo(id); // API có thể mất 300ms
    onUpdate(); // Refetch để sync với server
  }

  return (
    <ul>
      {optimisticTodos.map(todo => (
        <li
          key={todo.id}
          style={{ opacity: todo.pending ? 0.6 : 1 }}
          onClick={() => toggleTodo(todo.id)}
        >
          {todo.completed ? '✅' : '⬜'} {todo.text}
          {todo.pending && ' (đang lưu...)'}
        </li>
      ))}
    </ul>
  );
}
```

---

## 6. Prefetching & Preloading Data

### Hover-to-Prefetch

```jsx
import { useQueryClient } from '@tanstack/react-query';

function ProductCard({ product }) {
  const queryClient = useQueryClient();

  // Prefetch data khi hover — gần như chắc chắn user sẽ click
  const handleMouseEnter = () => {
    queryClient.prefetchQuery({
      queryKey: ['product', product.id],
      queryFn: () => fetchProductDetail(product.id),
      staleTime: 10 * 60 * 1000, // Cache 10 phút
    });
  };

  return (
    <div onMouseEnter={handleMouseEnter}>
      <img src={product.thumbnail} alt={product.name} />
      <h3>{product.name}</h3>
    </div>
  );
}
```

### Route-level Prefetch (Next.js)

```jsx
// Next.js Link tự động prefetch khi visible in viewport
import Link from 'next/link';

function ProductGrid({ products }) {
  return products.map(product => (
    // ✅ Next.js tự prefetch /products/[id] khi link visible
    <Link key={product.id} href={`/products/${product.id}`} prefetch>
      <ProductCard product={product} />
    </Link>
  ));
}
```

### Prefetch Data Trên Server (Next.js App Router)

```jsx
// app/products/page.tsx — Server Component
import { dehydrate, HydrationBoundary, QueryClient } from '@tanstack/react-query';

async function ProductsPage() {
  const queryClient = new QueryClient();

  // Prefetch trên server → gửi kèm HTML → client không cần fetch lại
  await queryClient.prefetchQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <ProductList /> {/* Client component — data đã có sẵn */}
    </HydrationBoundary>
  );
}
```

---

## 7. Infinite Scroll vs Pagination

### Infinite Scroll với React Query

```jsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { useIntersectionObserver } from './hooks/useIntersectionObserver';

function ProductFeed() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['products', 'feed'],
    queryFn: ({ pageParam }) => fetchProducts({ cursor: pageParam, limit: 20 }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });

  // Sentinel element để detect khi user scroll đến cuối
  const { ref: sentinelRef } = useIntersectionObserver({
    threshold: 0.1,
    onChange: (isIntersecting) => {
      if (isIntersecting && hasNextPage && !isFetchingNextPage) {
        fetchNextPage();
      }
    },
  });

  const allProducts = data?.pages.flatMap(page => page.items) ?? [];

  return (
    <div>
      <ProductGrid products={allProducts} />

      {/* Sentinel element — khi visible → load thêm */}
      <div ref={sentinelRef}>
        {isFetchingNextPage && <Spinner />}
        {!hasNextPage && <p>Đã hiển thị tất cả sản phẩm</p>}
      </div>
    </div>
  );
}
```

### Khi Nào Dùng Pagination vs Infinite Scroll?

| | Pagination | Infinite Scroll |
|---|-----------|----------------|
| SEO | ✅ Tốt (URLs riêng) | ❌ Khó index |
| Bookmark / Share | ✅ URL cụ thể | ❌ Không share được vị trí |
| Performance | ✅ Load đúng số lượng | ⚠️ DOM tăng dần |
| UX — Tasks | ✅ Tìm item cụ thể | ❌ Khó |
| UX — Browsing | ❌ Friction | ✅ Mượt mà |
| Dùng cho | Admin, search results | Social feed, content |

---

## 8. Real-time Data — WebSocket & SSE

### Server-Sent Events (SSE) — Một Chiều

```jsx
function LiveDashboard() {
  const [metrics, setMetrics] = useState(null);

  useEffect(() => {
    // SSE — server push, HTTP/1.1 compatible
    const eventSource = new EventSource('/api/metrics/stream');

    eventSource.onmessage = (event) => {
      setMetrics(JSON.parse(event.data));
    };

    eventSource.onerror = () => {
      eventSource.close();
      // Reconnect sau 5 giây
      setTimeout(() => eventSource, 5000);
    };

    return () => eventSource.close();
  }, []);

  return <MetricsView data={metrics} />;
}
```

### WebSocket — Hai Chiều

```jsx
function useWebSocket(url) {
  const [status, setStatus] = useState('connecting');
  const [messages, setMessages] = useState([]);
  const wsRef = useRef(null);

  useEffect(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;

    ws.onopen  = () => setStatus('connected');
    ws.onclose = () => setStatus('disconnected');
    ws.onerror = () => setStatus('error');
    ws.onmessage = (event) => {
      setMessages(prev => [...prev, JSON.parse(event.data)]);
    };

    return () => {
      ws.close();
      wsRef.current = null;
    };
  }, [url]);

  const send = useCallback((data) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(data));
    }
  }, []);

  return { status, messages, send };
}

function LiveChat({ roomId }) {
  const { status, messages, send } = useWebSocket(
    `wss://api.example.com/chat/${roomId}`
  );

  return (
    <div>
      <span>Status: {status}</span>
      <MessageList messages={messages} />
      <ChatInput onSend={send} disabled={status !== 'connected'} />
    </div>
  );
}
```

### React Query + WebSocket — Kết Hợp Tốt Nhất

```jsx
// Fetch initial data bằng React Query, real-time updates qua WebSocket
function LiveProductList() {
  const queryClient = useQueryClient();

  // Fetch initial data
  const { data: products } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  });

  // Real-time updates
  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com/products/updates');

    ws.onmessage = (event) => {
      const { type, product } = JSON.parse(event.data);

      if (type === 'UPDATE') {
        // Cập nhật cache trực tiếp — không cần refetch
        queryClient.setQueryData(['products'], (old) =>
          old.map(p => p.id === product.id ? product : p)
        );
      } else if (type === 'CREATE') {
        queryClient.setQueryData(['products'], (old) => [...old, product]);
      }
    };

    return () => ws.close();
  }, [queryClient]);

  return <ProductGrid products={products} />;
}
```

---

## 📎 Tổng Kết

```
Data load chậm?
  □ Parallel fetch thay vì waterfall
  □ React Query với staleTime phù hợp
  □ Prefetch khi hover, route transition

UX lag sau mutation?
  □ Optimistic updates (useOptimistic / React Query onMutate)
  □ Không chờ API để cập nhật UI

Real-time data?
  □ SSE cho push đơn giản, WebSocket cho 2 chiều
  □ Kết hợp React Query (initial) + WebSocket (updates)

Cache không hiệu quả?
  □ staleTime phù hợp với data volatility
  □ invalidateQueries sau mutation
  □ Prefetch server-side với Next.js HydrationBoundary
```

---

🔙 [State & Data Flow](./03-state-data-perf.md) | 🔙 [README](./README.md) | ➡️ [Interview Q&A](./05-interview-qna.md)
