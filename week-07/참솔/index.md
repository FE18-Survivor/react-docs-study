# Week 07

## [Referencing Values with Refs](https://react.dev/learn/referencing-values-with-refs)

> **Summary**
>
> - Ref object는 `current` property를 갖는 mutable한 일반적인 JavaScript object로, React가 rendering에 사용하지 않으므로 값 변경을 추적하지 않음
> - Re-rendering 되어도 값을 유지하지만 값읇 변경해도(`ref.current` 변경) component를 re-render 하지 않을 때 사용
> - `useRef(initialValue)` hook으로 ref object 생성
> - Refs는 app 외부의 external system이나 Web APIs와 함께 사용하면 좋음
>   - `setTimeout` 사용 시 timer ID 저장
>   - JSX의 `ref` attribute에 전달하여 DOM element에 접근
> - Ref 객체를 rendering 중에 읽거나 쓰지 않도록 주의. Event handler 또는 `useEffect` callback 안에서 사용한다.

---

- `ref` : React가 추적하지 않는 mutable value
  - React의 단방향(one-way) data flow를 벗어남 (escape hatch)
  - `current` property를 갖는 일반적인 JavaScript object
  - Mutable value이므로 `current` 값을 직접 변경할 수 있음
  - Rendering 중에 `ref.current` 값을 읽으면 안됨 (event handler나 `useEffect` callback에서 접근)
- State와 비교
  - `ref`는 state처럼 re-render 간에 값이 유지되지만, **state와 달리 값을 변경해도 re-render를 trigger 하지 않음**
  - **Re-rendering을 trigger 하지 않으면서 component에 data를 저장하고 싶을 때** 사용
- `useRef(initialValue)` hook을 사용해서 `ref` object를 생성하고, `ref.current`로 값 사용

  ```javascript
  import { useRef } from "react";

  export default function Counter() {
    let ref = useRef(0);

    function handleClick() {
      ref.current = ref.current + 1;
      alert("You clicked " + ref.current + " times!");
    }

    return <button onClick={handleClick}>Click me!</button>;
  }
  ```

- State와 `ref`를 함께 사용하는 예시

  ```javascript
  import { useState, useRef } from "react";

  export default function Stopwatch() {
    // Rendering에 사용하는 값은 state 사용
    const [startTime, setStartTime] = useState(null);
    const [now, setNow] = useState(null);

    // 값 변경이 re-render를 필요로 하지 않는 경우 ref 사용
    const intervalRef = useRef(null);

    function handleStart() {
      setStartTime(Date.now());
      setNow(Date.now());

      clearInterval(intervalRef.current);
      intervalRef.current = setInterval(() => {
        setNow(Date.now());
      }, 10);
    }

    function handleStop() {
      clearInterval(intervalRef.current);
    }

    // 'Start' 버튼을 누를 때마다 re-render 되면서 값이 초기화될 것
    let secondsPassed = 0;
    if (startTime != null && now != null) {
      secondsPassed = (now - startTime) / 1000;
    }

    return (
      <>
        <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
        <button onClick={handleStart}>Start</button>
        <button onClick={handleStop}>Stop</button>
      </>
    );
  }
  ```

### How does useRef work inside?

- `useRef`는 `useState`를 기반으로 구현됨
  ```javascript
  function useRef(initialValue) {
    const [ref, unused] = useState({ current: initialValue });
    return ref;
  }
  ```
  - 첫 번째 render 동안 `useRef`는 **'state variable'** 인 `{ current: initialValue }` object를 반환
  - Object가 React에 의해 state로 저장되므로, next render에서 동일한 object가 반환됨
  - 이 때, `useRef`는 항상 같은 object를 반환하므로 setter 함수는 필요 없음 (`unused`)
- 즉, `ref`는 setter가 없는 regular state variable

### Best practices for refs

- Refs를 'escape hatch'로 다룬다.
  - Refs는 external system을 다루거나 browser API와 함께 사용할 때 유용하다.
  - Application logic과 data flow 등은 refs에 의존하지 않는게 좋다.
- `ref.current`를 rendering 중에 읽거나 쓰지 않는다.
  - Rendering 중에 필요한 data는 state로 관리한다.
  - React는 `ref.current`가 변경되는 것을 알지 못하므로, rendering 중에 `ref.current` 값을 읽으면 component가 예상치 못하게 동작할 수 있다.
  - 단, 첫 번째 render에서 한 번만 값을 쓰는 것은 가능하다.
    ```javascript
    // Ref 초깃값이 falsy일 때(`null` 또는 `undefined`) 첫 번쨰 render에서만 아래 구문 실행.
    // 두 번째 render 부터는 `ref.current`에 truthy 값이 할당되어 있음
    if (!ref.current) {
      ref.current = new Thing();
    }
    ```
- Every render 마다 snapshot 처럼 동작하는 state와 달리, ref는 변경하는 즉시 값이 반영된다.
  ```javascript
  ref.current = 5;
  console.log(ref.current); // 항상 5
  ```
- Ref object는 rendering에 사용되지 않기 때문에, React는 ref를 신경쓰지 않는다.

### Refs and the DOM

- Ref는 DOM element에 접근할 때 주로 사용됨
- JSX의 `ref` attribute에 ref object를 전달하면, React가 해당하는 DOM element를 `ref.current`에 할당해 줌
- DOM element가 제거되면 React가 `ref.current`를 `null`로 초기화

## [Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs)

> **Summary**
>
> - JSX element의 `ref` attribute에 `useRef`로 만든 ref object를 전달하면, React가 DOM node가 준비 되었을 때 `ref.current`에 DOM node를 할당해 준다.
> - 이 때, DOM node 할당은 'commit phase'에 이루어지므로 rendering 중간에 ref object에 접근해서 Web API를 호출하면 안 된다.
> - `Array.map()`을 사용해서 동적으로 rendering하는 list item들의 DOM node는 다른 방법을 사용해야 한다.
>   1. List item 들의 parent element에 `ref`를 설정해서 DOM node를 얻고, `querySelectorAll`을 사용해서 list item의 DOM node에 접근
>   2. `ref` attribute에 callback 함수를 전달해서 DOM node를 받고, 이것을 `Map` 자료구조로 관리 (React 19부터 지원하는 '**ref callback**' 사용)
> - Parent component가 child component 내부의 element에 대한 DOM node에 접근하고 싶다면 `ref` prop을 통해 ref object를 전달한다.
>   - 이 때, child component는 `useImperativeHandle` hook을 사용해서 parent에 노출(expose)할 interface를 직접 정의할 수 있다. (외부에서 접근할 수 있는 기능 제한)
> - Ref를 state와 함께 사용할 때 state와 sync가 맞지 않는 문제가 발생한다면 `flushSync` 함수를 사용해서 React가 state 변경을 동기적으로 즉시 반영하도록 강제할 수 있다.
> - Refs는 'escape hatch' 이므로 React의 rendering과 무관하게 동작한다. 따라서, **React가 추가/수정/삭제하는 element는 ref를 통해 DOM node를 직접 조작하지 않도록 주의**해야 한다.

---

- React는 render output과 일치하도록 DOM을 자동 update 해 주기 때문에 직접 DOM element를 다루는 일은 드물다.
- 하지만, node에 focus 하거나 size 및 position을 측정하려는 경우 **React에 의해 관리되는 DOM element에 직접 접근**해야 할 수도 있다.
- JSX의 `ref` attribute에 `useRef`가 반환한 ref object를 전달하면, React가 DOM node를 생성할 때 참조를 `ref.current`에 할당
- Mount and render 이후, ref 객체를 통해 DOM node에 접근하거나 API를 호출하면 안 된다.
  ```javascript
  ref.current.scrollIntoView(); // Use any browser APIs
  ```
- `ref`를 통해 `<input />`을 코드로 focus 시키는 예시

  ```javascript
  import { useRef } from "react";

  export default function Form() {
    // <input> element의 DOM node를 참조할 ref object 생성
    const inputRef = useRef(null);

    function handleClick() {
      // Event handler 실행 시점에는 rendering 및 `ref.current` 값 할당이 완료된 상태
      // `current`로 Input DOM node에 접근해서 Web API 사용
      inputRef.current.focus();
    }

    return (
      <>
        <input ref={inputRef} />
        <button onClick={handleClick}>Focus the input</button>
      </>
    );
  }
  ```

- `ref`를 통해 특정 element로 스크롤하는 예시

  ```javascript
  import { useRef } from "react";

  const SCROLL_OPTIONS_SMOOTH_CENTER = {
    behavior: "smooth",
    block: "nearest",
    inline: "center",
  };

  export default function CatFriends() {
    const firstCatRef = useRef(null);
    const secondCatRef = useRef(null);
    const thirdCatRef = useRef(null);

    function handleScrollToFirstCat() {
      firstCatRef.current.scrollIntoView(SCROLL_OPTIONS_SMOOTH_CENTER);
    }

    function handleScrollToSecondCat() {
      secondCatRef.current.scrollIntoView(SCROLL_OPTIONS_SMOOTH_CENTER);
    }

    function handleScrollToThirdCat() {
      thirdCatRef.current.scrollIntoView(SCROLL_OPTIONS_SMOOTH_CENTER);
    }

    return (
      <>
        <nav>
          <button onClick={handleScrollToFirstCat}>Neo</button>
          <button onClick={handleScrollToSecondCat}>Millie</button>
          <button onClick={handleScrollToThirdCat}>Bella</button>
        </nav>
        <div>
          <ul>
            <li>
              <img
                src="https://placecats.com/neo/300/200"
                alt="Neo"
                ref={firstCatRef}
              />
            </li>
            <li>
              <img
                src="https://placecats.com/millie/200/200"
                alt="Millie"
                ref={secondCatRef}
              />
            </li>
            <li>
              <img
                src="https://placecats.com/bella/199/200"
                alt="Bella"
                ref={thirdCatRef}
              />
            </li>
          </ul>
        </div>
      </>
    );
  }
  ```

### How to manage a list of refs using a ref callback

- List(array)의 각 item에 대해 ref를 동적으로 생성해야 할 수도 있음
- Hook은 component의 top-level 에서만 호출할 수 있으므로 아래와 같이 작성할 수 없음
  ```javascript
  <ul>
    {items.map((item) => {
      // Doesn't work!
      const ref = useRef(null);
      return <li ref={ref} />;
    })}
  </ul>
  ```
- 이런 경우에는 **parent element에 대해 `ref`를 1개만 얻고, `querySelectorAll` 같은 DOM 조작 methods를 사용해서 특정 list item에 해당하는 DOM node에 접근**해야 함
- 또는, React 19부터 제공되는 **ref callback**을 사용
  - `ref` attribute에 ref object 대신 callback 함수를 전달
  - `ref.current`가 설정되었을 때 parameter로 DOM node 객체가 전달됨
  - DOM node가 cleanup 되는 시점에 callback 함수에서 반환한 cleanup 함수를 실행
  - Ref callback과 cleanup function을 사용해서 list item들을 `Map`으로 관리할 수 있음
    ```javascript
    export default function CatFriends() {
      // Ref가 DOM node 대신 list item들에 대한 ref를 관리하기 위한 Map을 저장
      const itemsRef = useRef(null);
      function getMap() {
        if (!itemsRef.current) {
          itemsRef.current = new Map();
        }
        return itemsRef.current;
      }
      // ...
    }
    ```
  - `map()` method를 통해 동적으로 rendering되는 element에서 `ref` callback을 통해 `ref` 객체를 `Map`에 저장하고, cleanup 시점에는 `Map`에서 제거
    ```javascript
    export default function CatFriends() {
      const [catList, setCatList] = useState(setupCatList);
      // ...
      return (
        <>
          {/*...*/}
          <div>
            <ul>
              {catList.map((cat) => (
                <li
                  key={cat.id}
                  ref={(node) => {
                    // List item들에 대한 DOM node를 `Map` ref로 관리
                    const map = getMap();
                    map.set(cat, node);
                    // ⚠️ DOM node가 cleanup 될 때 `Map`에서도 제거해야 함
                    return () => map.delete(cat);
                  }}
                >
                  <img src={cat.imageUrl} />
                </li>
              ))}
            </ul>
          </div>
        </>
      );
    }
    ```
  - 각 list item에 대해 Web API를 사용하려면 `itemsRef`에서 `Map`을 통해 DOM node에 접근
    ```javascript
    // 특정 고양이로 scroll 시키는 click event handler
    function scrollToCat(cat) {
      const map = getMap();
      const node = map.get(cat);
      node.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
        inline: "center",
      });
    }
    ```

### Accessing another component's DOM node

```javascript
import { useRef } from "react";

function MyInput({ ref }) {
  return <input ref={ref} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

- Parent component에서 `useRef`로 생성한 ref object를 prop을 통해 child component로 전달할 수 있음
- 이와 같은 구조에서 parent component는 child component 내부의 특정 DOM node를 조작할 수 있음
  - `focus()` 등 Web API 호출
  - CSS style 변경
- Parent component에서 전달한 `ref`가 허용한 기능만 실행할 수 있도록 제한하고 싶다면, child component에서 `ref`에 대해 `useImperativeHandle` hook을 사용해서 특정 기능만 노출(expose)할 수 있음

  - Child component는 내부에서 DOM node를 얻기 위해 자체적으로 ref object 생성
    ```javascript
    export default function MyInput({ ref }) {
      const realInputRef = useRef(null);
      // ...
      return <input ref={realInputRef} />;
    }
    ```
  - Parent component가 전달한 `ref` object에는 `useImperativeHandle(ref, createHandle)` hook을 사용해서 interface를 별도로 정의
    ```javascript
    export default function MyInput({ ref }) {
      // ...
      useImperativeHandle(ref, () => ({
        focus: () => {
          realInputRef.current.focus();
        },
        // 다른 메서드도 정의 가능
      }));
      // ...
    }
    ```
  - `useImperativeHandle`의 `createHandle`은 `ref`에 DOM node가 아닌 custom object를 할당

### When React attaches the refs

- React는 component update를 두 단계에 걸쳐 진행
  1. Render phase : component 함수를 호출해서 화면에 그릴 내용을 게산
  2. Commit phase : 변경사항을 실제 DOM에 반영
- DOM node는 'commit' phase에 `ref.current`에 할당됨
  - DOM을 update하기 이전에 React는 `ref.current`를 `null`로 할당
  - DOM updating이 끝난 직후 `ref.current`에 해당 element의 DOM node를 할당
- 따라서, rendering 중간에 `ref` object에 접근하거나 `ref.current`를 통해 DOM node에 접근하면 안됨
- 일반적으로 **event handler**에서 `ref.current`에 접근하고, 그 외에는 '**Effect**'를 사용해야 함

### Flushing state updates synchronously with flushSync

- State와 ref를 함께 사용할 때, state 변경과 ref 참조의 sycn가 맞지 않는 문제가 생길 수 있음
- Todo를 추가한 뒤 추가한 item element로 scroll하는 아래 코드에서, item이 추가되기 전에 scroll이 먼저 되는 문제가 있음

  ```javascript
  export default function TodoList() {
    const listRef = useRef(null);
    const [text, setText] = useState("");
    const [todos, setTodos] = useState(initialTodos);

    function handleAdd() {
      const newTodo = { id: nextId++, text: text };
      setText("");
      setTodos([...todos, newTodo]);
      listRef.current.lastChild.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
      });
    }

    return (
      <>
        <button onClick={handleAdd}>Add</button>
        <input value={text} onChange={(e) => setText(e.target.value)} />
        <ul ref={listRef}>
          {todos.map((todo) => (
            <li key={todo.id}>{todo.text}</li>
          ))}
        </ul>
      </>
    );
  }
  ```

  - State 변경은 queue에 저장되었다가 next render에서 한 번에 처리되므로, `setTodos()` setter 함수로 state를 변경하더라도 `handleAdd` event handler가 종료되기 전까지는 state가 변경되지 않음 (snapshot 처럼 동작)
  - 따라서, **`setTodos()` 호출 직후 `listRef.current.lastChild`는 previous state를 기준으로 DOM node를 찾기 때문에**, 새 todo가 추가되기 이전 list에서 last child를 참조하여 발생하는 문제
  - 이 문제를 해결하려면 **React가 DOM을 동기적으로 갱신하도록("flush") 강제**해야 함
  - `react-dom` package의 `flushSync(callback)` 함수는 React가 `callback` 함수가 실행된 직후 DOM을 동기적으로 갱신하도록 함
  - `callback`에서 state setter 함수를 실행하여 state를 즉시 변경하도록 만든다.
    ```javascript
    function handleAdd() {
      const newTodo = { id: nextId++, text: text };
      // 변경된 state를 DOM에 동기적으로 즉시 반영
      flushSync(() => {
        setText("");
        setTodos([...todos, newTodo]);
      });
      listRef.current.lastChild.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
      });
    }
    ```

### Best practices for DOM manipulation with refs

- Ref는 'escape hatch' 이므로 React를 벗어나고자 할 때만 사용해야 함 (focus 관리, scroll position, browser API 호출 등)
- Focusing이나 scrolling 등 layout을 변경하지 않으면 문제가 발생하지 않지만, **DOM을 직접 변경하는 경우에는 React가 만든 UI results와 충돌(conflict)할 수 있음**

  ```javascript
  import { useState, useRef } from "react";

  export default function Counter() {
    const [show, setShow] = useState(true);
    const ref = useRef(null);

    return (
      <div>
        <button onClick={() => setShow(!show)}>Toggle with setState</button>
        <button onClick={() => ref.current.remove()}>
          Remove from the DOM
        </button>
        {show && <p ref={ref}>Hello world</p>}
      </div>
    );
  }
  ```

  - State를 사용할 때는 React에 의해 rendering이 관리되므로, 조건부 렌더링에 의해 'Hello world' 문구가 toggle 됨
  - Ref를 사용할 때는 React의 동작과 무관하게 DOM을 직접 변경하므로, React 코드와 상관 없이 동작 ('Hello world' 문구가 DOM에서 아예 제거되어 다시 나타나지 않음)
  - `ref.current.remove()`로 DOM에서 element를 제거한 뒤, state를 toggle하면 error 발생
    <br><img src="./ref-state-error.png" /><br>

- 따라서, **React에서 ref를 통해 DOM을 직접 변경하는 작업을 주의해야 한다.**
- React가 갱신하지 않는 DOM node는 ref를 통해 직접 조작해도 된다.
- 가령, Child를 갖지 않는 빈 `<div>`는 React에서 children list를 touch할 일이 없으므로 ref를 통해 DOM을 직접 조작해도 괜찮다.
