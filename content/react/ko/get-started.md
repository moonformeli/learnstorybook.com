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
  NOTE: If <code>yarn test</code> throws an error, you may need to install <code>watchman</code> as advised in <a href="https://github.com/facebook/create-react-app/issues/871#issuecomment-252297884">this issue</a>.
</div>

Our three frontend app modalities: automated test (Jest), component development (Storybook), and the app itself.

![3 modalities](/app-three-modalities.png)

Depending on what part of the app you’re working on, you may want to run one or more of these simultaneously. Since our current focus is creating a single UI component, we’ll stick with running Storybook.

## Reuse CSS

Taskbox reuses design elements from the GraphQL and React Tutorial [example app](https://blog.hichroma.com/graphql-react-tutorial-part-1-6-d0691af25858), so we won’t need to write CSS in this tutorial. We’ll simply compile the LESS to a single CSS file and include it in our app. Copy and paste [this compiled CSS](https://github.com/hichroma/learnstorybook-code/blob/master/src/index.css) into the src/index.css file per CRA’s convention.

![Taskbox UI](/ss-browserchrome-taskbox-learnstorybook.png)

<div class="aside">
If you want to modify the styling, the source LESS files are provided in the GitHub repo.
</div>

## Add assets

We also need to add the font and icon [directories](https://github.com/hichroma/learnstorybook-code/tree/master/public) to the `public/` folder. After adding styling and assets, the app will render a bit strangely. That’s OK. We aren’t working on the app right now. We’re starting off with building our first component!
