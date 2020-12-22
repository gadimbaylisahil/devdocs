# Generic and Rails specific performance notes

Most of the notes in this document is from [Complete Guide to Rails Performance](https://www.railsspeed.com/) by Nate Berkopec.

## PROFILE flag for development.rb

Instead of running an application on production mode, we can add the PROFILE flag in development environment config and imitate production config.

See this [commit](https://github.com/rubygems/rubygems.org/pull/2148)

> Do not optimize, unless metrics tells you so...

## Micro-benchmarking with benchmark/ips

```ruby
require "benchmark/ips"
        
class TestBench
  def methods
    @methods ||= ("a".."z").to_a
  end

  def fast
    methods.shuffle
  end

  def slow
    max = methods.size
    methods.sort.sort_by { rand max }
  end
end
        
test = TestBench.new
        
Benchmark.ips do |x|
  x.report("fast") { test.fast }
  x.report("slow") { test.slow }
  x.compare!
end
```

Don't get married to micro-benchmark results, the bigger picture lies in profiling and macro benchmarking.

## Performance checklist & communication

- Quantiy performance in business metrics, aka money

- Use [webpagetest](Webpagetest.org) to measure and set a front-end load time budget

- Set MART and M95RT(Maximum 95th percentile response time) for server responses. It's important to know what's going on in the long tail and on average.

- Set page weight budgets

- Quantify integration costs - When adding 3rd party integrations such as scripts, quantify it's implementations drawbacks in traffic/monetary loss and act accordingly

- Add automated page weight tests inside CI pipelines

- Set aggressive timeouts on 3rd party integrations & be defensive about the downtime of the integrations

## Performance workflow

Profiler SASS such as NewRelic, Datadog, Appsignal is great for past data. We should focus on profiling in development as well to view changes we make actually mattered. If we just rely on APM SASS services, we would need to `wait` for production data to give us insights.

1. Monitor metrics; Is there anything out of ordinary or exceeds product's performanc budgets
2. Profile and figure out where time is spent
3. Experiment with benchmarks; Does removing that part which we spent 50% time on, really reduces the overall execution? Re-benchmark...

## Ruby-prof

[Ruby-prof](https://github.com/ruby-prof/ruby-prof) works by hooking itself into MRI directly and performing profiling each call in the stack. When running ruby-prof expect code to be run 2-3x slower as it's an intense process.

Thus, it's not suitable for profiling in production but is great for development.

benchmark/ips can give you an idea(`WHAT`) for the comparison of the performance of different code executions, ruby-prof in turn, can get you details on execution steps and their individual execution performance(`WHY`).

Example for quick'n'dirty profiling with ruby-prof.

```ruby
require 'ruby-prof'

RubyProf.measure_mode = RubyProf::CPU_TIME

profile = RubyProf.profile do 
  1_000_000.times do { some_code_to_profile }
end

printer = RubyProf::FlatPrinter.new(profile)
printer.print(STDOUT)
```

Check the results and go all the way from top to bottom by improving performance incrementally, until satisfied.

## Stackprof

Unline ruby-prof, [stackprof](https://github.com/tmm1/stackprof) uses sampling technique instead of stack traces to measure performance. This makes it suitable for production.

### Tracing vs Sampling

In tracing profiler such as ruby-prof, profiler hooks into every method invocation and tries to measure the execution time, which makes it accurate but slow.

In sampling profiler not every occurrence is measured, but every N occurrence of the event, thus, not resulting in performance issues in profiler itself.

> Use profiler such as ruby-prof to test the boot time of your application. Where does the time go?

## Rack mini profiler

[Rack mini profiler](https://github.com/MiniProfiler/rack-mini-profiler) is a performance tool for rack applications.

Provides entire suite of performance tools for rack based ruby web applications which is also suitable to run on `production`.

Run application in production environment settings with production like data size.

You can also activate rack-mini-profiler in production with a call-back in controllers such as:

```ruby
before_filter :check_rack_mini_profiler
def check_rack_mini_profiler
  # for example - if current_user.admin?
  if params[:rmp]
    Rack::MiniProfiler.authorize_request
  end
end
```

It's also better not to use default disk storage for rack-mini-profiler and use `in-memory` storage instead.

`Rack::MiniProfiler.config.storage = Rack::MiniProfiler::MemoryStore`

rack-mini-profiler adds a speed badge to the page from which we can view the total spent time on partials, queries etc...

### Flamegraphs

Add `?pp=flamegrahp` to see flamegrahps.

Works alongside with `flamegraph` gem.

### GC stats

Add `?pp=profile-gc` to enable memory profiling. Output is the output of `GC.stat`.

Watch out for pages that generates abnormally high values for memory(10MB+). You can detailed string allocations. Perhaps, look and froze strings that get allocated 100s, 1000s of times...

See [commit](https://github.com/rack/rack/commit/dc53a8c26dc55d21240233b3d83d36efdef6e924)

### Profile memory

Add `?pp=profile-memory`. This is a `premium` version of `profile-gc`. Instead of telling us what strings were allocated, it tells us `where`.

It works alongside with `memory_profiler` gem.

## Exceptions for control flow

Exceptions in ruby are [32X slower](https://simonecarletti.com/blog/2010/01/how-slow-are-ruby-exceptions/)

## Memory Profiling

Memory profiling tools are usually bound to the VM the language operates in. For the CPU profiling, generic tools such as Google's perftools can be used.

Several tools for ruby to profile memory:

- `ObjectSpace` module
- `GC::Profiler` module in stdlib
- [gc_tracer](https://github.com/ko1/gc_tracer)
- [derailed_benchmarks](https://github.com/schneems/derailed_benchmarks)
- [memory_profiler](https://github.com/SamSaffron/memory_profiler)

### ObjectSpace

```ruby
ObjectSpace.count_objects

{
 :TOTAL=>132882,
 :FREE=>49002,
 :T_OBJECT=>10011,
 :T_CLASS=>2010,
 :T_MODULE=>145,
 :T_FLOAT=>4,
 :T_STRING=>33066,
 :T_REGEXP=>704,
 :T_ARRAY=>17848,
 :T_HASH=>1000,
 :T_STRUCT=>39,
 :T_BIGNUM=>2,
 :T_FILE=>4,
 :T_DATA=>1467,
 :T_MATCH=>2,
 :T_COMPLEX=>1,
 :T_RATIONAL=>1,
 :T_SYMBOL=>49,
 :T_IMEMO=>17234,
 :T_ICLASS=>293
}
```

You can optionally `require 'objspace'`(on development only, as it will slow down your app due to tracing).

This will add some convenience methods such as:

```ruby
# Shows how much memory each object type is using
ObjectSpace.count_objects_size

# Displays how much memory used by [1000x...:a]
ObjectSpace.memsize_of(Array.new(1000) { :a })

# Shows memory used by specific class
ObjectSpace.memsize_of_all(String)
```

Use object space to enhance knowledge in ruby memory usage, build mini benchmark/profile tools that suit your needs.

### GC

```ruby
GC.disable

GC.enable

GC.start
```

#### GC Profiler

```ruby
GC::Profiler.enable

require 'set'

GC.start

GC::Profiler.report
GC::Profiler.disable
```

Invocation counts are since the process boot, not during the profiling state.

When tools points you towards GC running often or taking long time, use profilers.

### GC Tracer

An extension to GC::Profiler, which can log the memory usage into files/html, hooked into Rack middleware.

```ruby
gem 'gc_tracer', require: 'rack/gc_tracer'

Rails.application.middleware.use Rack::GCTracerMiddleware, view_page: '/gc_tracer', filename: 'log/gc_tracer'

# Custom instrumentation

GC::Tracer.start_logging do
  some_awful_code
end
```

Prefer to use this in development to track down memory leaks.

### Derailed Benchmarks

Use derailed benchmarks to monitor boot time memory load of gems and memory leaks

### Memory Profiler

```ruby
require 'memory_profiler'

MemoryProfiler.report do 
  # some block of code
end.pretty_print

# Total allocated: 914156 bytes (8503 objects)
# Total retained:  46834 bytes (645 objects)
```

Not all memory statistics will be accurate. Due to memory fragmentation over time, `memory_profiler` will be underestimating versus what you might see in the process's memory usage using `ps`.

> Use memory_profiler when manually instrumenting certain piece of code, such as in background jobs. For the rest, prefer rack-mini-profiler or derailed

### Checklist

- Perform audit of the gemfile using derailed benchmarks, to asses boot time memory usage brought by dependencies
- Experiment with ObjectSpace and write your own logging code to identify hotspots for memory
- Use memory_profiler alongside with rack-mini-profiler to activate it's memory functionalities

## Benchmarks Folder

You may opt in to have a `bechmarks` folder in your application repo such as [jekyll does](https://github.com/jekyll/jekyll/tree/master/benchmark).

Also see [Minitest::Benchmark](https://chriskottom.com/blog/2015/04/minitest-benchmark-an-introduction/)

## Load testing tools

### WRK

```bash
wrk

HTTP benchmarking tool.
More information: https://github.com/wg/wrk.

- Run a benchmark for 30 seconds, using 12 threads, and keeping 400 HTTP connections open:
  wrk -t12 -c400 -d30s "http://127.0.0.1:8080/index.html"

- Run a benchmark with a custom header:
  wrk -t2 -c5 -d5s -H "Host: example.com" "http://example.com/index.html"

- Run a benchmark with a request timeout of 2 seconds:
  wrk -t2 -c5 -d5s --timeout 2s "http://example.com/index.html"
```

Look into [loader.io](https://loader.io)

