# Week-05

(12.02)

## [Reacting to Input with State](https://react.dev/learn/reacting-to-input-with-state)

- 선언형 UI 프로그래밍과 명령형 UI 프로그래밍이 어떻게 다른지
- 컴포넌트에 있는, 서로 다른 시각적 state들을 어떻게 열거하는지
- 서로 다른 시각적 state들 사이의 변화를 어떻게 trigger 하는지

### How declarative UI compares to imperative

- 명령형 프로그래밍은 사용자와 상호작용이(intercation, event, etc) 기대된다면, 전부 나열해야합니다.
- "무엇"을 클릭하는지, "어떤" 행동을 해야하는지 등
- 하나부터 열까지 전부 commands를 입력해야 합니다.
- 모든 UI elements들에 대한 interaction을 나열해야 합니다.

리액트는 이러한 문제를 해결하기 위해 만들어졌습니다.

리액트에선 직접적으로 UI를 조작하지 않고, 무엇을 보여주고 싶은지 선언하면 됩니다. 선언을 하면 리액트가 UI를 어떻게 업데이트 할지 파악합니다.

### Thinking about UI declaratively

1. Identify your component’s different visual states
2. Determine what triggers those state changes
3. Represent the state in memory using useState
4. Remove any non-essential state variables
5. Connect the event handlers to set the state

#### Step 1: Identify your component’s different visual states

서로 다른 "states"에 대한 시각화가 필요합니다.

- Empty: Form has a disabled “Submit” button.
- Typing: Form has an enabled “Submit” button.
- Submitting: Form is completely disabled. Spinner is shown.
- Success: “Thank you” message is shown instead of a form.
- Error: Same as Typing state, but with an extra error message.

이러한 흐름을 mock up 하여 코드에 상태를 전달 할 수 있습니다.

UI가 무엇을 어떻게 행동 할지 명령하는것이 아니라, 어떤 "상태"에 있을것인지 먼저 파악하는 것입니다.

state가 가진 값이 변하면, 그 변화에 따라 어떤 행동을 할것인지를 선언하여 UI를 그릴 수 있습니다.

#### Step 2: Determine what triggers those state changes

states의 변화를 어떻게 trigger 할지 정해야 합니다.

- Human inputs: like clicking a button, typing in a field, navigating a link.
- Computer inputs: like a network response arriving, a timeout completing, an image loading.

![responding to input flow from react.dev](responding_to_input_flow.webp)

typing과 press는 human input,
submitting 이후의 행동은 computer input으로 구분할 수 있습니다.

#### Step 3: Represent the state in memory with useState

`useState`를 활용해서 먼저 mock up 했던 states를 선언하고 변경합니다. flow에 맞춰서 필요한 state를 전부 선언해보고 그 이후에 정리하면 됩니다.

#### Step 4: Remove any non-essential state variables

state는 단순하고, 중복을 피하는것이 좋습니다. 아래 문장을 기준으로 state를 정리하면 좋습니다.

- Does this state cause a paradox?
- Is the same information available in another state variable already?
- Can you get the same information from the inverse of another state variable?

#### Step 5: Connect the event handlers to set state

마지막으로 event handler와 state를 연결하면 됩니다.

### Example

#### 명령형

```javascript
//index.js
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = "none";
}

function show(el) {
  el.style.display = "";
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === "istanbul") {
        resolve();
      } else {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      }
    }, 1500);
  });
}

let form = document.getElementById("form");
let textarea = document.getElementById("textarea");
let button = document.getElementById("button");
let loadingMessage = document.getElementById("loading");
let errorMessage = document.getElementById("error");
let successMessage = document.getElementById("success");
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

<br />

#### 선언형

```javascript
import { useState } from "react";

export default function Form() {
  const [answer, setAnswer] = useState("");
  const [error, setError] = useState(null);
  const [status, setStatus] = useState("typing");

  if (status === "success") {
    return <h1>That's right!</h1>;
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus("submitting");
    try {
      await submitForm(answer);
      setStatus("success");
    } catch (err) {
      setStatus("typing");
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === "submitting"}
        />
        <br />
        <button disabled={answer.length === 0 || status === "submitting"}>
          Submit
        </button>
        {error !== null && <p className="Error">{error.message}</p>}
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== "lima";
      if (shouldError) {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

## [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)

- 언제 single, multiple을 구분해서 사용하는지
- state를 organizing 할때 무엇을 피해야 하는지
- state를 통해 공통 이슈를 해결할 수 있는지

### Principles for structuring state

state변수를 만들때 조금 더 좋은 선택을 위한 몇가지 원칙이 있습니다.

1. Group related state: 두 개 이상의 상태를 동시에 업데이트 해야한다면 하나의 state로 관리하는것이 좋습니다.

2. Avoid contradictions in state: 서로 반대되는 의미를 가지게 된다면 그런 구조는 피해야 합니다.

3. Avoid redundant state: 이미 존재하는 state로 계산할 수 있다면 계산을 위한 state를 만들지 않는것이 좋습니다.

4. Avoid duplication in state: 여러 state변수에 같은 데이터가 있으면 데이터간 동기화가 어렵습니다.

5. Avoid deeply nested state: state가 중첩구조가 되면 업데이트 하기 까다로워 집니다.

#### examples

- [Avoid contradictions in state](https://react.dev/learn/choosing-the-state-structure#avoid-contradictions-in-state)
- [Avoid duplication in state](https://react.dev/learn/choosing-the-state-structure#avoid-duplication-in-state)
