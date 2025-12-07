# React 공식 문서 스터디

## Overview

- 도구 : React 공식 문서 (v19.2 기준) ([Learn](https://react.dev/learn), [Reference](https://react.dev/reference/react))
- 일정 : 매주 화, 금 오전 10시
- 진행 방식
  - 정해진 분량을 각자 읽고 스터디 전까지 push
  - 스터디 당일 랜덤으로 선정된 발표자가 공식 문서 또는 정리한 문서를 바탕으로 발표 및 Q&A 진행

## 파일 작성 방법

- 주차 별로 `name/week-nn` branch를 만들어서 markdown file에 내용을 정리한 뒤 PR을 생성합니다.
- Markdown file 이름은 `name.md`로 합니다.
- 이미지를 사용하고 싶은 경우, `name` 폴더를 만들고 그 아래 `index.md` 파일에 내용을 정리합니다.
- 예시
  ```
  week-01
  ㄴ README.md
  ㄴ name1.md
  ㄴ name2.md
  week-02
  ㄴ README.md
  ㄴ name1
    ㄴ index.md
    ㄴ some-image.png
    ㄴ ...
  ㄴ name2.md
  ```

## 저장소 활용 방법

- 각자 스터디 전날까지 `name/week-nn` branch를 `main` branch로 병합하는 PR을 생성합니다.
- 다른 스터디원들은 자율적으로 PR에서 의견을 나눕니다.
- 스터디 당일 랜덤으로 선정된 발표자가 정리한 내용을 바탕으로 발표 및 Q&A를 진행합니다.
- 원하는 경우 다른 스티디원이 올린 PR을 함께 읽어보며 토론합니다.
- 스터디 준비 중에 생기는 질문은 'Discussion'에 남기고 자유롭게 토론합니다.

## History

|        Week        |                                                                                                                                   Subjects                                                                                                                                   |    Date    |
| :----------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :--------: |
| [01-1](./week-01/) |            [Your First Component](https://react.dev/learn/your-first-component)<br>[Importing and Exporting Components](https://react.dev/learn/importing-and-exporting-components)<br>[Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx)            | 2025.11.04 |
| [01-2](./week-01/) |                                                                                      [JavaScript in JSX with Curly Braces](https://react.dev/learn/javascript-in-jsx-with-curly-braces)                                                                                      | 2025.11.07 |
| [02-1](./week-02/) |                                                        [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)<br>[Conditional Rendering](https://react.dev/learn/conditional-rendering)                                                        | 2025.11.11 |
| [02-2](./week-02/) |                           [Rendering Lists](https://react.dev/learn/rendering-lists)<br>[Keeping Components Pure](https://react.dev/learn/keeping-components-pure)<br>[Your UI as a Tree](https://react.dev/learn/understanding-your-ui-as-a-tree)                           | 2025.11.14 |
| [03-1](./week-03/) |                                                           [Responding to Events](https://react.dev/learn/responding-to-events)<br>[State: A Component's Memory](https://react.dev/learn/state-a-components-memory)                                                           | 2025.11.18 |
| [03-2](./week-03/) |                   [Render and Commit](https://react.dev/learn/render-and-commit)<br>[State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)<br>[Queueing a Series of State Updates](https://react.dev/learn/queueing-a-series-of-state-updates)                   | 2025.11.21 |
| [04-1](./week-04/) |                                                        [Updating Objects in State](https://react.dev/learn/updating-objects-in-state)<br>[Updating Arrays in State](https://react.dev/learn/updating-arrays-in-state)                                                        | 2025.11.25 |
| [05-1](./week-05/) | [Reaching to Input with State](https://react.dev/learn/reacting-to-input-with-state)<br>[Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)<br>[Sharing State Between Components](https://react.dev/learn/sharing-state-between-components) | 2025.12.05 |
| [06-1](./week-06/) |                                      [Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state)<br>[Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)                                      | 2025.12.09 |
