### Open telemetry practises for images

A open telemetry collector is run up and in the cluster tests to prove that the open-tel SDK is used correctly in the
example-http-service application (at least in terms of the results). This section is capturing our
current understanding of the best open telemetry setup from an images perspective.

- Use auto instrumentation as primary means of tracing. This seperates concerns, the code doesn't need to add debugging all over the place.
- Add the auto instrumentation in a way that is seperated from the code proper. You can see that the instrumentation.ts is not known to example-http-server.ts
- Let the collector configuration worry about sampling and filtering and where the things go. We are not considering that here
- New logs should be in a structured json format. Use a tool to do this. The otel collector will pull these in and handle the filtering etc.
- As for metrics and any other open-tel signals. When we need them in practise we can add them to this example.
