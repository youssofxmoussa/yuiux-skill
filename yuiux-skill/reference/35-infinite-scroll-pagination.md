# 35 — Infinite Scroll & Pagination

> Pagination gives users control; infinite scroll keeps them moving. Choose based on whether users need to find, browse, or consume.

---

## Choose the Right Pattern

| Use Case | Best Pattern | Why |
|----------|-------------|-----|
| Search results | **Pagination** | Users jump to specific pages, share links |
| Social feed | **Infinite scroll** | Exploratory browsing, no specific target |
| Admin tables | **Pagination** | Users filter, sort, need page context |
| News/content | **Load more button** | Users decide when to load more |
| E-commerce list | **Pagination or Load more** | Allows returning to specific position |
| File list | **Pagination** | Users need to find specific item |
| Chat/messages | **Infinite scroll (bidirectional)** | Messages flow continuously |

**Key signal:** If users need to go "back to page 3," use pagination. If users just keep scrolling forward, infinite scroll is fine.

---

## Pagination

```tsx
interface PaginationProps {
  page: number;
  totalPages: number;
  onChange: (page: number) => void;
  siblingCount?: number;
}

function Pagination({ page, totalPages, onChange, siblingCount = 1 }: PaginationProps) {
  const pageNumbers = usePaginationRange({ page, totalPages, siblingCount });
  
  return (
    <nav aria-label="Pagination navigation" className="pagination">
      {/* Previous */}
      <button
        type="button"
        onClick={() => onChange(page - 1)}
        disabled={page <= 1}
        aria-label="Previous page"
        className="pagination-btn"
      >
        <ChevronLeftIcon aria-hidden />
        <span className="pagination-btn-label">Previous</span>
      </button>
      
      {/* Page numbers */}
      <ol className="pagination-pages" role="list">
        {pageNumbers.map((pageNum, index) => (
          pageNum === 'ellipsis' ? (
            <li key={`ellipsis-${index}`} className="pagination-ellipsis" aria-hidden>
              …
            </li>
          ) : (
            <li key={pageNum}>
              <button
                type="button"
                onClick={() => onChange(pageNum)}
                aria-label={`Page ${pageNum}`}
                aria-current={pageNum === page ? 'page' : undefined}
                className={`pagination-page ${pageNum === page ? 'active' : ''}`}
              >
                {pageNum}
              </button>
            </li>
          )
        ))}
      </ol>
      
      {/* Next */}
      <button
        type="button"
        onClick={() => onChange(page + 1)}
        disabled={page >= totalPages}
        aria-label="Next page"
        className="pagination-btn"
      >
        <span className="pagination-btn-label">Next</span>
        <ChevronRightIcon aria-hidden />
      </button>
    </nav>
  );
}

// Pagination range calculation with ellipsis
function usePaginationRange({
  page,
  totalPages,
  siblingCount = 1,
}: {
  page: number;
  totalPages: number;
  siblingCount?: number;
}): (number | 'ellipsis')[] {
  return useMemo(() => {
    const totalPageNumbers = siblingCount * 2 + 5; // siblings + first + last + current + 2 ellipsis
    
    if (totalPageNumbers >= totalPages) {
      return range(1, totalPages);
    }
    
    const leftSiblingIndex  = Math.max(page - siblingCount, 1);
    const rightSiblingIndex = Math.min(page + siblingCount, totalPages);
    const showLeftEllipsis  = leftSiblingIndex > 2;
    const showRightEllipsis = rightSiblingIndex < totalPages - 2;
    
    if (!showLeftEllipsis && showRightEllipsis) {
      const leftRange = range(1, 3 + siblingCount * 2);
      return [...leftRange, 'ellipsis', totalPages];
    }
    
    if (showLeftEllipsis && !showRightEllipsis) {
      const rightRange = range(totalPages - (3 + siblingCount * 2) + 1, totalPages);
      return [1, 'ellipsis', ...rightRange];
    }
    
    const middleRange = range(leftSiblingIndex, rightSiblingIndex);
    return [1, 'ellipsis', ...middleRange, 'ellipsis', totalPages];
  }, [page, totalPages, siblingCount]);
}

function range(start: number, end: number): number[] {
  return Array.from({ length: end - start + 1 }, (_, i) => start + i);
}
```

### URL-Synced Pagination

```tsx
// Keep pagination state in URL — allows deep linking, browser back/forward
function usePaginatedData<T>(endpoint: string, pageSize = 25) {
  const [searchParams, setSearchParams] = useSearchParams();
  const page = Number(searchParams.get('page') ?? '1');
  
  const { data, isLoading } = useQuery({
    queryKey: [endpoint, page, pageSize],
    queryFn: () => fetchPage<T>(endpoint, page, pageSize),
    keepPreviousData: true, // keeps old data while new page loads
  });
  
  function setPage(newPage: number) {
    setSearchParams(prev => {
      const next = new URLSearchParams(prev);
      next.set('page', String(newPage));
      return next;
    });
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
  
  return { data, isLoading, page, setPage, totalPages: data?.totalPages ?? 1 };
}
```

---

## Infinite Scroll with Intersection Observer

```tsx
function useInfiniteScroll({
  onLoadMore,
  hasNextPage,
  loading,
  rootMargin = '200px', // load when within 200px of viewport bottom
}: InfiniteScrollOptions) {
  const sentinelRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const el = sentinelRef.current;
    if (!el) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting && hasNextPage && !loading) {
          onLoadMore();
        }
      },
      { rootMargin }
    );
    
    observer.observe(el);
    return () => observer.disconnect();
  }, [hasNextPage, loading, onLoadMore, rootMargin]);
  
  return { sentinelRef };
}

// Usage with react-query infinite query
function InfiniteFeed() {
  const {
    data,
    isLoading,
    isFetchingNextPage,
    hasNextPage,
    fetchNextPage,
    isError,
    refetch,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam = 1 }) => fetchFeedPage(pageParam),
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });
  
  const { sentinelRef } = useInfiniteScroll({
    onLoadMore: fetchNextPage,
    hasNextPage: !!hasNextPage,
    loading: isFetchingNextPage,
  });
  
  const allItems = data?.pages.flatMap(p => p.items) ?? [];
  
  if (isLoading) return <FeedSkeleton />;
  if (isError) return <ErrorState onRetry={refetch} />;
  if (!allItems.length) return <EmptyFeed />;
  
  return (
    <>
      <ul aria-label="Feed" role="list">
        {allItems.map((item, i) => (
          <li key={item.id}>
            <FeedItem item={item} />
          </li>
        ))}
      </ul>
      
      {/* Loading more */}
      {isFetchingNextPage && (
        <div aria-live="polite" aria-label="Loading more items">
          <FeedItemSkeleton />
          <FeedItemSkeleton />
        </div>
      )}
      
      {/* End of feed */}
      {!hasNextPage && allItems.length > 0 && (
        <p className="feed-end" aria-live="polite">
          You've reached the end.
        </p>
      )}
      
      {/* Invisible sentinel element */}
      <div ref={sentinelRef} aria-hidden="true" />
    </>
  );
}
```

---

## Load More Button (Hybrid)

Best of both: user controls loading, no infinite scroll issues with footer access.

```tsx
function LoadMoreList<T>({
  initialItems,
  loadMore,
  renderItem,
  emptyMessage = 'No items found.',
}: LoadMoreListProps<T>) {
  const [items, setItems] = useState(initialItems.items);
  const [cursor, setCursor] = useState(initialItems.nextCursor);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  
  // Keep count for announcement
  const prevCount = useRef(items.length);
  const newlyLoadedCount = items.length - prevCount.current;
  
  async function handleLoadMore() {
    if (!cursor || loading) return;
    setLoading(true);
    setError('');
    
    try {
      const result = await loadMore(cursor);
      prevCount.current = items.length;
      setItems(prev => [...prev, ...result.items]);
      setCursor(result.nextCursor);
    } catch {
      setError('Failed to load more. Try again.');
    } finally {
      setLoading(false);
    }
  }
  
  if (!items.length) return <p>{emptyMessage}</p>;
  
  return (
    <div>
      {/* Announce newly loaded items */}
      <div aria-live="polite" aria-atomic className="sr-only">
        {newlyLoadedCount > 0 && `${newlyLoadedCount} more items loaded.`}
      </div>
      
      <ul role="list">
        {items.map(item => (
          <li key={(item as any).id}>{renderItem(item)}</li>
        ))}
      </ul>
      
      {error && <p role="alert" className="error">{error}</p>}
      
      {cursor && (
        <div className="load-more-container">
          <LoadingButton
            type="button"
            loading={loading}
            onClick={handleLoadMore}
            className="btn btn-outline load-more-btn"
          >
            {loading ? 'Loading…' : 'Load more'}
          </LoadingButton>
        </div>
      )}
      
      {!cursor && <p className="list-end">All items loaded.</p>}
    </div>
  );
}
```

---

## Virtual Scrolling (Large Lists)

For lists with thousands of items, render only visible rows:

```tsx
import { FixedSizeList, VariableSizeList } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

// Fixed height rows
function VirtualFixedList({ items }: { items: Item[] }) {
  const ROW_HEIGHT = 64;
  
  function Row({ index, style }: { index: number; style: React.CSSProperties }) {
    const item = items[index];
    return (
      <div style={style} className="virtual-row">
        <ListItem item={item} />
      </div>
    );
  }
  
  return (
    <AutoSizer>
      {({ height, width }) => (
        <FixedSizeList
          height={height}
          width={width}
          itemCount={items.length}
          itemSize={ROW_HEIGHT}
          overscanCount={3} // render 3 extra rows above/below viewport
        >
          {Row}
        </FixedSizeList>
      )}
    </AutoSizer>
  );
}
```

---

## Pagination Checklist

- [ ] Pagination synced to URL (`?page=3`) for deep linking
- [ ] `aria-label="Pagination navigation"` on nav element
- [ ] `aria-current="page"` on current page button
- [ ] Prev/Next buttons disabled at boundaries (not hidden)
- [ ] `keepPreviousData: true` in react-query to avoid blank flash
- [ ] Page resets to 1 when filters/sort changes
- [ ] Scroll to top on page change
- [ ] Screen reader announces page change

### Infinite Scroll Checklist

- [ ] Load triggered 200px before sentinel (not exactly at bottom)
- [ ] Skeleton items shown while loading next page
- [ ] "You've reached the end" message when no more pages
- [ ] Error recovery if page fails to load
- [ ] "Back to top" button accessible for long feeds
- [ ] Screen reader live region announces "Loading more" and count loaded
- [ ] Footer still accessible (not covered by sentinel loading)

### Virtual Scroll Checklist

- [ ] Used for lists > 500 items
- [ ] `overscanCount` set (3–5 rows) to prevent visual pop-in
- [ ] Accessibility: focusable items within virtual list
- [ ] Keyboard navigation works within virtual list
