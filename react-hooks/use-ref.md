# useRef

## Basic Usage
---
```typescript
import { useRef } from 'react';

function ExampleComponent() {
  const renderTimesCounter = useRef(0);

  renderTimesCounter.current += 1;
}
```

## Scenario
---
> 當你想在某個 React Component 中單純使用一個 ```變數```，而且可以允許你 ```同步重新賦予新的數值```，就非常適合使用。
* 優點
  - 具有不因為 update 元件而被改變 reference 的特性。
  - 不會觸發 Virtual DOM 重新渲染，是效能優化的大幫手。
  - 同步賦值，也就是他是瞬間完成 數值更新 -> 很好用做資料操作的暫時容器。
* 缺點
  - 必須從 .current 使用/更新 數值，會讓程式碼變得比較不簡潔俐落。
  - 不會觸發重新渲染，使用時要特別注意，不然會有元件無法更新的狀況。

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

### Variants A

> 當 渲染資料 可能需要頻繁更新 (looping)，如果直接操作同一個 state，可能造成短時間內觸發太多次渲染，進而出現資料更新不全的情況下。

> 可以考慮 使用 useRef 來作為該元件中的 ```中心資料來源```。
- state 只管從 ref.current 那邊拿資料 (並且觸發渲染)
- ref 就是專門負責所有資料高頻更新。

```typescript
function FilesInput() {
  const filesTemp = useRef([]);
  const [filesToRender, setFilesToRender] = useState([]);

  function handleFilesUpdate(files: File[]) {
    if (!Array.isArray(files) || !files.length) return;

    // 高頻同步資料交給 filesTemp 
    files.forEach((file) => {
      const uniqueVirtualDomKey = getUniqueVirtualDomKey() // some logic to get unique render key

      filesTemp.current.push({
        key: uniqueVirtualDomKey,
        file,
      });
    });

    // 待 filesTemp 同步完最新資料後，統一渲染 '一' 次
    setFilesToRender(Array.from(filesTemp.current));

    // 至於為什麼要再 new Array 一次是因為，如果直接 set filesTemp.current，會使得每次 state 更新都指向 filesTemp.current 的 reference 使得 state 不會被觸發重新渲染，所以需要把最新渲染資料放進不同的 reference 同時給予對應的 uniqueVirtualDomKey，可以保證原先的 component 不會完全因為 reference 不同而重新渲染。
  }

  return (
    <div>
      <input
        onChange={e => handleFilesUpdate(Array.from(e.target.files))} />

      {filesToRender.map(fileInfo) => (
        <FileInfoCard
          key={fileInfo.id} // 非常重要
          file={fileInfo.file} />
      )}
    </div>
  );
}
```
> ### 專案實作 Implementation
 - [ITRI 圖片上傳狀態顯示列](https://itri-ai-dashboard.rytass.info/)
   - 分批上傳，觸發的 re-render 不會阻斷不同批次的元件狀態。
   - 一次上傳，只會觸發一次 re-render。
  - [ITRI 實現程式碼](https://github.com/Rytass/ITRI-AI-DashBoard/blob/main/client/src/core/input-field/components/image-input/index.jsx)

---

### Variants B
> 利用 ```原生javascript``` 操作 DOM。

> 如上述所說，其實 useRef 就是在 React Component 生命週期中，略過 virtual dom 直接使用普通變數。
所以當有需求要用 原生 js 去 操作 DOM 時，不會希望你在操作時，產生 與 virtual dom 的交互效應 (不然就乖乖用 react 的規則就好了)，所以最快想到應該就是非 useRef 莫屬了。

```typescript
function RootComponent() {
  const ref = useRef<HTMLDivElement | null>(null);

  // React.RefObject 無視 Virtual DOM 規則，所以無需放進 side-effect dependencyList
  useEffect(() => {
    ref.current.addEventsListener('click', ...);

    return () => {
      ref.current.removeEventsListener('click', ...);
    };
  }, []);
  // 其實直接用 onClick 比較好啦，但如果有奇怪的需求不能用，有可能就會用到

  return (
    <div ref={ref}>
      <ChildComponent loading={loading.current}>
    </div>
  );
}
```

### Variants C
> 避免 useEffect 在建立元素時被執行

> 某些狀況下，會希望元件生命週期剛被建立時，不要做某些事，直到某些條件符合後，再開始做。

```typescript
function RootComponent(props) {
  const { listeningInfo } = props;
  const hasInit = useRef(false);

  // React.RefObject 無視 Virtual DOM 規則，所以無需放進 side-effect dependencyList
  useEffect(() => {
    if (hasInit.current) {
      // do something to  if has init 
    } else {
      hasInit.current = true;
    }
  }, [listeningInfo]);

  return ...;
}
```