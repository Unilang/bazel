Project: /_project.yaml
Book: /_book.yaml

# Debugging Remote Cache Hits for Local Execution

{% include "_buttons.html" %}

This page describes how to investigate cache misses in the context of local
execution.

This page assumes that you have a build and/or test that successfully builds
locally and is set up to utilize remote caching, and that you want to ensure
that the remote cache is being effectively utilized.

For tips on how to check your cache hit rate and how to compare the execution
logs between two Bazel invocations, see
[Debugging Remote Cache Hits for Remote Execution](/docs/remote-execution-caching-debug).
Everything presented in that guide also applies to remote caching with local
execution. However, local execution presents some additional challenges.

## Checking your cache hit rate {:#cache-hits}

Successful remote cache hits will show up in the status line, similar to
[Cache Hits rate with Remote
Execution](/docs/remote-execution-caching-debug#check-cache-hits).

In the standard output of your Bazel run, you will see something like the
following:

```none {:.devsite-disable-click-to-copy}
   INFO: 7 processes: 3 remote cache hit, 4 linux-sandbox.
```

This means that out of 7 attempted actions, 3 got a remote cache hit and 4
actions did not have cache hits and were executed locally using `linux-sandbox`
strategy. Local cache hits are not included in this summary. If you are getting
0 processes (or a number lower than expected), run `bazel clean` followed by
your build/test command.

## Troubleshooting cache hits {:#troubleshooting-cache-hits}

If you are not getting the cache hit rate you are expecting, do the following:

### Ensure successful communication with the remote endpoint {:#remote-endpoint-success}

To ensure your build is successfully communicating with the remote cache, follow
the steps in this section.

1. Check your output for warnings

   With remote execution, a failure to talk to the remote endpoint would fail
   your build. On the other hand, a cacheable local build would not fail if it
   cannot cache. Check the output of your Bazel invocation for warnings, such
   as:

   ```none {:.devsite-disable-click-to-copy}
      WARNING: Error reading from the remote cache:
   ```


   or

   ```none {:.devsite-disable-click-to-copy}
      WARNING: Error writing to the remote cache:
   ```


   Such warnings will be followed by the error message detailing the connection
   problem that should help you debug: for example, mistyped endpoint name or
   incorrectly set credentials. Find and address any such errors. If the error
   message you see does not give you enough information, try adding
   `--verbose_failures`.

2. Follow the steps from [Troubleshooting cache hits for remote
   execution](/docs/remote-execution-caching-debug#troubleshooting_cache_hits) to
   ensure that your cache-writing Bazel invocations are able to get cache hits
   on the same machine and across machines.

3. Ensure your cache-reading Bazel invocations can get cache hits.

   a. Since cache-reading Bazel invocations will have a different command-line set
      up, take additional care to ensure that they are properly set up to
      communicate with the remote cache. Ensure the `--remote_cache` flag is set
      and there are no warnings in the output.

   b. Ensure your cache-reading Bazel invocations build the same targets as the
      cache-writing Bazel invocations.

   c. Follow the same steps as to [ensure caching across
      machines](/docs/remote-execution-caching-debug#caching-across-machines),
      to ensure caching from your cache-writing Bazel invocation to your
      cache-reading Bazel invocation.
