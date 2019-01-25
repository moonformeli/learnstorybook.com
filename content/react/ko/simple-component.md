---
title: "간단한 컴포넌트 제작하기"
tocTitle: "간단한 컴포넌트"
description: "간단한 컴포넌트를 제작해봅시다"
commit: 
---

# 간단한 컴포넌트 제작하기

우리는 [컴포넌트 주도 개발](https://blog.hichroma.com/component-driven-development-ce1109d56c8e) (CDD) 에 입각해 UI를 만들어 볼 것입니다. CDD는 아주 작은 컴포넌트부터 만들어 하나의 거대한 컴포넌트에 이르기까지 조금씩 조합해나가는 UI 컴포넌트 설계론입니다. CDD는 여러분들이 UI를 설계 시 따라오는 복잡성에 대한 실마리를 제공해줍니다.

## Task

![3가지 상태에서의 task 컴포넌트](/task-states-learnstorybook.png)

`Task` 는 우리의 어플리케이션에서 코어 컴포넌트에 해당됩니다. 각각의 task는 어떤 상태값을 가지냐에 따라 화면에 조금씩 다르게 표현합니다. 지금부터 체크 가능한 (체크 해제도 가능한) 체크박스, task에 대한 정보, "pin" 버튼을 표시해보겠습니다. 이 컴포넌트들은 리스트 내에서 순서 변경이 가능합니다. 우선, 아래와 같은 props들이 필요합니다.

* `title` – task 를 설명하는 string 값
* `state` - '어떤 리스트가 task에 해당하고 체크 온(오프)가 되었는지'에 대한 값

먼저 위의 스케치에서 본 다양한 종류의 `Task` 에 부합하는 테스트 상태를 작성해보겠습니다. 조금 뒤에는 Storybook 을 사용해 컴포넌트를 목업 데이터를 결합시켜 만들어보겠습니다. 또한 "비주얼 테스트" 를 통해 우리가 의도한 대로 컴포넌트가 맞게 그려졌는지 테스트 해볼 것입니다.

이 과정은 "[비주얼 TDD](https://blog.hichroma.com/visual-test-driven-development-aec1c98bed87)" 라고도 부르는 [테스트 주도 개발](https://en.wikipedia.org/wiki/Test-driven_development) 과도 비슷합니다.

## 설치하기

가장 먼저 task 컴포넌트와 작업에 같이 필요한 `src/components/Task.js`, `src/components/Task.stories.js` 스토리 파일을 생성합니다.

간단한 `Task` 의 구현부를 작성해봅시다. 간단한 속성 값들과 task 에 적용할 몇 가지 액션들을 가져왔습니다.

```javascript
import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```

Todos 어플리케이션의 HTML 구조를 기반으로 `Task` 컴포넌트의 마크업을 렌더링했습니다.

다음은 `Task` 에서 다루고 있는 story 파일 내의 3 가지 테스트 상대값에 대해 정의해 놓았습니다.

```javascript
import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

import Task from './Task';

export const task = {
  id: '1',
  title: 'Test Task',
  state: 'TASK_INBOX',
  updatedAt: new Date(2018, 0, 1, 9, 0),
};

export const actions = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

storiesOf('Task', module)
  .add('default', () => <Task task={task} {...actions} />)
  .add('pinned', () => <Task task={{ ...task, state: 'TASK_PINNED' }} {...actions} />)
  .add('archived', () => <Task task={{ ...task, state: 'TASK_ARCHIVED' }} {...actions} />);
```

Stroybook 에는 기본적으로 기초 수준의 조직 단위가 있습니다. 컴포넌트와 자식 story 객체들입니다. 각 story 들이 어떤 컴포넌트의 순열 형태를 이루고 있다고 생각해보세요. 컴포넌트당 원하는 만큼의 story 를 생성할 수 있습니다.

* **Component**
  * Story
  * Story
  * Story

Storybook 을 시작하기 위해선 먼저 `storiesOf()` 메소드를 호출해 컴포넌트에 등록합니다. 컴포넌트의 이름도 등록해 줍니다. 이 이름은 Storybook 어플리케이션의 사이드 바에 나타날 이름입니다.

`action()` 메소드는 Storybook 의 UI 화면상의 **actions** 패널의 클릭이 감지가 되었을 때 파라미터로 넘겨줄 수 있는 콜백을 정의할 수 있도록 해줍니다. 덕분에 우리가 핀 버튼을 만들면, 테스트 UI 내에서 클릭이 성공적으로 실행되는지를 판단할 수 있습니다.

컴포넌트의 모든 순열에 같은 액션들을 전달해야하기 때문에, `actions` 들을 하나로 묶어 React의 `{...actions}` props를 이용한다면 보다 간편하게 한 번에 전송할 수 있습니다. 예로, `<Task {...actions}>` 는 `<Task onPinTask={actions.onPinTask} onArchiveTask={actions.onArchiveTask}>` 와 같습니다.

컴포넌트가 필요로하는 `actions`들을 하나로 묶었을 때의 얻을 수 있는 또 다른 장점은 여러 story에서 컴포넌트로 재사용성이 좋고 `export` 를 통해 내보낼 수 있다는 점이 있습니다. 이 것은 조금 후에 다시 살펴보겠습니다.

story 들을 정의하기 위해선 각 테스트 상태값을 위해 `add()` 메소드를 한 번씩 실행줘야합니다. 액션 story 는 주어진 상태 값 안에서 렌더링 된 엘리먼트들(예를 들면 props 세트를 가진 컴포넌트 클래스)을 반환하는 메소드입니다. 이 것은 [무상태 함수형 컴포넌트(SFC)](https://reactjs.org/docs/components-and-props.html)와 정말 비슷합니다.

스토리를 만들 때, 우리는 기본 작업인 (`task`) 를 사용해 컴포넌트가 바라는 작업의 모양으로 만듭니다. 이 것은 실제 데이터가 어떻게 생겼는지를 기반으로 모델링 됩니다. 다시 한 번, `export` 를 통해 내보내는 것은 앞으로 이제 우리가 들여다보겠지만 추후에 언제든지 다시 사용할 수 있도록 해줍니다.

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#2-native-addons"><b>액션</b></a>들은 격리 된 UI 구성 요소를 작성할 때 요소와의 상호작용을 확인하는데 도움을 줍니다. 여러분은 대부분 함수나 상태에 직접 접근하는 것이 아니라 <code>action()</code>을 사용해서 가져오게 될 것입니다.
</div>

## 설정

`.stories.js` 파일들과 CSS 파일들을 인식하게 만드려면 Storybook 환경 설정 파일(`.storybook/config.js`)을 조금 수정해야합니다. 기본적으로 스토리들을 `/stories` 디렉토리에서 찾긴 합니다만 지금의 튜토리얼은 자동 테스트를 위한 CRA에서 선호하는 `.test.js`와 같은 네이밍 스키마를 사용하고 있기 때문에 변경을 해주어야 합니다.

```javascript
import { configure } from '@storybook/react';
import '../src/index.css';

const req = require.context('../src', true, /.stories.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

완료가 됐으면, 스토리북 서버를 재시작하면 화면에 3가지의 작업 상태 목록이 보일 것입니다.

<video autoPlay muted playsInline controls >
  <source
    src="/inprogress-task-states.mp4"
    type="video/mp4"
  />
</video>

## 상태 값 생성하기

이제 스토리북도 설치가 되었고, 스타일도 불러왔고 테스트 케이스들도 만들어졌으니 디자인에 맞는 컴포넌트의 마크업 렌더링을 신속하게 실행할 수 있습니다.

하지만 아직까지 우리의 컴포넌트는 아무것도 없는 빈 상태나 마찬가지입니다. 우선 너무 깊숙하게는 들어가지 않는 선에서 디자인적인 면을 고려해 작성해보겠습니다.

```javascript
import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className={`list-item ${state}`}>
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={state === 'TASK_ARCHIVED'}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" onClick={() => onArchiveTask(id)} />
      </label>
      <div className="title">
        <input type="text" value={title} readOnly={true} placeholder="Input title" />
      </div>

      <div className="actions" onClick={event => event.stopPropagation()}>
        {state !== 'TASK_ARCHIVED' && (
          <a onClick={() => onPinTask(id)}>
            <span className={`icon-star`} />
          </a>
        )}
      </div>
    </div>
  );
}
```

불러온 CSS를 사용해 마크업을 추가로 작성하면 아래와 같은 UI가 보이게 됩니다.

<video autoPlay muted playsInline loop>
  <source
    src="/finished-task-states.mp4"
    type="video/mp4"
  />
</video>

## 데이터 요구사항을 명확히

React에서는 `propTypes`를 사용하는 것이 컴포넌트가 원하는 데이터의 성질을 구체적으로 표현할 수 있어서 가장 좋은 연습입니다. 그런데 단순히 문서와 같은 기록면에서만 좋은게 아니라 오류 검증 또한 조기에 할 수 있습니다.

```javascript
import React from 'react';
import PropTypes from 'prop-types';

function Task() {
  ...
}

Task.propTypes = {
  task: PropTypes.shape({
    id: PropTypes.string.isRequired,
    title: PropTypes.string.isRequired,
    state: PropTypes.string.isRequired,
  }),
  onArchiveTask: PropTypes.func,
  onPinTask: PropTypes.func,
};

export default Task;
```

이제 Task 컴포넌트가 잘못 사용되면 경고 메시지가 나타날 것입니다.

<div class="aside">
자바스크립트의 타입 시스템인 타입스크립트를 사용해 컴포넌트 프로퍼티들을 검사하면 비슷한 효과를 얻을 수 있습니다.
</div>

## 컴포넌트 빌드!

우리는 지금까지 어떠한 서버 자원이나 프론트엔드 어플리케이션의 도움 없이 성공적으로 컴포넌트를 만들었습니다. 다음 단계는 남은 Taskbox 컴포넌트들을 비슷한 패턴으로 하나씩 만드는 일입니다.

보시다시피, 어플리케이션과 격리된 환경에서 컴포넌트를 만드는 것은 쉽고 빠릅니다. 모든 가능한 상태에 대해 깊고 면밀히 테스트를 하는 것이 가능하기 때문에 더 적은 버그를 가진, 그러나 더 높은 퀄리티의 UI를 생산할 수 있음을 기대하고 있습니다.

## 자동화 테스트

스토리북은 어플리케이션이 만들어지는 도중에도 시각적으로 테스트를 할 수 있는 훌륭한 길을 제시합니다. 이 '스토리' 라고 하는 것은 설령 어플리케이션을 계속 개발하고 있다하더라도 우리가 만든 작업을 시각적으로 망가뜨리지 않을 것입니다. 그래도, 아직까지는 철저한 수동 작업을 동반하기때문에, 에러나 버그 없이 화면에 잘 나오는지 확인하기 위해 누군가는 계속 마우스를 클릭해야 할지도 모릅니다. 어떻게 자동으로 좀 안될까요?

### 스냅샷 테스트

스냅샷 테스트는 외부에서 입력 값을 받고 언제라도 상태가 변하면 컴포넌트의 상태를 플래그로 저장하고 그 결과를 "상태 양호" 로 기록할 수 있는 좋은 시스템이라고 보시면 됩니다. 컴포넌트의 새로운 버젼을 빠르게 확인할 수 있고 변화를 확인할 수 있기 때문에 스냅샷 테스트야 말로 스토리북을 더욱 완성도 있게 만들어주는 요소입니다.

<div class="aside">
스냅샷 테스트가 매번 실패하지 않으려면 컴포넌트가 지속적으로 변하는 데이터를 렌더링하고 있는지 주의하세요. 날짜 값이나 랜덤한 값들은 특히 더욱 주의해야 합니다.
</div>

각 스토리들을 위한 스냅샷 테스트 애드온 중에는 [스토리샷 애드온](https://github.com/storybooks/storybook/tree/master/addons/storyshots)이란 것도 있습니다. 패키지의 development dependency에 추가해 사용하면 됩니다.

```bash
yarn add --dev @storybook/addon-storyshots react-test-renderer require-context.macro
```

그리고 `src/storybook.test.js` 파일을 만들어 아래 내용을 넣으세요.

```javascript
import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```

`require.context`가 Jest환경에서 제대로 실행되려면 [바벨 매크로](https://github.com/kentcdodds/babel-plugin-macros)를 사용할 것을 권장합니다. `.storybook/config.js`의 내용을 아래와 같이 변경하세요.

```js
import { configure } from '@storybook/react';
import requireContext from 'require-context.macro';

import '../src/index.css';

const req = requireContext('../src/components', true, /\.stories\.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

(`require.context`로 작성된 부분을 매크로에서 가져온 `requireContext`를 호출하는 방식으로 변경했습니다.)

코드를 수정했다면 `yarn test`를 실행해 다음과 같은 결과가 나오는지 확인하세요.

![Task test runner](/task-testrunner.png)

이제 각 `Task`(작업) 스토리들의 스냅샷 테스트를 생성했습니다. 만약 `Task`의 실행에 대한 내용을 변경한다면 관련 변경 사항들에 대해 즉시 알 수 있게 될 것입니다.
