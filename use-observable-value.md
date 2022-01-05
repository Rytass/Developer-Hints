# use-observable-value (Abstract)

## 關鍵字 (key)
---
- 底層元件 -> 狀態更新
- 大量元件 -> 減少重新渲染
- 前台效能優化

## 範例 (Implementation)
---
- use-selector (Sportsbook)

- ![Image](https://media.giphy.com/media/l7xx2FRg6oULswYrqt/giphy.gif)

## 情境 (Scenario)
---
> 當前台需要 ```大量``` 子元件狀態監聽時。

1. 如果把狀態監聽寫在較高的層級，當有更新時，會導致不需要更新的子元件，因爲其他元件更新而重新渲染。

2. 在小規模的資料結構上面，或許這是可以接受的。畢竟效能過剩時，是完全沒有差別的，然而當規模提昇至 百、千 個元件時，這樣的更新，會大大影響效能，輕則耗電飆高，重則卡當影響使用體驗。

> 想法：

1. 較高層級的資料結構只需要，把所有的子元件 onMount 出來，然後每個子元件在 生命週期建立時，透過 ```EventEmitter``` 進行狀態監聽以及通知，透過 ```Observable``` 進行狀態更新時的行為。

2. ```EventEmitter``` 負責監聽狀態，當對應的子元件有狀態變動，可以透過 ```EventEmitter``` 主動通知 ```Observable``` 去更新重渲染對到的元件。因此每個 子元件 都有獨立的狀態 scope，不需要仰賴高層級的元件重繪，達成低層級的狀態更新。

> 優劣分析：

* 優點
  - 可以達成效能極大優化。
  - 每個小元件的更新，不會影響到其他同層級的元件重新渲染。
  - 效能 效能 效能。
* 缺點
  - 需要自己寫資料結構，控制每個元件的更新，變得比較麻煩。
  - 程式碼變多，維護成本提高。
  - 需要學習使用 ```rxjs```, ```event-emitter```。

## 基本資料結構建立
---
- ```EventEmitterStore (class)```
```typescript
import { EventEmitter } from 'eventemitter3';
import { Observable } from 'rxjs';

export class SelectorStore<T = boolean, R = Error> extends EventEmitter {
  static Events = {
    UPDATE: 'E/UPDATE',
    FAILED: 'E/FAILED',
  };

  private _observableMap: Map<string, Observable<T>> = new Map();

  private _selectorStore: Map<string, T> = new Map();

  /** @Method 取得對應監聽中元件的狀態 (值) */
  get(requestKey: string) {
    return this._selectorStore.get(requestKey);
  }

  /** @Method 更新對應監聽元件的狀態，這是 public setter */
  add(requestKey: string, data: T) {
    this._selectorStore.set(requestKey, data);

    /** @INFO 觸發 observable 去更新重新渲染對應 requestKey 的 訂閱元件 */
    this.emit(`${SelectorStore.Events.UPDATE}:${requestKey}`, data);
  }

  /** @Method 監聽對應的 key -> 回傳一個 observable */
  watch(requestKey: string) {
    const storedObservable = this.observableMap.get(requestKey);

    const observable = storedObservable || new Observable<T>((subscriber) => {
      /** @INFO 更新時的行為函式 */
      function onUpdate(selected: T) {
        subscriber.next(selected);
      }

      function onError(ex: R) {
        subscriber.error(ex);
      }

      // ... 可以自定義更多的監聽行為

      /** @INFO 註冊更新時要觸發的行為 */
      this.on(`${SelectorStore.Events.UPDATE}:${requestKey}`, onUpdate);

      this.on(`${SelectorStore.Events.FAILED}:${requestKey}`, onError);

      // ...
    });

    if (!storedObservable) {
      this._selectorObservableMap.set(requestKey, observable);
    }

    return observable;
  }
}

/** @INFO EventEmitterStore is a singleton */
export const selectorStore = new SelectorStore();
```

## React Custom Hook
---
> 示範子元件是否被選到。

- ```useIsSelected```
```typescript
export function useIsSelected(selectorId: string) {
  const [isSelected, setIsSelected] = useState<boolean>(selectorStore.get(selectorId) ?? false);

  useEffect(() => {
    const subscription = selectorStore
      .watch(selectorId) // 訂閱監聽
      .subscribe({ // observable 註冊更新行為
        next(nextData) { // 觸發定義的 更新行為
          setIsSelected(nextData);
        },

        // error ...etc
      });

    return () => {
      subscription.unsubscribe(); // componentWillUnMount 要取消狀態訂閱狀態，以免發生 memory leak 的狀況。
    };
  }, [selectorId]);

  return isSelected;
}

```

## References

---

- [rxjs - Observable](https://rxjs.dev/guide/observable)
- [eventemitter3](https://github.com/primus/eventemitter3#readme)