# 블루프린트 SDK

![블루프린트](\img\blueprint\logo.svg)

스마트 계약을 작성, 테스트 및 배포하기 위한 TON 개발 환경.

## 빠른 시작 🚀

터미널에서 다음 명령을 실행하여 새 프로젝트를 만들고 화면 안내에 따르세요:

```bash
npm create ton@latest
```

### 핵심 기능

- 스마트 계약 빌드, 테스트 및 배포를 위한 간소화된 워크플로우
- 원하는 지갑 (예: Tonkeeper)을 사용하여 메인넷/테스트넷으로 간단한 배포
- 독립된 프로세스에서 실행 중인 격리된 블록체인에서 여러 스마트 계약을 빠르게 테스트

### 기술 스택

- https://github.com/ton-community/func-js를 사용하여 FunC 컴파일 (CLI 없음)
- https://github.com/ton-org/sandbox를 사용하여 스마트 계약 테스트
- TON Connect 2.0 호환 지갑 또는 `ton://` 딥링크를 사용하여 스마트 계약 배포

### 요구 사항

- [Node.js](https://nodejs.org/): 최신 버전 (예: v18)과 함께 사용, 버전 확인은 `node -v`로 가능
- TypeScript 및 FunC 지원이 있는 IDE (예: [Visual Studio Code](https://code.visualstudio.com/)) 및 [FunC 플러그인](https://marketplace.visualstudio.com/items?itemName=tonwhales.func-vscode)이 설치된 환경

## 참고 자료

### GitHub

- https://github.com/ton-org/blueprint

### 자료

- [DoraHacks 스트림에서 사용하는 Blueprint](https://www.youtube.com/watch?v=5ROXVM-Fojo)
- [새 프로젝트 만들기](https://github.com/ton-org/blueprint#create-a-new-project)
- [새로운 스마트 계약 개발](https://github.com/ton-org/blueprint#develop-a-new-contract)
- [[YouTube] 블루프린트와 함께하는 FunC EN](https://www.youtube.com/watch?v=7omBDfSqGfA&list=PLtUBO1QNEKwtO_zSyLj-axPzc9O9rkmYa) ([RU 버전](https://youtube.com/playlist?list=PLyDBPwv9EPsA5vcUM2vzjQOomf264IdUZ))

## 관련 정보

- [스마트 계약 소개](/develop/smart-contracts/)
- [지갑 스마트 계약 사용 방법](/develop/smart-contracts/tutorials/wallet)
- [toncli 사용하기](/develop/smart-contracts/sdk/toncli)
- [SDKs](/develop/dapps/apis/sdk)
