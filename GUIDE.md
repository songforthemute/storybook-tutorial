# 목차

-   [References](#references)
-   [시작하기](#시작하기)
-   []()

---

## References

-   [리액트를 위한 스토리북 튜토리얼 | Storybook.js.org](https://storybook.js.org/tutorials/intro-to-storybook/react/ko/get-started/)

---

## 시작하기

스토리북은 개발 모드에서 앱과 함께 실행, 비즈니스 로직과 맥락(context)으로부터 분리된 UI 컴포넌트를 만들 수 있도록 도와주는 라이브러리. 리액트 뿐만 아니라 RN, Vue, Angular, Svelte 등을 지원.

[컴포넌트 주도 개발 방법론(CDD, Component-Driven Development)](https://www.componentdriven.org)에 따라 UI를 구성할 것이며, 이는 컴포넌트부터 시작하여 마지막 화면에 이르기 까지 상향식(bottom-up)으로 UI를 개발하는 과정. CDD는 UI 구축 시 직면하게 되는 규모의 복잡성을 해결하는 데 효과적.

```json
"scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build",
    "init-msw": "msw init public/"
},
```

```bash
# Start the component explorer on port 6006:
npm run storybook
yarn storybook
```

---

## 간단한 컴포넌트

```jsx
// src/components/Task.jsx
import React from "react";

export default function Task({
    task: { id, title, state },
    onArchiveTask,
    onPinTask,
}) {
    return (
        <div className="list-item">
            <input type="text" value={title} readOnly={true} />
        </div>
    );
}
```

```javascript
// src/components/Task.stories.js
import React from "react";

import Task from "./Task";

export default {
    component: Task,
    title: "Task",
};

const Template = (args) => <Task {...args} />;

// 컴포넌트(Task)의 각각의 상태
export const Default = Template.bind({});
Default.args = {
    task: {
        id: "1",
        title: "Test Task",
        state: "TASK_INBOX",
        updatedAt: new Date(2021, 0, 1, 9, 0),
    },
};

export const Pinned = Template.bind({});
Pinned.args = {
    task: {
        ...Default.args.task,
        state: "TASK_PINNED",
    },
};

export const Archived = Template.bind({});
Archived.args = {
    task: {
        ...Default.args.task,
        state: "TASK_ARCHIVED",
    },
};
```

-   스토리북은 컴포넌트와 컴포넌트 스토리, 두 가지 기본 단계로 구성됨.
-   각 스토리는 컴포넌트에 대응되며, 필요한 만큼의 스토리를 컴포넌트 별로 작성 가능.
-   스토리북에게 문서화하고 있는 컴포넌트를 알려주기 위해, `export default`를 생성.
    -   `component` 해당 컴포넌트
    -   `title` 스토리북 앱의 사이드바에서 컴포넌트를 참조하는 플래그
-   컴포넌트의 스토리가 여러 개이므로, `Template`변수에 `bind`함수를 통해 할당하고 관리하는 패턴으로 작성할 코드를 줄일 수 있음.
    -   _`Template.bind({})`는 함수의 복사본을 만드는 표준 JavaScript의 기법. 이 기법을 사용하여 각각의 스토리가 고유한 속성(properties)을 갖지만 동일한 결과물을 사용하도록 함._
-   `args`를 사용하여 스토리북을 재시작하지 않고도, Controls addon으로 컴포넌트를 실시간으로 수정 가능. 즉, `args` 값이 변하면 컴포넌트도 동시에 변화.
-   스토리 생성 시, 기본 `task` 인수를 사용해 컴포넌트가 예상하는 task의 형태를 구성. 이는 일반적으로 실제 데이터를 모델로 생성됨.
    -   [`액션(Actions addon)`](https://storybook.js.org/docs/react/essentials/actions)은 UI 컴포넌트를 독립적으로 만들 때, 컴포넌트 상호작용을 확인할 때 도움이 됨. 앱의 컨텍스트에서 함수와 state에 접근하지 못할 때, `action()`을 사용해 끼워 넣으면 됨.
