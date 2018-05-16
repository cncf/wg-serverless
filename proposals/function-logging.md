# Common function logging, observing, and monitoring 

Functions generate logs which are stored in the underline platform (e.g. Kubernetes logs, AWS Cloud watch, Azure App insight, elastic search..). each serverless platform has its own way of writing to a log. If we had a common way/api for logging it could have made functions portable AND allow simple integration between log services and function platform providers.

Nuclio as an example adopted Ubers' ZAP (https://github.com/uber-go/zap) interface which has great performance, support both structured and unstructured logs, and log hierarchies. 
Having such an abstraction also allowed us to integrate with multiple log services (HTTP, Console, Kubernetes, iguazio CDP, Azure App insights) w/o changing the function implementation.

Logger interface example (in Go):
```go
type Logger interface {

    // emit a log entry of a given verbosity. the first argument may be an object, a string
    // or a format string. in case of the latter, the following varargs are passed
    // to a formatter (e.g. fmt.Sprintf)

    // Error emits an unstructured error log
    Error(format interface{}, vars ...interface{})
    // Warn emits an unstructured warning log
    Warn(format interface{}, vars ...interface{})
    // Info emits an unstructured informational log
    Info(format interface{}, vars ...interface{})
    // Debug emits an unstructured debug log
    Debug(format interface{}, vars ...interface{})

    // emit a structured log entry. example:
    //
    // l.InfoWith("The message",
    //  "first-key", "first-value",
    //  "second-key", 2)
    //

    // ErrorWith emits a structured error log
    ErrorWith(format interface{}, vars ...interface{})
    // WarnWith emits a structured warning log
    WarnWith(format interface{}, vars ...interface{})
    // InfoWith emits a structured info loglog
    InfoWith(format interface{}, vars ...interface{})
    // DebugWith emits a structured debug log
    DebugWith(format interface{}, vars ...interface{})
    // Flush flushes buffered logs, if applicable
    Flush()

    // GetChild returns a child logger, if underlying logger supports hierarchal logging
    GetChild(name string) Logger
}
```

In addition to logs standardizing and integrating other APIs for custom metrics counting and tracing can simplify developer work. 
