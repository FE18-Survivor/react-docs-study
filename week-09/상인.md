# Removing Effect Dependencies

Effect는 내부의 모든 **reactive value**에 반응하며, 의존성 배열에 반드시 포함해야한다.

의존성 배열에 reactive value를 포함하지 않으면 린터가 알려준다. 린터를 끄는 것이 아니라, 코드 구조를 바꾸는 것을 추천한다.

> eslint-disable-next-line react-hooks/exhaustive-deps
>
> Effect가 실제로 의존하는 값과 의존성 배열이 불일치하게 된다. 값은 변했는데 Effect는 안 돈다 또는 안 변했다고 생각했는데 계속 돈다 같은 버그들이 발생할 가능성이 있다.

## 의존성 제거를 위한 질문들

1. Effect인가 이벤트 핸들러인가
2. Effect가 서로 다른 일을 여러 개 하고 있는지
3. 이전 state를 읽어서 다음 state를 계산하고 있는지
4. 단순히 값만 읽고 싶을 때
5. 의도치 않게 변경되었는지

### 1. Effect인가 이벤트 핸들러인가

- `theme`가 바뀜에 따라서 Effect가 실행되는 의도치 않은 동작
- 폼 제출과 같은 특정 상호작용에 따른 로직은 이벤트 핸들러로 옮기기

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      post("/api/register");
      showNotification("Successfully registered!", theme);
    }
  }, [submitted, theme]);

  function handleSubmit() {
    setSubmitted(true);
  }
}
```

```jsx
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    post("/api/register");
    showNotification("Successfully registered!", theme);
  }
}
```

### 2. Effect가 서로 다른 일을 여러 개 하고 있는지

- 두 개의 동기화 과정이 하나의 Effect에서 처리

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then((response) => response.json())
      .then((json) => {
        if (!ignore) {
          setCities(json);
        }
      });
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then((response) => response.json())
        .then((json) => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]);
}
```

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then((response) => response.json())
      .then((json) => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then((response) => response.json())
        .then((json) => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);
}
```

### 3. 이전 state를 읽어서 다음 state를 계산하고 있는지

- `messages`가 변경될 때마다 채팅 연결
- `messages`를 Effect 내부에서 직접 읽지 말고 업데이터 함수 이용하기

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on("message", (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]);
}
```

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on("message", (receivedMessage) => {
      setMessages((msgs) => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]);
}
```

### 4. Effect 내부에서 단순히 값만 일고 싶을 때

- `isMuted`에 의해서 채팅이 연결됨
- Effect Event 만들기 (useEffectEvent)

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on("message", (receivedMessage) => {
      setMessages((msgs) => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]);
}
```

```jsx
import { useState, useEffect, useEffectEvent } from "react";

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent((receivedMessage) => {
    setMessages((msgs) => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on("message", (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]);
}
```

- `onReceiveMessage`를 prop으로 받는 상황: 부모에서 렌더마다 새 함수를 전달한다고 가정할 때 재연결 발생
- Effect Event 만들기

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent((receivedMessage) => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on("message", (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]);
}
```

### 5. 의도치 않게 변경되었는지

- 객체/함수가 deps에 있을 때, 의도치 않은 동기화가 일어날 수 있음
- 가능한 객체/함수를 의존성으로 사용하지 않기
- 컴포넌트 외부 혹은 Effect 내부로 이동하기
-

```jsx
function ChatRoom({ roomId }) {
  // message가 바뀌어도 새로운 렌더링에 의해 options가 새로 생성됨
  const [message, setMessage] = useState("");

  const options = {
    serverUrl: serverUrl,
    roomId: roomId,
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);
}
```

```jsx
const options = {
  serverUrl: "https://localhost:1234",
  roomId: "music",
};

function ChatRoom() {
  const [message, setMessage] = useState("");

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []);
}
```

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
}
```

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState("");

  const { roomId, serverUrl } = options; // options의 프로퍼티를 의존성으로 사용하기
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl,
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

# Reusing Logic with Custom Hooks

`useState, useEffect`와 같이 내장된 hook이 아닌, 목적에 따라서 직접 만드는 hook

## Hook의 규칙

- 네이밍: Hook은 반드시 use 뒤에 대문자로 시작하게 만들어야한다.

## Hook을 사용하는 컴포넌트끼리 상태를 공유하는 것이 아닌 로직을 공유

- Hook을 사용하는 두 컴포넌트의 state와 Effect는 독립적이다.

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
}

function SaveButton() {
  const isOnline = useOnlineStatus();
}
```

```jsx
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
}
```

## Hook은 순수해야한다

- 커스텀 훅의 코드는 컴포넌트 렌더링마다 다시 실행되므로 순수해야 하고, 항상 최신 `props,state` 를 받음
- 예시로 `serverUrl`을 인자로 넘기면, `serverUrl`이 바뀔 때마다 Hook 내부의 Effect도 재동기화 된다.

```jsx
import { useState } from "react";
import { useChatRoom } from "./useChatRoom.js";

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
  });
}
```

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on("message", (msg) => {
      showNotification("New message: " + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

## Hook에서 Effect Event

- Hook에 핸들러를 넘겨줄 때 마찬가지로 의존성 문제로 인한 버그가 있다면 Effect Event를 사용해서 래핑하기

```jsx
import { useEffect, useEffectEvent } from "react";
// ...

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on("message", (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

## 커스텀 Hook을 언제 쓰는지

- 단순히 사소한 중복은 굳이 안만들어도 됨
- 만들었을 때 컴포넌트의 흐름과 의도가 명확해 지는지 고려
- Effect 동기화 시 커스텀 Hook이 유용

## Effect를 커스텀 Hook으로 만들었을 때 유용한 이유

- 처음에는 `useState,useEffect`로 만들었다가 `useSyncExternalStore`로 교체하는 상황
- 새로운 API가 생기거나 더 나은 구현이 가능해졌을 때 커스텀 Hook의 내부만 교체가능

```jsx
export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);
  return isOnline;
}
```

```jsx
export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );
}
```
