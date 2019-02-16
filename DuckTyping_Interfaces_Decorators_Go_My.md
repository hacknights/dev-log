# DuckTyping, Interfaces and Decorators, Go My!

One strange fact about DuckTyping, Interfaces and Decorators...  it's hard to predict which methods "must" be wrapped to maintain [Liskov Substitution](https://en.wikipedia.org/wiki/Liskov_substitution_principle).

*tl;dr* always embed the subject of the decorator instead of creating a named field... Embedding allows you to intercept the calls you care about, while automatically delegating any other calls to the embedded instance.

## Example

When writing middleware, it's sometimes convenient to wrap the http.ResponseWriter; to intercept the writes and capture metadata. I've done this in my performance logger to capture the status code and the length of bytes. However, since there are two ways I could have done this, I want to highlight the drawbacks on doing it the "wrong" way.... compare these two struct declarations:

### Example 1 (Bad)

``` go
type loggingResponseWriter struct {
  w http.ResponseWriter
  statusCode int
  length     int
}
```

### Example 2 (Good)

``` go
type loggingResponseWriter struct {
  http.ResponseWriter
  statusCode int
  length     int
}
```

Did you catch the difference? in ex 1, by not embedding the type, your struct assumes all responsibility for delegating to the wrapped object. The compiler will complain about hard dependencies, but in some cases there are runtime expectations that are evaluated dynamically (via reflection).

In this case, the ResponseWriter interface is only required to have three methods. But in practice, it can respond to other interfaces as well, for example http.Pusher. Since the compiler doesn't complain about methods not described by the interface; in ex 1, if we simply implement the minimum, we will accidentally disable HTTP/2 support, which would be very surprising and hard to debug.

To ensure you don't subsume runtime behavior, always ensure you embed your Decorated types. (note: you can access the fields/functions of the embedded instance by its type name.

### Full listing

``` go
type loggingResponseWriter struct {
  http.ResponseWriter
  statusCode int
  length     int
}

func newLoggingResponseWriter(w http.ResponseWriter) *loggingResponseWriter {
  return &loggingResponseWriter{
    ResponseWriter: w,
    statusCode:     http.StatusOK,
    length:         0}
}

func (l *loggingResponseWriter) WriteHeader(code int) {
  l.statusCode = code
  l.ResponseWriter.WriteHeader(code)
}

func (l *loggingResponseWriter) Write(b []byte) (int, error) {
  n, err := l.ResponseWriter.Write(b)
  l.length += n
  return n, err
}
