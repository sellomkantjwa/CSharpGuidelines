## Performance Guidelines

### Consider using `Any()` to determine whether an `IEnumerable<T>` is empty :large_blue_circle:
When a member or local function returns an `IEnumerable<T>` or other collection class that does not expose a `Count` property, use the `Any()` extension method rather than `Count()` to determine whether the collection contains items. If you do use `Count()`, you risk that iterating over the entire collection might have a significant impact (such as when it really is an `IQueryable<T>` to a persistent store).

**Note:** If you return an `IEnumerable<T>` to prevent changes from calling code, consider read-only classes.

### Beware of mixing up `async`/`await` with `Task.Wait` :red_circle:
`await` does not block the current thread but simply instructs the compiler to generate a state-machine. However, `Task.Wait` blocks the thread and may even cause deadlocks.

### Prefer creating background tasks and batches with Hangfire for CPU-intensive activities :red_circle:
:police_car: Original rule stated that you `Prefer Task.Run for CPU-intensive activities`
Offload any heavy processing to a background task managed by the worker.
