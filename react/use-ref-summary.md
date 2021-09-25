# useRef

## Basic Usage
---
```typescript
import { useRef } from 'react';

function ExampleComponent() {
  const renderTimesCounter = useRef(0);

  renderTimesCounter.current += 1;

  console.log('元件渲染次數：', renderTimesCounter.current);
}
```

## Scenario
---
> 當你想在某個 React Component 中使用一個 ```靜態變數```，或是在操作 DOM 上 ```防止重複渲染```、```重新呼叫函式```，就非常適合使用。
* 優點
  - 具有不因為 update 元件而被改變 reference 的特性。
  - 不會觸發 Virtual DOM 重新渲染，是效能優化的大幫手。
  - 同步賦值，也就是他是瞬間完成 數值更新 -> 很好用做資料操作的暫時容器。
* 缺點
  - 必須從 .current 使用/更新 數值，會讓程式碼變得比較不簡潔俐落。
  - 不會觸發重新渲染，使用時要特別注意，不然會有元件無法更新的狀態。

## Bad Pattern
---
> ### 錯誤的使用 useRef 作為 props 向子元件傳。

```typescript
function RootComponent() {
  const loading = useRef(false);

  return (
    <div>
      // 就算改變 loading.current = true
      // 子元件也不會知道。
      <ChildComponent loading={loading.current}>
    </div>
  );
}
```

## Correct Pattern

---

- [native-js-dom](./native-js-dom.md)
- [reach-end-fetch-more](./reach-end-fetch-more.md)