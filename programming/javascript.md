# Javascript

## Promise

### Promise.all 中如果有执行出错会怎样？ <!-- {docsify-ignore} -->

    > Promise.all([1,2,3,4,5].map(async(i)=>{ if(i==3){throw new Error('xxxx')}else{console.log(i);return i}})).then(res=>console.log(res)).catch(err=>console.log(err))
    1
    2
    4
    5
    Promise { <pending> }
    > Error: xxxx
        at REPL11:1:56
        at Array.map (<anonymous>)
        at REPL11:1:25
        at Script.runInThisContext (vm.js:133:18)
        at REPLServer.defaultEval (repl.js:484:29)
        at bound (domain.js:413:15)
        at REPLServer.runBound [as eval] (domain.js:424:12)
        at REPLServer.onLine (repl.js:817:10)
        at REPLServer.emit (events.js:327:22)
        at REPLServer.EventEmitter.emit (domain.js:467:12)

每个都会执行，最终抛被 catch