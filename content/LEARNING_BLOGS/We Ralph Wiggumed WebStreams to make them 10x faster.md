- [link](https://vercel.com/blog/we-ralph-wiggumed-webstreams-to-make-them-10x-faster)

# MY FINDINGS
- The blog was written by the vercel CTO MatleUbl
- They noticed the framework overhead during profiling flamegraphs and theo's benchmark video
- This blog highlights the performance they added by working with webstreams which is becoming a standard in nodejs as per the mentioned PR
- The problem is first based on the difference in node.js 's old streaming API and WHATWG streams API , the second is the web standard or browser standard , the problem is that even when the data is present in buffer already and has to be read , it needs to go through four promise allocations and a microtask hop or if PipeTo is used it goes through the entire promise chain , this is not a flawed process as the web requires this level of security boundries  assumption and that we cannot control both sides involved in the process . Meaning when we have to push server components from the server constantly it becomes a huge performance backlog .

```
We benchmarked native WebStream `pipeThrough` at **630 MB/s** for 1KB chunks. Node.js `pipeline()` with the same passthrough transform: **~7,900 MB/s**. That is a 12x gap, and the difference is almost entirely Promise and object allocation overhead.
```

- Vercel 's solution was to build a performant lib , WHATWG webstreams compliant powered internally by node.js native streams .
- One was form of batching in which we can make multiple streams but when pipeTo an pipeThrough are called , they are called for the collective batch instead for every chunk , this bring the result: **~6,200 MB/s**. That is ~10x faster than native WebStreams and close to raw Node.js pipeline performance.
- A way to work around Reading buffer issues mentioned above , through synchronous call of nodeReable.node() , skips event loop processing entirely , so instant pulling of data , if not present in the buffer then we fallback to normal process 
- Now a important point in response bodies fetching , remember that every operation in the process is async meaning a promise callback is involved , so the native webstream creates a promise for every transform present and this is quite slow , what the lib solved was that it would not start promise when .transform is used it first identifies them , then chain link them and executes it using the node.js pipeline() in one go , reducing the number of promises to deal with .
- For react flight pattern they have created a reusable stream with better lite-weight buffer making construction cost of stream less .
- Props Matteo Collina for the node.js patch ,... should be out by now ig , i am late to this .
- Their is mention of some interesting lessons leanrt which i cannot comprehend due to my limited idea about stream and piping mechanism in node.js and browsers ... so we come back one day .
- Interesting read about streams , their is a whole network stack hidden when working with browsers ig .... , refer to the original blog for more details on benchmarks and stuff i am too dumb to tell yet , vercel 's blogs too good not to peek at .....