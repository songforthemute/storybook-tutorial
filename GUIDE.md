# 목차

-   [References](#references)
-   [시작하기](#시작하기)
-   [간단한 컴포넌트](#간단한-컴포넌트)
-   [복잡한 컴포넌트](#복잡한-컴포넌트)
-   [데이터 연결](#데이터-연결)
-   [스크린 구성하기](#스크린-구성하기)

---

## References

-   [리액트를 위한 스토리북 튜토리얼 | Storybook.js.org](https://storybook.js.org/tutorials/intro-to-storybook/react/en/get-started/)

---

ㄴ

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

-   [Setup Task](#setup-task)
-   [Config](#config)
-   [Catch accessibilisty issues](#catch-accessibility-issues)

### Setup Task

#### `Task.jsx`

```jsx
// src/components/Task.jsx
import React from "react";

export default function Task({
    task: { id, title, state },
    onArchiveTask,
    onPinTask,
}) {
    return (
        <div className={`list-item ${state}`}>
            <label
                htmlFor="checked"
                aria-label={`archiveTask-${id}`}
                className="checkbox"
            >
                <input
                    type="checkbox"
                    disabled={true}
                    name="checked"
                    id={`archiveTask-${id}`}
                    checked={state === "TASK_ARCHIVED"}
                />
                <span
                    className="checkbox-custom"
                    onClick={() => onArchiveTask(id)}
                />
            </label>

            <label htmlFor="title" aria-label={title} className="title">
                <input
                    type="text"
                    value={title}
                    readOnly={true}
                    name="title"
                    placeholder="Input title"
                />
            </label>

            {state !== "TASK_ARCHIVED" && (
                <button
                    className="pin-button"
                    onClick={() => onPinTask(id)}
                    id={`pinTask-${id}`}
                    aria-label={`pinTask-${id}`}
                    key={`pinTask-${id}`}
                >
                    <span className={`icon-star`} />
                </button>
            )}
        </div>
    );
}
```

#### `Task.stories.jsx`

```jsx
// src/components/Task.stories.jsx
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

### Config

-   스토리북의 구성 파일을 몇 가지 변경해 최근 생성 스토리를 인식하고 애플리케이션의 CSS 파일(src/index.css에 위치한)을 사용할 수 있도록 해야 함.

#### `main.js`

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

#### `preview.js`

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

### Catch accessibility issues

-   접근성 테스트는 자동화된 도구를 사용하여 렌더링된 DOM을 WCAG 규칙 및 기타 업계에서 인정하는 모범 사례에 기반한 일련의 휴리스틱에 따라 감사하는 관행.

-접근성 테스트는 시각 장애, 청각 장애, 인지 장애 등의 장애가 있는 사람을 포함하여 최대한 많은 사람이 애플리케이션을 사용할 수 있도록 노골적인 접근성 위반을 잡아내는 QA의 첫 번째 라인 역할.

-   스토리북에는 공식 접근성 애드온이 포함되어 있습니다. Deque의 액스 코어로 구동되는 이 애드온은 최대 57%의 WCAG 문제를 포착.

```shell
$ npm install --save-dev @storybook/addon-a11y
$
```

-   `@storybook/addon-a11y` 플러그인을 설치하고, `.storybook/main.js` 파일의 `addon`에 추가.
-   서버를 재시작하면, 컴포넌트의 상태에 따라 접근성 요소가 문제가 있으면 Accessibility 탭에서 원인을 파악 가능.

---

## 복잡한 컴포넌트

-   `Task`를 담고 있는--Pinned Task를 기본 작업 위에 배치하여 강조하는 `TaskBox`같은 컴포넌트나, `Task` 데이터는 비동기적으로 전송될 수 있으므로 연결이 없을 때 렌더링할 로딩 상태도 필요.

-   [Setup TaskList](#setup-tasklist)

### Setup TaskList

#### `TaskList.jsx`

```jsx
// src/components/TaskList.jsx
import React from "react";
import Task from "./Task";

export default function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
    const events = {
        onPinTask,
        onArchiveTask,
    };

    const LoadingRow = (
        <div className="loading-item">
            <span className="glow-checkbox" />
            <span className="glow-text">
                <span>Loading</span> <span>cool</span> <span>state</span>
            </span>
        </div>
    );

    if (loading) {
        return (
            <div className="list-items" data-testid="loading" key={"loading"}>
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
            </div>
        );
    }

    if (tasks.length === 0) {
        return (
            <div className="list-items" key={"empty"} data-testid="empty">
                <div className="wrapper-message">
                    <span className="icon-check" />
                    <p className="title-message">You have no tasks</p>
                    <p className="subtitle-message">Sit back and relax</p>
                </div>
            </div>
        );
    }

    const tasksInOrder = [
        ...tasks.filter((t) => t.state === "TASK_PINNED"),
        ...tasks.filter((t) => t.state !== "TASK_PINNED"),
    ];
    return (
        <div className="list-items">
            {tasksInOrder.map((task) => (
                <Task key={task.id} task={task} {...events} />
            ))}
        </div>
    );
}
```

#### `TaskList.stories.jsx`

```jsx
// src/components/TaskList.stories.jsx
import TaskList from "./TaskList";
import * as TaskStories from "./Task.stories";

export default {
    component: TaskList,
    title: "TaskList",
    decorators: [(story) => <div style={{ padding: "3rem" }}>{story()}</div>],
    tags: ["autodocs"],
};

/**
 * 데코레이터는 스토리에 어떤 Wrapper를 제공하는 방법.
 * 여기서는 데코레이터를 사용해 렌더링된 컴포넌트 주위에 패딩을 추가하고 있음.
 * React Context를 설정하는 라이브러리 컴포넌트에서 스토리를 감싸는 데도 사용할 수 있음.
 */

export const Default = {
    args: {
        // Shaping the stories through args composition.
        // The data was inherited from the Default story in Task.stories.jsx.
        tasks: [
            { ...TaskStories.Default.args.task, id: "1", title: "Task 1" },
            { ...TaskStories.Default.args.task, id: "2", title: "Task 2" },
            { ...TaskStories.Default.args.task, id: "3", title: "Task 3" },
            { ...TaskStories.Default.args.task, id: "4", title: "Task 4" },
            { ...TaskStories.Default.args.task, id: "5", title: "Task 5" },
            { ...TaskStories.Default.args.task, id: "6", title: "Task 6" },
        ],
    },
};

export const WithPinnedTasks = {
    args: {
        tasks: [
            ...Default.args.tasks.slice(0, 5),
            { id: "6", title: "Task 6 (pinned)", state: "TASK_PINNED" },
        ],
    },
};

export const Loading = {
    args: {
        tasks: [],
        loading: true,
    },
};

export const Empty = {
    args: {
        // Shaping the stories through args composition.
        // Inherited data coming from the Loading story.
        ...Loading.args,
        loading: false,
    },
};
```

---

## 데이터 연결

-   위에서 작성한 `TaskList` 컴포넌트는 자체 구현 외에는 외부와 통신하지 않는 프레젠테이션 컴포넌트. 그러모르 데이터를 가져오려면 데이터 Provider와 연결해야함.
-   이 튜토리얼에서는 `@reduxjs/toolkit`과 `react-redux`를 사용.

```shell
$ npm install react-redux @reduxjs/toolkit
$ yarn add react-redux @reduxjs/toolkit
```

#### `store.js`

```javascript
// src/lib/store.js
/* A simple redux store/actions/reducer implementation.
 * A true app would be more complex and separated into different files.
 */
import { configureStore, createSlice } from "@reduxjs/toolkit";

/*
 * The initial state of our store when the app loads.
 * Usually, you would fetch this from a server. Let's not worry about that now
 */
const defaultTasks = [
    { id: "1", title: "Something", state: "TASK_INBOX" },
    { id: "2", title: "Something more", state: "TASK_INBOX" },
    { id: "3", title: "Something else", state: "TASK_INBOX" },
    { id: "4", title: "Something again", state: "TASK_INBOX" },
];
const TaskBoxData = {
    tasks: defaultTasks,
    status: "idle",
    error: null,
};

/*
 * The store is created here.
 * You can read more about Redux Toolkit's slices in the docs:
 * https://redux-toolkit.js.org/api/createSlice
 */
const TasksSlice = createSlice({
    name: "taskbox",
    initialState: TaskBoxData,
    reducers: {
        updateTaskState: (state, action) => {
            const { id, newTaskState } = action.payload;
            const task = state.tasks.findIndex((task) => task.id === id);
            if (task >= 0) {
                state.tasks[task].state = newTaskState;
            }
        },
    },
});

// The actions contained in the slice are exported for usage in our components
export const { updateTaskState } = TasksSlice.actions;

/*
 * Our app's store configuration goes here.
 * Read more about Redux's configureStore in the docs:
 * https://redux-toolkit.js.org/api/configureStore
 */
const store = configureStore({
    reducer: {
        taskbox: TasksSlice.reducer,
    },
});

export default store;
```

#### `TaskList.jsx`

```jsx
/// src/components/TaskList.jsx
import React from "react";
import Task from "./Task";
import { useDispatch, useSelector } from "react-redux";
import { updateTaskState } from "../lib/store";

export default function TaskList() {
    // We're retrieving our state from the store
    const tasks = useSelector((state) => {
        const tasksInOrder = [
            ...state.taskbox.tasks.filter((t) => t.state === "TASK_PINNED"),
            ...state.taskbox.tasks.filter((t) => t.state !== "TASK_PINNED"),
        ];
        const filteredTasks = tasksInOrder.filter(
            (t) => t.state === "TASK_INBOX" || t.state === "TASK_PINNED"
        );
        return filteredTasks;
    });

    const { status } = useSelector((state) => state.taskbox);

    const dispatch = useDispatch();

    const pinTask = (value) => {
        // We're dispatching the Pinned event back to our store
        dispatch(updateTaskState({ id: value, newTaskState: "TASK_PINNED" }));
    };
    const archiveTask = (value) => {
        // We're dispatching the Archive event back to our store
        dispatch(updateTaskState({ id: value, newTaskState: "TASK_ARCHIVED" }));
    };
    const LoadingRow = (
        <div className="loading-item">
            <span className="glow-checkbox" />
            <span className="glow-text">
                <span>Loading</span> <span>cool</span> <span>state</span>
            </span>
        </div>
    );
    if (status === "loading") {
        return (
            <div className="list-items" data-testid="loading" key={"loading"}>
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
                {LoadingRow}
            </div>
        );
    }
    if (tasks.length === 0) {
        return (
            <div className="list-items" key={"empty"} data-testid="empty">
                <div className="wrapper-message">
                    <span className="icon-check" />
                    <p className="title-message">You have no tasks</p>
                    <p className="subtitle-message">Sit back and relax</p>
                </div>
            </div>
        );
    }

    return (
        <div className="list-items" data-testid="success" key={"success"}>
            {tasks.map((task) => (
                <Task
                    key={task.id}
                    task={task}
                    onPinTask={(task) => pinTask(task)}
                    onArchiveTask={(task) => archiveTask(task)}
                />
            ))}
        </div>
    );
}
```

### 데코레이터로 컨텍스트 공급하기

-   위의 예제로 변경하면 스토리북 스토리가 더 이상 작동하지 않는데, `TaskList`가 이제 연결된 컴포넌트이기 때문에 `Task`를 검색하고 업데이트하는데 Redux 스토어에 의존하기 때문.

-   다양한 방법으로 해결할 수 있지만, 데코레이터를 사용해 스토리북 스토리에서 모킹 스토어를 제공할 수 있음.

-   [`excludeStories`](https://storybook.js.org/docs/react/api/csf#non-story-exports) 모킹된 상태가 스토리로 취급되지 않도록 하는 스토리북 구성 필드.

#### `TaskList.stories.jsx`

```jsx
// src/components/TaskList.stories.jsx
import TaskList from "./TaskList";
import * as TaskStories from "./Task.stories";
import { Provider } from "react-redux";
import { configureStore, createSlice } from "@reduxjs/toolkit";

// A super-simple mock of the state of the store
export const MockedState = {
    tasks: [
        { ...TaskStories.Default.args.task, id: "1", title: "Task 1" },
        { ...TaskStories.Default.args.task, id: "2", title: "Task 2" },
        { ...TaskStories.Default.args.task, id: "3", title: "Task 3" },
        { ...TaskStories.Default.args.task, id: "4", title: "Task 4" },
        { ...TaskStories.Default.args.task, id: "5", title: "Task 5" },
        { ...TaskStories.Default.args.task, id: "6", title: "Task 6" },
    ],
    status: "idle",
    error: null,
};

// A super-simple mock of a redux store
const Mockstore = ({ taskboxState, children }) => (
    <Provider
        store={configureStore({
            reducer: {
                taskbox: createSlice({
                    name: "taskbox",
                    initialState: taskboxState,
                    reducers: {
                        updateTaskState: (state, action) => {
                            const { id, newTaskState } = action.payload;
                            const task = state.tasks.findIndex(
                                (task) => task.id === id
                            );
                            if (task >= 0) {
                                state.tasks[task].state = newTaskState;
                            }
                        },
                    },
                }).reducer,
            },
        })}
    >
        {children}
    </Provider>
);

export default {
    component: TaskList,
    title: "TaskList",
    decorators: [(story) => <div style={{ padding: "3rem" }}>{story()}</div>],
    tags: ["autodocs"],
    // excludeStories는 모킹된 상태가 스토리로 취급되지 않도록 하는 스토리북 구성 필드.
    excludeStories: /.*MockedState$/,
};

export const Default = {
    decorators: [
        (story) => <Mockstore taskboxState={MockedState}>{story()}</Mockstore>,
    ],
};

export const WithPinnedTasks = {
    decorators: [
        (story) => {
            const pinnedtasks = [
                ...MockedState.tasks.slice(0, 5),
                { id: "6", title: "Task 6 (pinned)", state: "TASK_PINNED" },
            ];

            return (
                <Mockstore
                    taskboxState={{
                        ...MockedState,
                        tasks: pinnedtasks,
                    }}
                >
                    {story()}
                </Mockstore>
            );
        },
    ],
};

export const Loading = {
    decorators: [
        (story) => (
            <Mockstore
                taskboxState={{
                    ...MockedState,
                    status: "loading",
                }}
            >
                {story()}
            </Mockstore>
        ),
    ],
};

export const Empty = {
    decorators: [
        (story) => (
            <Mockstore
                taskboxState={{
                    ...MockedState,
                    tasks: [],
                }}
            >
                {story()}
            </Mockstore>
        ),
    ],
};
```

---

## 스크린 구성하기

-   [화면 연결하기](#화면-연결하기)

### 화면 연결하기
