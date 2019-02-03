#Async类
[https://caolan.github.io/async/](https://caolan.github.io/async/)
Async.js是一个强大的类库，用来处理复杂的异步操作
Async.js可以简化和组织我们的多个异步函数
动态页面环境中，也直接暴露了Async类

下面我们列出几个常用的方法：

1. <span id="Async.series">**Async.series**</span>: 串行执行一个个异步任务
```
/**
 * 串行执行任务数组
 * @param tasks {Array | Iterable | Object} 异步任务数组
 * @param [callback] {function} 全部任务执行完后的回调
 */
series(tasks, callback)
```
示例：
```
Async.series([
    function(callback) {
        // do some stuff ...
        callback(null, 'one');
    },
    function(callback) {
        // do some more stuff ...
        callback(null, 'two');
    }
],
// optional callback
function(err, results) {
    // results is now equal to ['one', 'two']
});
```

1. <span id="Async.parallel">**Async.parallel**</span>: 并行执行一个个异步任务
```
/**
 * 并行执行一个个异步任务
 * @param tasks {Array | Iterable | Object} 异步任务数组
 * @param [callback] {function} 全部任务执行完后的回调
 */
parallel(tasks, callback)
```
示例：
```
Async.parallel([
    function(callback) {
        setTimeout(function() {
            callback(null, 'one');
        }, 200);
    },
    function(callback) {
        setTimeout(function() {
            callback(null, 'two');
        }, 100);
    }
],
// optional callback
function(err, results) {
    // the results array will equal ['one','two'] even though
    // the second function had a shorter timeout.
});
```

1. <span id="Async.waterfall">**Async.waterfall**</span>: 瀑布式执行一个个异步任务，上一个异步任务的结果可以传递到下一个任务中去
```
/**
 * 串行执行异步任务，每个任务结果都会传递到下一个任务中
 * @param tasks {Array | Iterable | Object} 异步任务数组
 * @param [callback] {function} 全部任务执行完后的回调
 */
waterfall(tasks, callback)
```
示例：
```
async.waterfall([
    function(callback) {
        callback(null, 'one', 'two');
    },
    function(arg1, arg2, callback) {
        // arg1 now equals 'one' and arg2 now equals 'two'
        callback(null, 'three');
    },
    function(arg1, callback) {
        // arg1 now equals 'three'
        callback(null, 'done');
    }
], function (err, result) {
    // result now equals 'done'
});
```

1. <span id="Async.each">**Async.each**</span>: 遍历任务数组，并行执行这些任务
```
/**
 * 遍历数组，对每一个数据并行执行异步任务
 * @param coll {Array | Iterable | Object} 遍历的对象
 * @param iteratee {function} 异步任务
 * @param [callback] {function} 全部任务执行完后的回调
 */
each(coll, iteratee, callback)
```
示例：
```
// assuming openFiles is an array of file names
async.each(openFiles, function(file, callback) {

    // Perform operation on file here.
    console.log('Processing file ' + file);

    if( file.length > 32 ) {
      console.log('This file name is too long');
      callback('File name too long');
    } else {
      // Do work to process file here
      console.log('File processed');
      callback();
    }
}, function(err) {
    // if any of the file processing produced an error, err would equal that error
    if( err ) {
      // One of the iterations produced an error.
      // All processing will now stop.
      console.log('A file failed to process');
    } else {
      console.log('All files have been processed successfully');
    }
});
```

1. <span id="Async.eachSeries">**Async.eachSeries**</span>: 遍历任务数组，串行执行这些任务
```
/**
 * 遍历数组，对每一个数据串行执行异步任务
 * @param coll {Array | Iterable | Object} 遍历的对象
 * @param iteratee {function} 异步任务
 * @param [callback] {function} 全部任务执行完后的回调
 */
eachSeries(coll, iteratee, callback)
```
