---
title: "Custom traces on App Engine Standard"
subtitle: "Sending custom traces to Stackdriver with OpenCensus"
date: 2019-02-12 00:28:01+08:00
---

[Bus Eta Bot](https://t.me/BusEtaBot) is a Telegram bot for getting bus arrival times from the LTA DataMall API. 

{{< figure src="bus-eta-bot.png" alt="Screenshot of a conversation with Bus Eta Bot on Telegram showing bus arrival times for a bus stop." >}}

It is written in Go and hosted on the App Engine standard environment for Go.

## Stackdriver tracing on App Engine standard

In the [App Engine standard environment](https://cloud.google.com/appengine/docs/standard/), applications run in a sandboxed environment and are subject to various restrictions and vendor lock-in. However, in return you do get tracing for free with [Stackdriver Trace](https://cloud.google.com/trace/docs/setup/#app_engine). For example, here's a trace of a typical request handled by Bus Eta Bot:

{{< figure src="trace1.png" alt="Stackdriver Trace screenshot showing requests made by Bus Eta Bot." >}}

Because Bus Eta Bot uses the App Engine specific [appengine/urlfetch](https://godoc.org/google.golang.org/appengine/urlfetch) and [appengine/datastore](https://godoc.org/google.golang.org/appengine/datastore) packages, outgoing HTTP requests and Cloud Datastore operations are automatically traced and can be viewed on the Cloud Console.

### Notes on the App Engine Go 1.11 Standard Environment

In the App Engine Go 1.11 Standard Environment, the standard library `net/http` HTTP client and generic Google Cloud client libraries can be used instead of their App Engine-specific counterparts. While doing so eliminates some inconveniences such as having to create a new HTTP client per request, it also loses the automatic Stackdriver Trace instrumentation.

## Adding custom traces to Stackdriver

Here's a trace which includes with a custom, non-App Engine span in the in the midst of all the others:

{{< figure src="trace2.png" alt="Stackdriver Trace screenshot showing a custom trace." >}}

Here, I've added a trace for the `InMemoryBusStopRepository.Nearby` method in my code. This method finds bus stops within a certain distance of a certain location by looping through a list of bus stops in memory, and I wanted to compare its performance to another implementation which used the [App Engine Search API](https://cloud.google.com/appengine/docs/standard/go/search/).

### Setting up OpenCensus

First, we need to set up a OpenCensus exporter for StackDriver, as documented in [Setting up Stackdriver Trace for Go](https://cloud.google.com/trace/docs/setup/go):

```go
// import "contrib.go.opencensus.io/exporter/stackdriver"
// import "go.opencensus.io/trace"

// Create and register a OpenCensus Stackdriver Trace exporter.
exporter, err := stackdriver.NewExporter(stackdriver.Options{})
if err != nil {
	log.Printf("error setting up opencensus stackdriver exporter: %+v\n", err)
} else {
	trace.RegisterExporter(exporter)
	trace.ApplyConfig(trace.Config{DefaultSampler: trace.AlwaysSample()})
}
```

The provided `stackdriver.Options` can be empty because there's no need to provide application credentials inside App Engine. Depending on the amount of traffic you're expecting, you can also configure OpenCensus to always sample traces to see results immediately (otherwise, not all traces will be recorded).

### Getting the correct parent span

The [`StartSpan`](https://godoc.org/go.opencensus.io/trace#StartSpan) method in the [`go.opencensus.io/trace`](https://godoc.org/go.opencensus.io/trace) package will start a new child span if its `context.Context` argument already contains a span. However, because we're integrating with the existing non-OpenCensus instrumentation, we can't simply do this. Instead, we need to create a parent span based on the information contained in the [`X-Cloud-Trace-Context` header](https://cloud.google.com/trace/docs/troubleshooting#force-trace) passed to an App Engine request. 

We can do this using the [`SpanContextFromRequest`](https://godoc.org/go.opencensus.io/exporter/stackdriver/propagation#HTTPFormat.SpanContextFromRequest) method on the `HTTPFormat` struct exported by the [`go.opencensus.io/exporter/stackdriver/propagation`](https://godoc.org/go.opencensus.io/exporter/stackdriver/propagation) package:

```go
// import "go.opencensus.io/exporter/stackdriver/propagation"

var HTTPFormat = propagation.HTTPFormat{}
parent, ok := HTTPFormat.SpanContextFromRequest(req)
```

### Creating a new child span

If we manage to extract a parent span, we can create a new child span using [`trace.StartSpanWithRemoteParent`](https://godoc.org/go.opencensus.io/trace#StartSpanWithRemoteParent) and use it normally:

```go
// import "go.opencensus.io/trace"
// import "go.opencensus.io/exporter/stackdriver/propagation"

var HTTPFormat = propagation.HTTPFormat{}
if parent, ok := HTTPFormat.SpanContextFromRequest(req); ok {
	_, span := trace.StartSpanWithRemoteParent(ctx, "InMemoryBusStopRepository/Nearby", parent)
	defer span.End()
}
```

You will be able to see your created spans together with the rest of the App Engine spans in Stackdriver Trace after the request is complete.

## Wrapping up

The code samples above are intentionally brief to focus on the essential methods. In practice, the functions you want to trace may not be directly handling the incoming request from App Engine, so you will need a way to pass the parent `trace.SpanContext` to the function you actually want to create the child span in. One way to do this is by putting it on a unique key in a `context.Context` you pass down the call chain:

```go
import (
	"context"
	"net/http"

	"contrib.go.opencensus.io/exporter/stackdriver/propagation"
	"go.opencensus.io/trace"
	"google.golang.org/appengine"
)

var HTTPFormat = propagation.HTTPFormat{}

type parentSpanKey struct{}

// newContext creates a new context from an incoming App Engine request to be passed through the application.
func newContext(r *http.Request) (ctx context.Context) {
	// create a new appengine context from the incoming request
	ctx = appengine.NewContext(r)
	if parent, ok := HTTPFormat.SpanContextFromRequest(r); ok {
		ctx = context.WithValue(ctx, parentSpanKey{}, parent)
	}
	return
}

// parentSpanFromContext extracts a *trace.SpanContext from ctx. 
func parentSpanFromContext(ctx context.Context) (*trace.SpanContext, bool) {
	parent, ok := ctx.Value(parentSpanKey{}).(*trace.SpanContext)
	return parent, ok
}

func doStuff() {}

func someTracedFunction(ctx context.Context) {
	if parent, ok := parentSpanFromContext(ctx); ok {
		_, span := trace.StartSpanWithRemoteParent(ctx, "someTracedFunction", parent)
		defer span.End()
	}
	doStuff()
}
```
