# 목차

-   [References](#references)
-   [시작하기](#시작하기)
-   []()

---

## References

-   [리액트를 위한 스토리북 튜토리얼 | Storybook.js.org](https://storybook.js.org/tutorials/intro-to-storybook/react/en/get-started/)

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

-   [Setup](#setup)
-   [Config](#config)
-   [Catch accessibilisty issues](#catch-accessibility-issues)

#### Setup

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
            <label htmlFor="title" aria-label={title}>
                <input type="text" value={title} readOnly={true} name="title" />
            </label>
        </div>
    );
}
```

```javascript
// src/components/Task.stories.js
import Task from "./Task";

export default {
    component: Task,
    title: "Task",
    tags: ["autodocs"],
};

export const Default = {
    args: {
        task: {
            id: "1",
            title: "Test Task",
            state: "TASK_INBOX",
        },
    },
};

export const Pinned = {
    args: {
        task: {
            ...Default.args.task,
            state: "TASK_PINNED",
        },
    },
};

export const Archived = {
    args: {
        task: {
            ...Default.args.task,
            state: "TASK_ARCHIVED",
        },
    },
};
```

-   스토리북은 컴포넌트와 컴포넌트 스토리, 두 가지 기본 단계로 구성됨.
-   각 스토리는 컴포넌트에 대응되며, 필요한 만큼의 스토리를 컴포넌트 별로 작성 가능.
-   스토리북에게 문서화하고 있는 컴포넌트를 알려주기 위해, `export default`를 생성.

    -   `component` 해당 컴포넌트
    -   `title` 스토리북 사이드바에서 컴포넌트를 그룹화하고 분류하는 방법
    -   `tags` 컴포넌트에 대한 문서를 자동으로 생성하기 위함

-   `args`를 사용하여 스토리북을 재시작하지 않고도, Controls addon으로 컴포넌트를 실시간으로 수정 가능. 즉, `args` 값이 변하면 컴포넌트도 동시에 변화.

-   스토리 생성 시, 기본 `task` 인수를 사용해 컴포넌트가 예상하는 task의 형태를 구성하고 이는 일반적으로 실제 데이터를 모델로 생성됨. `export`하는 것은 차후 스토리에서 이를 재사용할 수 있도록 함.
    -   [`액션(Actions addon)`](https://storybook.js.org/docs/react/essentials/actions)은 UI 컴포넌트를 독립적으로 만들 때, 컴포넌트 상호작용을 확인할 때 도움이 됨. 앱의 컨텍스트에서 함수와 state에 접근하지 못할 때, `action()`을 사용해 끼워 넣으면 됨.

#### Config

-   스토리북의 구성 파일을 몇 가지 변경해 최근 생성 스토리를 인식하고 애플리케이션의 CSS 파일(src/index.css에 위치한)을 사용할 수 있도록 해야 함.

```javascript
// .storybook/main.js
/** @type { import('@storybook/react-vite').StorybookConfig } */
const config = {
    stories: ["../src/components/**/*.stories.@(js|jsx)"],
    staticDirs: ["../public"],
    addons: [
        "@storybook/addon-links",
        "@storybook/addon-essentials",
        "@storybook/addon-interactions",
    ],
    framework: {
        name: "@storybook/react-vite",
        options: {},
    },
    docs: {
        autodocs: "tag",
    },
};
export default config;
```

```javascript
// .storybook/preview.js
import "../src/index.css";

//👇 Configures Storybook to log the actions( onArchiveTask and onPinTask ) in the UI.
/** @type { import('@storybook/react').Preview } */
const preview = {
    parameters: {
        actions: { argTypesRegex: "^on[A-Z].*" },
        controls: {
            matchers: {
                color: /(background|color)$/i,
                date: /Date$/,
            },
        },
    },
};

export default preview;
```

-   `parameters`는 일반적으로 스토리북의 기능과 애드온의 동작을 제어하기 위해 사용. 이를 사용하여 `actions(mocked callbacks)`이 처리되는 방식을 구성.
-   `actions`은 클릭이 되었을 때, 스토리북 UI의 actions 패널에 나타날 콜백을 생성함. 따라서 pin 버튼을 만들때, UI에서 버튼 클릭의 성공 여부를 확인 가능.

#### Catch accessibility issues

-   접근성 테스트는 자동화된 도구를 사용하여 렌더링된 DOM을 WCAG 규칙 및 기타 업계에서 인정하는 모범 사례에 기반한 일련의 휴리스틱에 따라 감사하는 관행.

-접근성 테스트는 시각 장애, 청각 장애, 인지 장애 등의 장애가 있는 사람을 포함하여 최대한 많은 사람이 애플리케이션을 사용할 수 있도록 노골적인 접근성 위반을 잡아내는 QA의 첫 번째 라인 역할.

-   스토리북에는 공식 접근성 애드온이 포함되어 있습니다. Deque의 액스 코어로 구동되는 이 애드온은 최대 57%의 WCAG 문제를 포착.

```shell
$ npm install --save-dev @storybook/addon-a11y
$
```

-   `@storybook/addon-a11y` 플러그인을 설치하고, `.storybook/main.js` 파일의 `addon`에 추가.
-   서버를 재시작하면, 컴포넌트의 상태에 따라 접근성 요소가 문제가 있으면 Accessibility 탭에서 원인을 파악 가능.
