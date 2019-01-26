---
title: "합성 컴포넌트 조립하기"
tocTitle: "합성 컴포넌트"
description: "간단한 컴포넌트들에서 합성 컴포넌트를 조립하기"
---

# 합성 컴포넌트 조립하기

지난 챕터에서 우리는 첫 번째 컴포넌트를 생성했습니다. 이번 챕터에서는 Task의 목록인 TaskList에 대해 좀 더 이야기하겠습니다. 컴포넌트들을 합쳐 복잡성이 증가했을 때 어떤 일이 발생하는지 살펴보겠습니다.

## 작업목록 - TaskList

작업목록(Taskbox)은 고정 작업들을 기본 작업들 위에 배치시켜서 눈에 띠게 해줍니다. 이러한 효과는 스토리를 만들기위한 작업 목록(기본 작업과 중요 작업을 포함한)에 두 가지 변화를 불러일으킵니다.

![default and pinned tasks](/tasklist-states-1.png)

`작업(Task)`은 데이터를 비동기적으로 가져오기때문에, 우리는 연결 상태가 좋지 않을 때 필요한 로딩 상태 **역시** 필요합니다. 게다가, 작업 목록이 하나도 없을 때 필요한 빈 상태도 필요합니다.

![empty and loading tasks](/tasklist-states-2.png)

## 구성하기

합성 컴포넌트는 기본 컴포넌트와 크게 다르지 않습니다. `TaskList`을 만들고 그에 맞는 스토리 파일도 만들어주세요.: `src/components/TaskList.js`, `src/components/TaskList.stories.js`.

`TaskList`의 실행부를 간단하게 구현해보는 것으로 시작합시다. `Task` 컴포넌트를 불러오고 속성과 액션을 입력값으로 전달합니다.

```javascript
import React from 'react';

import Task from './Task';

function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
  const events = {
    onPinTask,
    onArchiveTask,
  };

  if (loading) {
    return <div className="list-items">loading</div>;
  }

  if (tasks.length === 0) {
    return <div className="list-items">empty</div>;
  }

  return (
    <div className="list-items">
      {tasks.map(task => <Task key={task.id} task={task} {...events} />)}
    </div>
  );
}

export default TaskList;
```

다음 `TaskList`의 테스트 상태를 스토리 파일안에 만들겠습니다.

```javascript
import React from 'react';
import { storiesOf } from '@storybook/react';

import TaskList from './TaskList';
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
  .add('default', () => <TaskList tasks={defaultTasks} {...actions} />)
  .add('withPinnedTasks', () => <TaskList tasks={withPinnedTasks} {...actions} />)
  .add('loading', () => <TaskList loading tasks={[]} {...actions} />)
  .add('empty', () => <TaskList tasks={[]} {...actions} />);
```

`addDecorator()` 메소드는 렌더링되는 각 작업(task)에 "컨텍스트"를 추가할 수 있게합니다. 지금같은 경우는 시각적으로 더 눈에 잘 보이게 리스트에 padding을 주었습니다.

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#1-decorators"><b>데코레이터</b></a>는 스토리를 감싸는 임의의 포장지입니다. 위의 코드에서는 코드에 스타일을 적용하기 위해 데코레이터를 사용하고 있습니다. 데코레이터는 리액트 컨텍스트를 설정하는 라이브러리 컴포넌트와 같이 "providers" 내에서 사용될 수도 있습니다.
</div>

`task`는 `Task.stories.js` 파일에서 내보냈던 `작업(Task)`의 외형을 만듭니다. 비슷하게, `actions`는 `Task(작업)` 컴포넌트와 `TaskList(작업 목록)`가 기대하는 액션(임시로 만든 콜백 메소드들)을 정의합니다.  

이제 새로운 `TaskList` 스토리들의 스토리북을 확인해보겠습니다.

<video autoPlay muted playsInline loop>
  <source
    src="/inprogress-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

## 상태 생성하기

우리의 컴포넌트는 아직 어딘가 부족하지만 그래도 앞으로 어떻게 진행해야하는지에 대한 스토리의 개념은 이해하고 있습니다. 혹시라도 `.list-items` 클래스를 가진 태그가 너무 지나치게 간단하다고 생각하실 수도 있습니다. 맞습니다. 사실 거의 대부분의 경우엔 단순히 wrapper 태그를 만들기 위해 새로운 컴포넌트를 만들지 않습니다. 그러나 `TaskList` 컴포넌트의 **실제 복잡성**은 `withPinnedTasks`, `loading`, 그리고 `empty`와 같은 엣지케이스에서 나타납니다.

```javascript
import React from 'react';

import Task from './Task';

function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
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
      <div className="list-items">
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
      <div className="list-items">
        <div className="wrapper-message">
          <span className="icon-check" />
          <div className="title-message">You have no tasks</div>
          <div className="subtitle-message">Sit back and relax</div>
        </div>
      </div>
    );
  }

  const tasksInOrder = [
    ...tasks.filter(t => t.state === 'TASK_PINNED'),
    ...tasks.filter(t => t.state !== 'TASK_PINNED'),
  ];

  return (
    <div className="list-items">
      {tasksInOrder.map(task => <Task key={task.id} task={task} {...events} />)}
    </div>
  );
}

export default TaskList;
```

새로 추가된 마크업은 아래 UI에서 확인해보세요.

<video autoPlay muted playsInline loop>
  <source
    src="/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

리스트내의 고정 아이템의 위치에 대해 주목하세요. 우리는 사용자들을 위해 고정 아이템으로 선택된 아이템들을 상단에 렌더링하길 원합니다.

## 데이터 요구사항과 props

컴포넌트가 점점 커짐에 따라, 입력 값에 대한 요구사항 또한 많아지고 있습니다. `TaskList`의 prop 요구사항에 대해 정의하세요. `Task`는 자식 컴포넌트이기 때문에, 렌더링하기 위한 적절한 데이터를 보내고 있는지 확인하세요. 일전에 `Task` 안에서 정의한 propTypes를 사용한다면 머리를 쥐어뜯는 불필요한 시간을 절약할 수 있습니다.

```javascript
import React from 'react';
import PropTypes from 'prop-types';

function TaskList() {
  ...
}


TaskList.propTypes = {
  loading: PropTypes.bool,
  tasks: PropTypes.arrayOf(Task.propTypes.task).isRequired,
  onPinTask: PropTypes.func.isRequired,
  onArchiveTask: PropTypes.func.isRequired,
};

TaskList.defaultProps = {
  loading: false,
};

export default TaskList;
```

## 자동화 테스트

지난 챕터에서는 Storyshot을 이용해 스토리의 스냅샷 테스트를 하는 방법을 배웠습니다. `Task` 만 테스트했을 땐 제대로 렌더링이 되었는지 확인하는 것 외에는 그다지 테스트 할 것이 많지 않았습니다. 그러나 `TaskList`는 다른 레이어를 추가함으로써 복잡성이 증가했기때문에 특정한 입력에 특정한 결과가 제대로 출력이 되는지 익숙한 자동 테스트 방식으로 검사해보고 싶습니다. 이 방법을 위해선 [Enzyme](http://airbnb.io/enzyme/)와 같은 테스트 렌더러와 함께 [Jest](https://facebook.github.io/jest/)를 이용해 유닛 테스트를 만들 것입니다.

![Jest logo](/logo-jest.png)

### 유닛 테스트와 Jest

스토리북의 스토리들은 수동 테스트와 스냅샷 테스트(위 내용을 참고하세요)를 한 쌍으로 가져가 UI 버그를 멀리합니다. 만약 스토리들이 컴포넌트 유즈 케이스들을 포괄적으로 덮어준다면, 우리는 스토리 변화를 확인시켜줄 도구를 사용하게 되고, 에러는 더욱 줄어들 것입니다.

그러나, 때로는 골치아픈 일일 수 있습니다. 유닛 테스르르 위한 상세를 명확하게 해주어야 하는 테스트 프레임워크가 필요합니다.

우리가 속한 경우로 보자면, 우린 `TaskList`가 `tasks` prop으로 전달된 고정되지 않은 작업(task)들 **보다 먼저** 고정 작업(task)들을 렌더링하길 원합니다. 정확하게 같은 시나리오를 테스트 해줄 (`withPinnedTasks`) 스토리를 가지고 있지만 사람의 눈에는 이러한 렌더링 결과가 혹시라도 작업들의 우선순위를 정렬하는 행위를 **하지 않는 것**이 아닐까, 혹시라도 버그가 아닐까 하는 착각이 들게 할 수도 있습니다. 유닛 테스트는 적어도 **잘못 됐어!** 라고 소리는 지르지 않을테니 걱정마세요.

결국 이런 문제를 만나고 싶지 않다면 Jest를 사용해서 스토리를 DOM에 렌더링하고 DOM 쿼리를 수행하는 코드를 실행시켜 결과로 나와야 하는 값의 주요 스펙을 확인하세요.

`TaskList.test.js` 테스트 파일을 생성하세요. 다음은 결과에 대한 상세 내용을 설명한 테스트를 만들어 놓은 코드입니다.

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import TaskList from './TaskList';
import { withPinnedTasks } from './TaskList.stories';

it('리스트 첫 부분에는 고정 작업(task)들을 렌더링한다.', () => {
  const div = document.createElement('div');
  const events = { onPinTask: jest.fn(), onArchiveTask: jest.fn() };
  ReactDOM.render(<TaskList tasks={withPinnedTasks} {...events} />, div);

  // "Task 6 (pinned)" 타이틀을 가진 작업(task)이 마지막이 아닌 처음에 렌더링 되는 것이 결과의 기대값입니다.
  const lastTaskInput = div.querySelector('.list-item:nth-child(1) input[value="Task 6 (pinned)"]');
  expect(lastTaskInput).not.toBe(null);

  ReactDOM.unmountComponentAtNode(div);
});
```

![TaskList test runner](/tasklist-testrunner.png)

`withPinnedTasks` 를 스토리와 유닛 테스트 모두 재사용할 수 있었던 것에 주목하세요. 이런 방법은 실제 소스 코드(컴포넌트의 설정을 표현하는 예제)에 미치는 효력을 유지할 수 있게 해줍니다.

이 테스트는 단단하게 짜여진 테스트가 아니란 점도 주목하세요. 프로젝트가 커지고 `Task`의 실행부가 바뀌게된다면 --아마도 다른 다른 클래스명을 사용하거나 `input` 대신 `textarea`를 사용한다던지의 경우겠지요-- 테스트는 실패할 겁니다. 그리고 재조정이 필요하겠지요. 이게 꼭 문제라고 볼 필요는 없지만 UI에 대한 단위 테스트를 좀 더 주의해서 사용해야한다는 일종의 경고문으로 생각하시면 됩니다. 이러한 단위 테스트들은 관리하기가 쉽지는 않습니다. 되도록이면 시각적, 스냅샷 및 시각적 회귀 테스트([테스트 챕터](/test/) 참조)에 의존하세요.
