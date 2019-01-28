---
title: "데이터 연결하기"
tocTitle: "데이터"
description: "UI 컴포넌트에 데이터 연결하기"
---

# 데이터 연결하기

지금까지 우리는 stateless한 컴포넌트들을 만들었습니다. (스토리북에 적합하지만 어플리케이션에 데이터를 넣어주지 않는 한 그다지 유용하다고는 볼 수는 없습니다.)

이 튜토리얼은 개별적인 어플리케이션을 만드는 과정에 지나치게 몰두하지는 않을 것입니다. 그 대신 흔히 사용되는 컨테이너 컴포넌트에 데이터를 연결하는 패턴을 살펴볼 것입니다.

## 컨테이너 컴포넌트

우리의 `TaskList` 컴포넌트는 현재 껍데기에 불과한 "피상적인" 형태에 가깝습니다. ([블로그를 참고](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)하세요) 아직 이 컴포넌트는 외부의 다른 누구에게도 실행 코드에 대한 어떤 것도 설명해주고 있지 않습니다. 데이터를 전달해주기 위해서는, "컨테이너"라고 부르는 것이 필요합니다.

이 예제는 리액트의 라이브러리 중 데이터 정렬로 가장 유명한 [리덕스](https://redux.js.org/)를 사용합니다. 우리의 어플리케이션의 데이터 모델을 좀 더 심플하게 가져갈 수 있는 좋은 라이브러리입니다. 하지만, 이 곳에서 사용되는 패턴은 다른 데이터 상태관리 라이브러리들과 크게 다르지 않습니다. 예를 들면 [Apollo](https://www.apollographql.com/client/) 와 [MobX](https://mobx.js.org/)입니다.

새로운 디펜던시 정보를 `package.json`에 추가하세요.

```bash
yarn add react-redux
```

먼저 작업(task)들의 상태 변화에 대해 대응할 액션(action)을 갖는 리덕스 스토어를 `src`폴더 안에 `lib/redux.js`에 만들겠습니다. (계속해서 간결해지고자 함이 이 작업의 목적입니다.)

```javascript
// 리덕스 스토어/액션/리듀서 실행의 간단한 예제입니다.
// 실제 어플리케이션은 더 복잡하고 각기 다른 파일로 나뉘어져 있을겁니다.
import { createStore } from 'redux';

// 액션은 스토어에서 어떠한 변화를 가리키는 "이름"들로 구분되어 있습니다.
export const actions = {
  ARCHIVE_TASK: 'ARCHIVE_TASK',
  PIN_TASK: 'PIN_TASK',
};

// The action creators are how you bundle actions with the data required to execute them
export const archiveTask = id => ({ type: actions.ARCHIVE_TASK, id });
export const pinTask = id => ({ type: actions.PIN_TASK, id });

// All our reducers simply change the state of a single task.
function taskStateReducer(taskState) {
  return (state, action) => {
    return {
      ...state,
      tasks: state.tasks.map(
        task => (task.id === action.id ? { ...task, state: taskState } : task)
      ),
    };
  };
}

// The reducer describes how the contents of the store change for each action
export const reducer = (state, action) => {
  switch (action.type) {
    case actions.ARCHIVE_TASK:
      return taskStateReducer('TASK_ARCHIVED')(state, action);
    case actions.PIN_TASK:
      return taskStateReducer('TASK_PINNED')(state, action);
    default:
      return state;
  }
};

// The initial state of our store when the app loads.
// Usually you would fetch this from a server
const defaultTasks = [
  { id: '1', title: 'Something', state: 'TASK_INBOX' },
  { id: '2', title: 'Something more', state: 'TASK_INBOX' },
  { id: '3', title: 'Something else', state: 'TASK_INBOX' },
  { id: '4', title: 'Something again', state: 'TASK_INBOX' },
];

// We export the constructed redux store
export default createStore(reducer, { tasks: defaultTasks });
```

Then we’ll update the default export from the `TaskList` component to connect to the Redux store and render the tasks we are interested in:

```javascript
import React from 'react';
import PropTypes from 'prop-types';

import Task from './Task';
import { connect } from 'react-redux';
import { archiveTask, pinTask } from '../lib/redux';

export function PureTaskList({ loading, tasks, onPinTask, onArchiveTask }) {
  /* previous implementation of TaskList */
}

PureTaskList.propTypes = {
  loading: PropTypes.bool,
  tasks: PropTypes.arrayOf(Task.propTypes.task).isRequired,
  onPinTask: PropTypes.func.isRequired,
  onArchiveTask: PropTypes.func.isRequired,
};

PureTaskList.defaultProps = {
  loading: false,
};

export default connect(
  ({ tasks }) => ({
    tasks: tasks.filter(t => t.state === 'TASK_INBOX' || t.state === 'TASK_PINNED'),
  }),
  dispatch => ({
    onArchiveTask: id => dispatch(archiveTask(id)),
    onPinTask: id => dispatch(pinTask(id)),
  })
)(PureTaskList);
```

At this stage our Storybook tests will have stopped working, as the `TaskList` is now a container, and no longer expects any props, instead it connects to the store and sets the props on the `PureTaskList` component it wraps.

However, we can easily solve this problem by simply rendering the `PureTaskList` --the presentational component-- in our Storybook stories:

```javascript
import React from 'react';
import { storiesOf } from '@storybook/react';

import { PureTaskList } from './TaskList';
import { task, actions } from './Task.stories';

export const defaultTasks = [
  { ...task, id: '1', title: 'Task 1' },
  { ...task, id: '2', title: 'Task 2' },
  { ...task, id: '3', title: 'Task 3' },
  { ...task, id: '4', title: 'Task 4' },
  { ...task, id: '5', title: 'Task 5' },
  { ...task, id: '6', title: 'Task 6' },
];

export const withPinnedTasks = [
  ...defaultTasks.slice(0, 5),
  { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
];

storiesOf('TaskList', module)
  .addDecorator(story => <div style={{ padding: '3rem' }}>{story()}</div>)
  .add('default', () => <PureTaskList tasks={defaultTasks} {...actions} />)
  .add('withPinnedTasks', () => <PureTaskList tasks={withPinnedTasks} {...actions} />)
  .add('loading', () => <PureTaskList loading tasks={[]} {...actions} />)
  .add('empty', () => <PureTaskList tasks={[]} {...actions} />);
```

<video autoPlay muted playsInline loop>
  <source
    src="/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

Similarly, we need to use `PureTaskList` in our Jest test:

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { PureTaskList } from './TaskList';
import { withPinnedTasks } from './TaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  const div = document.createElement('div');
  const events = { onPinTask: jest.fn(), onArchiveTask: jest.fn() };
  ReactDOM.render(<PureTaskList tasks={withPinnedTasks} {...events} />, div);

  // We expect the task titled "Task 6 (pinned)" to be rendered first, not at the end
  const lastTaskInput = div.querySelector('.list-item:nth-child(1) input[value="Task 6 (pinned)"]');
  expect(lastTaskInput).not.toBe(null);

  ReactDOM.unmountComponentAtNode(div);
});
```
