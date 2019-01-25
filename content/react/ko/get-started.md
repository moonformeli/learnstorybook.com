---
title: "시작하기"
tocTitle: "시작하기"
description: "개발 환경에서 React Storybook 설치하기"
commit: a439924
---

# 시작하기

Storybook 은 여러분의 개발 환경과 같이 실행됩니다. 여러분의 어플리케이션의 어떠한 상황이나 환경과 분리된 채로 UI 컴포넌트를 
개발할 수 있도록 도와줄 것입니다. 지금 보고 계신 페이지는 React 사용자들을 위한 가이드 페이지입니다. 
다른 개발 환경으로는 [Vue](/vue/en/get-started) 와 [Angular](/angular/en/getstarted) 도 있습니다.

![Storybook and your app](/storybook-relationship.jpg)

## 리액트 Storybook 설치하기

이제부터 시작해봅시다. 개발 환경을 셋팅하려면 몇 가지 절차가 필요합니다. 시작하기에 앞서, 
빌드 시스템을 설치하기 위해 [Create React App](https://github.com/facebook/create-react-app) (CRA) 과
테스팅 환경에 필요한 [Jest](https://facebook.github.io/jest/), 마지막으로 [Storybook](https://storybook.js.org/) 을
설치해야합니다. 아래와 같이 커맨드를 입력하세요. 

```bash
# 어플리케이션 생성하기:
npx create-react-app taskbox
cd taskbox

# Storybook 추가:
npx -p @storybook/cli sb init
```

설치가 끝났다면 어플리케이션이 제대로 동작하는지 확인해볼 필요가 있습니다. 다음과 같이 입력하면 확인하실 수 있습니다.

```bash
# 터미널에서 테스트 러너 (Jest) 실행 :
yarn test

# 포트 9009번에서 컴포넌트 탐색기 시작
yarn run storybook

# 포트 3000번에서 프론트엔드 앱 실행
yarn start
```

<div class="aside">
  NOTE: 만약 <code>yarn test</code> 에서 오류가 나온다면, <a href="https://github.com/facebook/create-react-app/issues/871#issuecomment-252297884">가이드</a>를 참고해 <code>watchman</code>를 설치해보시는 것을 권장드립니다.
</div>

아래에는 다음과 같은 어플리케이션 활용 양식이 있습니다: 자동 테스트 툴 (Jest), 컴포넌트 개발 환경 (Storybook), 어플리케이션

![3 modalities](/app-three-modalities.png)

어떤 어플리케이션에서 작업을 하시게 될 지는 모르지만, 아마도 한 개 이상의 기능을 사용하시게 될 것이라 생각합니다. 그렇지만 지금 우리는 UI 컴포넌트에 대해서 알아보는 중이므로, 계속해서 Storybook 에 대해서 알아보겠습니다.

## 재사용 가능한 CSS

Taskbox 는 GraphQL과 React 튜토리얼 [example app](https://blog.hichroma.com/graphql-react-tutorial-part-1-6-d0691af25858)에서 설명하고 있는 디자인 요소들을 재활용하고 있기때문에 이 곳에서는 CSS를 처음부터 작성하지는 않을 것입니다. 대신 단일 CSS 파일에 LESS를 컴파일한 결과를 저장하고 어플리케이션에 적용해보겠습니다. [컴파일 된 CSS](https://github.com/hichroma/learnstorybook-code/blob/master/src/index.css)를 각 CRA 컨벤션의 src/index.css 파일마다 적용해주세요.

![Taskbox UI](/ss-browserchrome-taskbox-learnstorybook.png)

<div class="aside">
혹시라도 스타일을 변경하고 싶으시다면, Github 저장소에서 LESS 파일 소스를 제공받으실 수 있습니다.
</div>

## 정적 파일 추가

우리는 폰트와 아이콘 [디렉토리](https://github.com/hichroma/learnstorybook-code/tree/master/public) 를 `public/` 폴더에 추가해야 합니다. 스타일과 정적 파일들을 추가하고 나면 어플리케이션이 조금 낯설어 보일 수 있습니다. 하지만 아직은 어플리케이션에서 본격적으로 작업하는 것이 아니기때문에 괜찮습니다. 이제 첫 번째 컴포넌트를 만드는 것으로 시작해보겠습니다!
