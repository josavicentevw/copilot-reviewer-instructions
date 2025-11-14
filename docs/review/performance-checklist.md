# Performance and Cost Optimization Checklist

This checklist helps identify performance bottlenecks and cost inefficiencies in code changes. Focus on scalability, resource usage, and cloud cost optimization.

---

## 1. Database Query Optimization {#database-queries}

### N+1 Query Prevention
- [ ] No N+1 query patterns (loops that execute queries)
- [ ] Use proper joins, eager loading, or batch queries
- [ ] ORM relationships configured for efficient loading
- [ ] Data fetching optimized with includes/joins

**Example:**
```typescript
// ❌ BAD - N+1 query problem
const users = await User.findAll();
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } });
}

// ✅ GOOD - Single query with join
const users = await User.findAll({
  include: [{ model: Post }]
});
```

### Database Indexes {#database-indexes}
- [ ] Indexes exist for columns used in WHERE clauses
- [ ] Indexes for columns used in ORDER BY and JOIN conditions
- [ ] Composite indexes for multi-column filters
- [ ] Index usage verified with EXPLAIN/EXPLAIN ANALYZE

**Index guidelines:**
- Create indexes on foreign keys
- Index columns used frequently in WHERE clauses
- Consider composite indexes for multi-column queries
- Avoid over-indexing (impacts write performance)

**Kotlin Example:**
```kotlin
// Query that needs an index
// SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at DESC;

// Create appropriate index in migration
@Entity
@Table(
    name = "orders",
    indexes = [
        Index(name = "idx_orders_status_created", columnList = "status,created_at DESC")
    ]
)
data class Order(
    @Id val id: Long,
    @Column(name = "status") val status: String,
    @Column(name = "created_at") val createdAt: Instant
)
```

### Query Optimization
- [ ] SELECT only needed columns (avoid SELECT *)
- [ ] Use LIMIT for queries that may return many rows
- [ ] Avoid unnecessary DISTINCT operations
- [ ] Subqueries optimized or converted to joins where appropriate

---

## 2. Pagination and Streaming {#pagination}

### Large Result Sets
- [ ] Pagination implemented for endpoints returning lists
- [ ] Cursor-based pagination for large datasets (preferred over offset)
- [ ] Maximum page size enforced (e.g., max 100 items)
- [ ] Total count calculated efficiently (or avoided when possible)

**React/TypeScript Example:**
```typescript
// ✅ GOOD - Cursor-based pagination
router.get('/api/posts', async (req, res) => {
  const limit = Math.min(parseInt(req.query.limit) || 20, 100);
  const cursor = req.query.cursor;
  
  const posts = await Post.findAll({
    where: cursor ? { id: { [Op.gt]: cursor } } : {},
    limit: limit + 1,
    order: [['id', 'ASC']]
  });
  
  const hasMore = posts.length > limit;
  const results = hasMore ? posts.slice(0, limit) : posts;
  
  res.json({
    data: results,
    cursor: hasMore ? results[results.length - 1].id : null,
    hasMore
  });
});
```

**Kotlin Example:**
```kotlin
// ✅ GOOD - Cursor-based pagination
@GetMapping("/api/posts")
fun getPosts(
    @RequestParam(defaultValue = "20") limit: Int,
    @RequestParam(required = false) cursor: Long?
): ResponseEntity<PagedResponse<Post>> {
    val pageSize = minOf(limit, 100)
    
    val posts = if (cursor != null) {
        postRepository.findByIdGreaterThanOrderByIdAsc(cursor, PageRequest.of(0, pageSize + 1))
    } else {
        postRepository.findAllByOrderByIdAsc(PageRequest.of(0, pageSize + 1))
    }
    
    val hasMore = posts.size > pageSize
    val results = if (hasMore) posts.dropLast(1) else posts
    
    return ResponseEntity.ok(
        PagedResponse(
            data = results,
            cursor = if (hasMore) results.last().id else null,
            hasMore = hasMore
        )
    )
}
```

### Streaming
- [ ] Large file processing uses streaming (not loading entire file into memory)
- [ ] API responses stream when data size is large
- [ ] Database cursors used for processing large result sets

---

## 3. Caching Strategy {#caching}

### Cache Implementation
- [ ] Caching used for expensive computations or frequent reads
- [ ] Cache keys well-designed and collision-free
- [ ] TTL (Time To Live) defined appropriately
- [ ] Cache invalidation strategy explicit and correct

### Cache Levels
- [ ] Application-level caching for computed values
- [ ] HTTP caching headers set appropriately (Cache-Control, ETag)
- [ ] CDN caching for static assets
- [ ] Database query result caching where appropriate

**React/TypeScript Example:**
```typescript
// ✅ GOOD - Caching with TTL and invalidation
class ProductService {
  constructor(private cache: CacheService) {}
  
  async getProduct(productId: string): Promise<Product> {
    const cacheKey = `product:${productId}`;
    const cached = await this.cache.get<Product>(cacheKey);
    
    if (cached) {
      return cached;
    }
    
    const product = await db.queryProduct(productId);
    await this.cache.set(cacheKey, product, { ttl: 300 }); // 5 minutes
    return product;
  }
  
  async updateProduct(productId: string, data: UpdateProductDto): Promise<void> {
    await db.updateProduct(productId, data);
    // Explicit cache invalidation
    await this.cache.delete(`product:${productId}`);
  }
}
```

**Kotlin Example:**
```kotlin
// ✅ GOOD - Caching with TTL and invalidation
class ProductService(private val cache: CacheService) {
    
    suspend fun getProduct(productId: String): Product {
        val cacheKey = "product:$productId"
        cache.get<Product>(cacheKey)?.let { return it }
        
        val product = db.queryProduct(productId)
        cache.set(cacheKey, product, ttl = 300) // 5 minutes
        return product
    }
    
    suspend fun updateProduct(productId: String, data: UpdateProductDto) {
        db.updateProduct(productId, data)
        // Explicit cache invalidation
        cache.delete("product:$productId")
    }
}
```

### Cache Size Limits
- [ ] In-memory caches have size limits to prevent memory exhaustion
- [ ] LRU or similar eviction policy implemented
- [ ] Cache memory usage monitored

---

## 4. Algorithmic Complexity {#complexity}

### Time Complexity
- [ ] No unnecessary O(n²) operations (nested loops on same dataset)
- [ ] Algorithms use appropriate data structures (hash maps vs arrays)
- [ ] Sorting algorithms efficient for use case
- [ ] Recursive algorithms have proper base cases and bounds

**React/TypeScript Example:**
```typescript
// ❌ BAD - O(n²) complexity
function findDuplicates(arr: number[]): number[] {
  const duplicates: number[] = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) duplicates.push(arr[i]);
    }
  }
  return duplicates;
}

// ✅ GOOD - O(n) complexity with hash set
function findDuplicates(arr: number[]): number[] {
  const seen = new Set<number>();
  const duplicates = new Set<number>();
  
  for (const item of arr) {
    if (seen.has(item)) {
      duplicates.add(item);
    }
    seen.add(item);
  }
  
  return Array.from(duplicates);
}
```

**Kotlin Example:**
```kotlin
// ❌ BAD - O(n²) complexity
fun findDuplicates(arr: List<Int>): List<Int> {
    val duplicates = mutableListOf<Int>()
    for (i in arr.indices) {
        for (j in i + 1 until arr.size) {
            if (arr[i] == arr[j]) duplicates.add(arr[i])
        }
    }
    return duplicates
}

// ✅ GOOD - O(n) complexity with hash set
fun findDuplicates(arr: List<Int>): List<Int> {
    val seen = mutableSetOf<Int>()
    val duplicates = mutableSetOf<Int>()
    
    for (item in arr) {
        if (item in seen) {
            duplicates.add(item)
        }
        seen.add(item)
    }
    
    return duplicates.toList()
}
```

### Space Complexity
- [ ] Memory usage reasonable for expected data sizes
- [ ] No unnecessary data copying
- [ ] Large data structures created only when necessary
- [ ] Resources released when no longer needed

---

## 5. Network and I/O Optimization

### HTTP Requests
- [ ] Batch API calls when possible (reduce round trips)
- [ ] Parallel requests used for independent operations
- [ ] Connection pooling configured for HTTP clients
- [ ] Request/response compression enabled (gzip, brotli)

### File I/O
- [ ] Files read/written in chunks for large files
- [ ] Buffered I/O used appropriately
- [ ] Temporary files cleaned up
- [ ] File operations async where possible

**React/TypeScript Example:**
```typescript
// ❌ BAD - Sequential API calls
async function getUserData(userIds: string[]) {
  const users = [];
  for (const id of userIds) {
    const user = await api.getUser(id);
    users.push(user);
  }
  return users;
}

// ✅ GOOD - Parallel API calls
async function getUserData(userIds: string[]) {
  return await Promise.all(
    userIds.map(id => api.getUser(id))
  );
}

// ✅ BETTER - Batch API endpoint
async function getUserData(userIds: string[]) {
  return await api.getUsers({ ids: userIds });
}
```

**Kotlin Example:**
```kotlin
// ❌ BAD - Sequential API calls
suspend fun getUserData(userIds: List<String>): List<User> {
    val users = mutableListOf<User>()
    for (id in userIds) {
        val user = api.getUser(id)
        users.add(user)
    }
    return users
}

// ✅ GOOD - Parallel API calls with coroutines
suspend fun getUserData(userIds: List<String>): List<User> {
    return coroutineScope {
        userIds.map { id ->
            async { api.getUser(id) }
        }.awaitAll()
    }
}

// ✅ BETTER - Batch API endpoint
suspend fun getUserData(userIds: List<String>): List<User> {
    return api.getUsers(userIds)
}
```

---

## 6. Cloud Cost Optimization

### Resource Usage
- [ ] Auto-scaling configured appropriately
- [ ] Resources right-sized (not over-provisioned)
- [ ] Spot instances or reserved capacity considered
- [ ] Scheduled scaling for predictable workloads

### Data Transfer
- [ ] Minimize cross-region or cross-AZ data transfer
- [ ] Data stored in same region as compute when possible
- [ ] CloudFront or CDN used for static assets
- [ ] Compression enabled for data transfer

### Database Costs
- [ ] Database instance size appropriate for workload
- [ ] Read replicas used for read-heavy workloads
- [ ] Database connection pooling prevents connection exhaustion
- [ ] Old/unused data archived or deleted

---

## 7. Frontend Performance

### Asset Optimization
- [ ] Images optimized and properly sized
- [ ] Lazy loading for images and components
- [ ] Code splitting implemented
- [ ] Bundle size monitored and optimized

### Rendering Performance
- [ ] Avoid unnecessary re-renders (React.memo, useMemo, useCallback)
- [ ] Virtualization for long lists
- [ ] Debouncing/throttling for frequent events
- [ ] Web Vitals metrics monitored (LCP, FID, CLS)

---

## 8. Monitoring and Profiling

### Performance Metrics
- [ ] Response time tracked for critical endpoints
- [ ] Database query times monitored
- [ ] Resource usage (CPU, memory) tracked
- [ ] Performance regressions detected in CI/CD

### Profiling Tools
- [ ] Profiling done for performance-critical code
- [ ] Bottlenecks identified and documented
- [ ] Load testing performed for high-traffic features

---

## Review Checklist Summary

Use this quick checklist during PR reviews:

- [ ] **Database**: No N+1 queries, indexes exist, queries optimized
- [ ] **Pagination**: Implemented for large datasets, limits enforced
- [ ] **Caching**: TTL defined, invalidation strategy clear
- [ ] **Complexity**: No unnecessary O(n²), appropriate data structures
- [ ] **Network**: Batched requests, parallel execution, compression
- [ ] **Cloud Cost**: Right-sized resources, data transfer optimized
- [ ] **Frontend**: Assets optimized, rendering efficient
- [ ] **Monitoring**: Metrics tracked, profiling done

---

## Performance Testing Commands

**Node.js profiling:**
```bash
node --prof app.js                    # CPU profiling
node --inspect app.js                 # Chrome DevTools profiling
clinic doctor -- node app.js          # Comprehensive diagnostics
```

**Database query analysis:**
```sql
EXPLAIN ANALYZE SELECT ...;           # PostgreSQL
EXPLAIN SELECT ...;                   # MySQL
```

**Load testing:**
```bash
ab -n 1000 -c 10 http://localhost/   # Apache Bench
wrk -t12 -c400 -d30s http://localhost # wrk
```

---

## References

- [Web Performance Best Practices](https://web.dev/fast/)
- [Database Indexing Strategies](https://use-the-index-luke.com/)
- [AWS Well-Architected Framework - Performance Efficiency](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/)
- [Google Cloud Performance Optimization](https://cloud.google.com/architecture/performance)
