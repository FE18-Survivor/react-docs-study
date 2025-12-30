# Week 09
## Removing Effect Dependencies
> 여러 예제를 통해 Effect와, Effect의 Dependency list를 어떻게 사용하고 수정하면 좋을지 알아보겠습니다.

### Dependencies should match the code 
dependencies에 "reactive" 코드가 없으면 오류가 발생합니다.

chatRoom 예시코드로 얘기해보겠습니다.

```javascript
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- 🚨 오류
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        ... {/* 생략 */}
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}

```

roomId가 "reactive"되는 props이기 때문에 dependencies에 추가해줘야 오류가 발생하지 않습니다.

```javascript
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

dependencies를 제거하고 싶다면 이와 같은 "reactive" 변수가 effect에 없어야 합니다.

```javascript
const serverUrl = 'https://localhost:1234';
const roomId = 'music'; 

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 
  // ...
}
```

따라서 위와 같이 변경하면 dependency list가 공란일 수 있게 됩니다. 값이 변화하는 변수가 없으니 Effect는 한번만 렌더링 할 것입니다.

#### To change the dependencies, change the code 
dependencies 변경의 workflow는 다음처럼 정리할 수 있습니다.

1. Effect의 코드나 반응형 값이 선언된 방식을 변경합니다.
2. 린터의 안내에 따라 변경된 코드에 맞게 의존성 목록을 조정합니다.
3. 만약 의존성 목록이 마음에 들지 않는다면, 다시 첫 단계로 돌아갑니다.

즉 dependency list는 당신의 코드를 서술합니다.

### Removing unnecessary dependencies 
Effect는 종종 dependencies가 바뀌어도 재실행하지 않아야 할수도 있습니다.

- 서로 다른 조건에 따른 Effect의 서로 다른 부분만 다시 실행하고 싶을 수도 있습니다.
- 어떤 의존성의 변경에 **반응**하고 싶지는 않고, 단지 그 최신 값만 읽고 싶을 수도 있습니다. 
- 의존성이 객체나 함수이기 때문에, 의도하지 않게 너무 자주 변경될 수도 있습니다. 

몇가지 질문을 통해 Effect를 수정할 수 있습니다.

#### [Should this code move to an event handler?](https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler)

```javascript
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      post('/api/register');
      showNotification('Successfully registered!', theme);
    }
  }, [submitted, theme]); // ✅

  function handleSubmit() {
    setSubmitted(true);
  }  

  // ...
}
```

dependencies에 reactive value를 전부 넣었지만, `submitted`, `theme` 중 하나가 변경되어 다른 하나에도 영향을 줄 수 있습니다. 따라서 이런경우엔 Event handler로 분리하는것이 좋습니다.

```javascript
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    post('/api/register');
    showNotification('Successfully registered!', theme);
  }  

  // ...
}
```

#### [Is your Effect doing several unrelated things? ](https://react.dev/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things)

이 예제는 country를 선택하고, 그 선택값을 통해서 fetch되어 온 data가 있다면 city의 값을 설정할 수 있도록 하는 코드입니다.

```javascript
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅

  // ...
```

첫번째 예제를 보면 Effect를 활용한 좋은 fetch 예제입니다. country의 값이 변경되면 서버로부터 바로 data를 가져와야 하기 때문입니다.

하지만 그 아래에 city에 대한 코드를 추가한다면 어떻게 될까요?

```javascript
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    // 🔴 추가된 부분
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]); // ✅ 

  // ...
```

city가 dependencies에 추가된것은 맞지만, country가 변경될때마다 city가 계속 호출되어집니다.

이 코드는 서로 관련없는 작업이 synchronizing 된다는 문제가 있는것입니다.

따라서 두 코드를 분리하는것이 더 좋습니다.

```javascript
...
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅
```

city와, area의 state를 만들고, 사용자가 선택한 city와 fetch 될 data를 area에 담을 수 있도록 분리했습니다.


#### [Are you reading some state to calculate the next state?](https://react.dev/learn/removing-effect-dependencies#are-you-reading-some-state-to-calculate-the-next-state)

```javascript
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ 
  // ...
```

messages를 받게되어 dependencies에 같이 추가한 예제 코드입니다. 이렇게 하게 되면 message를 받을때마다 re-render -> re-synchronize -> re-connect 까지 계속 발생되기 때문에 이를 바꿔야 합니다.

```javascript
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]); // ✅ 
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ 
  // ...
```

set 함수에 updater function을 넘겨주어 Effect가 message를 읽는것이 아니라, React가 해당 function을 queue에 넣고 해당 message를 읽을 것입니다.

#### [Do you want to read a value without “reacting” to its changes?](https://react.dev/learn/removing-effect-dependencies#do-you-want-to-read-a-value-without-reacting-to-its-changes)

```javascript
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ 
  // ...
```
위의 코드대로 라면, 사용자가 toggle 버튼을 통해서 isMuted가 변경되었을때 message도 re-connect됩니다. 이를 해결하는 방법으로 EffectEvent를 활용할 수 있습니다. isMuted의 최신값은 읽고, Effect를 다시 실행시키지는 않는거죠.


```javascript
import { useState, useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent (receivedMessage => { // ✅ 
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ 
  // ...
```
EffectEvent를 통해 Effect를 `reactive`한 부분과 `non-reactive`으로 나눌 수 있게 됩니다.

- [예제 1: event hadler wrapping하기](https://react.dev/learn/removing-effect-dependencies#wrapping-an-event-handler-from-the-props)
- [예제 2: 로그 남기기](https://react.dev/learn/removing-effect-dependencies#separating-reactive-and-non-reactive-code)


#### [Does some reactive value change unintentionally?](https://react.dev/learn/removing-effect-dependencies#does-some-reactive-value-change-unintentionally)

```javascript
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      {/* ... */}
      <ChatRoom roomId={roomId} />
    </>
  );
}

```

options에 따라 Effect가 실행되는 코드입니다. 이때 문제는 options 내부 property 값이 동일해도 re-render 될때마다 다른 객체로 본다는 점입니다.

아래의 예제와 방법으로 해결할 수 있습니다.

1. [Move static objects and functions outside your component](https://react.dev/learn/removing-effect-dependencies#move-static-objects-and-functions-outside-your-component)

2. [Move dynamic objects and functions inside your Effect](https://react.dev/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

3. [Read primitive values from objects](https://react.dev/learn/removing-effect-dependencies#read-primitive-values-from-objects)

4. [Calculate primitive values from functions](https://react.dev/learn/removing-effect-dependencies#calculate-primitive-values-from-functions)

