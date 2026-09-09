# SignPost Integration

This package creates `os_signpost` `begin` and `end` calls when spans are started or ended. It allows automatic integration of applications
instrumented with opentelemetry to show their spans in a profiling app like `Instruments`. It also exports the `OSLog` it uses for posting so the user can add extra signpost events. This functionality is shown in `Simple Exporter` example

## Version Notice

- **iOS 15+, macOS 12+, tvOS 15+, watchOS 8+**:  
  Use **`OSSignposterIntegration`**, which utilizes the modern `OSSignposter` API for improved efficiency and compatibility.
- **Older systems**:  
  Use **`SignPostIntegration`**, which relies on the traditional `os_signpost` API.

## Usage 

Add the appropriate span processor to your `TracerProviderSdk`, based on your deployment target:

```swift
let tracerProvider = TracerProviderSdk()
```

### For iOS 15+, macOS 12+, tvOS 15+, watchOS 8+:

```swift
tracerProvider.addSpanProcessor(OSSignposterIntegration())
```

### For older systems

```swift
tracerProvider.addSpanProcessor(SignPostIntegration())
```

### Or, to select automatically at runtime:

```swift
if #available(iOS 15, macOS 12, tvOS 15, watchOS 8, *) {
    tracerProvider.addSpanProcessor(OSSignposterIntegration())
} else {
    tracerProvider.addSpanProcessor(SignPostIntegration())
}
```

Then register the provider with `OpenTelemetry.registerTracerProvider(tracerProvider: tracerProvider)`.

### Custom logs and MetricKit

Both processors accept an `OSLog` through `init(log:)`. The zero-argument initializers still use
the `OpenTelemetry` subsystem and `.pointsOfInterest` category. Passing `OSLog.disabled`
disables signpost output without disabling span export.

On iOS, you can pass a log created by
[`MXMetricManager.makeLogHandle(category:)`](https://developer.apple.com/documentation/metrickit/mxmetricmanager/makeloghandle(category:)):

```swift
import MetricKit
import OpenTelemetryApi
import OpenTelemetrySdk
import SignPostIntegration

let tracerProvider = TracerProviderSdk()
let log = MXMetricManager.makeLogHandle(category: "OpenTelemetrySpans")
if #available(iOS 15, *) {
    tracerProvider.addSpanProcessor(OSSignposterIntegration(log: log))
} else {
    tracerProvider.addSpanProcessor(SignPostIntegration(log: log))
}
OpenTelemetry.registerTracerProvider(tracerProvider: tracerProvider)
```

This sends span intervals to the supplied log; it does not subscribe to MetricKit payloads.
MetricKit resource measurements such as CPU time, memory usage, and logical writes require
[`mxSignpost`](https://developer.apple.com/documentation/metrickit/monitoring-app-performance-with-metrickit),
which these processors do not call. Passing a MetricKit log alone does not populate those measurements.
