# Native JavaScript DOM


## Scenario
---
> 利用 ```原生javascript``` 操作 DOM。

> 所以當有需求要用 原生 js 去 操作 DOM 時，不會希望自己在操作時，產生與 virtual dom 的交互效應 (不然就乖乖用 react 的規則就好了)，根據以上條件最快最適合的 React built-in 應該非 useRef 莫屬了。

```typescript
function RootComponent() {
  const ref = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    const element = ref.current;

    element?.addEventsListener(EVENT, ...);

    return () => {
      // 資料清除的時候無法保證 ref.current 全等於 addEventListener 時的實例
      // 所以要先確認是否有 event 可以解除
      element?.removeEventsListener(EVENT, ...);
    };
  }, []);
  return ...
}
```

## Hint
---
> useRef 目前使用上最主要除了拿來當 DOM ref，其他用途會使用到大多都是因為被放在```「useEffect」```底下的靜態參數。