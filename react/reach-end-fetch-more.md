# Reach End Fetch More


## Scenario
---
> 滾動觸發 fetchMore 的用法，```避免在快速滾動時，連續觸發 fetchMore```，所以會利用 useRef 來作為靜態參數（loading, reach end ... etc）阻止 function 被連續呼叫。


```typescript
function Component() {
  // 預設一開始是允許拉資料的
  const allowedToFetchMore = useRef<boolean>(true);

  // 進行資料拉取的 callback
  const { data, fetchMore } = useMutation(...);

  // 監聽滾軸是否達到底部
  const hasReachEnd: boolean = useReachEnd();

  useEffect(() => {
    if (hasReachEnd && allowedToFetchMore.current) {
      // 呼叫 callback 權限 先關起來，防止快速滾動連續觸發 fetchMore callback後，再執行拉取。
      allowedToFetchMore.current = false;

      fetchMore()
        .then((response) => {
          // else state handle logic
          ...

          allowedToFetchMore.current = true;
        })
    }
  }, [hasReachEnd]);
}
```
