object GlobalScope : CoroutineScope {
override val coroutineContext: CoroutineContext
get() = EmptyCoroutineContext
}
.
