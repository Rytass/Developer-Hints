# Promise Waterfall

## Scenario

當有多個 Promise 需要依序完成，不可同時進行時

## Bad Pattern

### 錯誤的使用 Promise.all

Promise.all 提供的是並行 (Parallels) 執行，所以循序漸進時不可以使用這個 API

```typescript
const tasks: Promise[];

await Promise.all(tasks);
```

## Correct Pattern

不使用 Library 的前提下，可以透過 Array.reduce 的方式串起 Promise 並透過 Promise.then 讓其依序執行，並可使用 Promise.catch 在任意錯誤發生時中斷

```typescript
const tasks: Promise[];

await tasks
  .map((task) => () => task)
  .reduce((prev, next) => Promise.resolve());
```

### Variants A

一包需要循序執行非同步工作的資料

```typescript
const data = [1,2,3,4,5];

await data
  .map((element) => async () => {
    await someAsyncTaskNeedData(element);
  })
  .reduce((prev, next) => Promise.resolve());
```

### Variants B

一包需要循序執行非同步工作的資料，並需要回傳執行結果

> Notice: 需提供回傳 Type，並且因為 reduce 特性，可以同時實作篩選

```typescript
const data = [1,2,3,4,5];

await data
  .map((element) => async (results) => {
    const result = await someAsyncTaskNeedData(element);

    // If filter need
    // if (result !== condition) {
    //   return results
    // }

    return [
      ...results,
      result,
    ];
  })
  .reduce((prev, next) => Promise.resolve([] as ResultType[]));
```
