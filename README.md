# 첫번째 - 수동으로 npm package를 만들기

## TypeScript 설정 파일 (tsconfig.json) 구성

### 1. TypeScript 설치

```bash
npm install --save-dev typescript
```

- 개발 의존성으로 설치 (`-save-dev`)

### 2. tsconfig.json 설정

```json
{
  "compilerOptions": {
    "declaration": true, // 선언 파일(.d.ts) 생성
    "declarationDir": "lib/types", // 선언 파일 저장 위치
    "target": "ES6", // 트랜스파일 타겟
    "moduleResolution": "node" // 모듈 해석 방식
  },
  "include": [
    "src/**/*" // src 폴더의 모든 파일 포함
  ],
  "exclude": [
    "node_modules", // node_modules 제외
    "dist" // 빌드 결과물 제외
  ]
}
```

### 3. 주요 설정 설명

1. **컴파일러 옵션**
   - `declaration`: 타입 선언 파일 생성 여부
   - `declarationDir`: 선언 파일 저장 경로
   - `target`: JavaScript 변환 목표 버전
   - `moduleResolution`: 모듈 해석 방식
2. **파일 관리**
   - `include`: TypeScript가 처리할 파일 지정
   - `exclude`: 처리에서 제외할 파일 지정

### 4. 구조

- `dist/`: 빌드 결과물 저장 폴더
- `src/`: 소스 코드 폴더
- `dist/types/`: 타입 선언 파일 저장 폴더

이러한 설정으로 TypeScript 컴파일러가 코드를 올바르게 처리하고 필요한 결과물을 생성할 수 있습니다.

### 모듈 해석 방식

TypeScript의 `moduleResolution` 옵션에는 주로 다음과 같은 값들이 사용됩니다:

1. `"node"` (가장 일반적)

- Node.js의 모듈 해석 방식을 따릅니다
- 예시 파일 경로: `/root/src/module.ts`에서 `import { foo } from "bar"` 할 때
- 검색 순서:
  1. `/root/src/node_modules/bar`
  2. `/root/node_modules/bar`
  3. `/node_modules/bar`

1. `"classic"` (레거시)

- TypeScript의 이전 해석 방식
- 상대 경로만을 사용하며, 검색 범위가 제한적
- 현재는 거의 사용되지 않음

1. `"bundler"` (최신)

- Vite, webpack, esbuild 같은 번들러에 최적화된 방식
- `package.json`의 `exports` 필드를 지원
- 조건부 내보내기 지원

1. `"node16"` 또는 `"nodenext"`

- Node.js의 ECMAScript 모듈 시스템(ESM) 지원
- `.mjs`, `.cjs` 확장자 인식
- `package.json`의 `type` 필드 지원

사용 예시와 차이점:

```tsx
// node 방식
import { foo } from "some-package"; // 작동
import { bar } from "./local-module"; // 작동

// classic 방식
import { foo } from "some-package"; // 제한적 작동
import { bar } from "./local-module.ts"; // .ts 확장자 필요할 수 있음

// bundler 방식
import { foo } from "some-package/styles"; // package.json exports 필드 지원
import { bar } from "./local-module?raw"; // 쿼리 파라미터 지원

// node16/nodenext 방식
import { foo } from "some-package.mjs"; // ESM 확장자 필요
import { bar } from "./local-module.js"; // .js 확장자 필요
```

주의사항:

- 프로젝트의 환경과 목적에 따라 적절한 값을 선택해야 합니다
- 최신 웹 개발에서는 `"bundler"`나 `"node16"`을 권장
- 레거시 프로젝트가 아니라면 `"classic"`은 피하는 것이 좋습니다
- 대부분의 프로젝트에서는 `"node"`가 안전한 기본값입니다

## Rollup 설정과 패키지 빌드 구성

### 1. rollup.config.js 설정

```jsx
import typescript from "rollup-plugin-typescript2";
import del from "rollup-plugin-delete";

export default {
  input: "src/index.ts", // 시작점
  output: [
    {
      file: "dist/index.cjs", // CJS 버전 출력
      format: "cjs",
    },
    {
      file: "dist/index.esm.js", // ESM 버전 출력
      format: "esm",
    },
  ],
  plugins: [
    del({ targets: "dist/*" }), // 기존 빌드 삭제
    typescript({
      useTsconfigDeclarationDir: true, // tsconfig의 선언 디렉토리 사용
    }),
  ],
};
```

### 2. package.json 수정

```json
{
  "type": "module", // ESM 모듈 타입 지정
  "scripts": {
    "build": "rollup -c" // 빌드 스크립트 추가
  }
}
```

### 3. 주요 설정 설명

1. **입력 설정**
   - `src/index.ts`를 시작점으로 지정
   - 모든 메서드를 재익스포트하는 파일
2. **출력 설정**
   - CJS 포맷: `dist/index.cjs`
   - ESM 포맷: `dist/index.esm.js`
3. **플러그인**
   - `rollup-plugin-delete`: 이전 빌드 파일 정리
   - `rollup-plugin-typescript2`: TypeScript 컴파일

### 4. 빌드 실행

```bash
npm run build
```

이 설정으로 하나의 소스코드로부터 CJS와 ESM 두 가지 포맷의 빌드 결과물을 생성할 수 있습니다.

## NPM 패키지의 진입점(Entry Points) 설정 가이드

### 1. 기존 방식

패키지는 세 가지 주요 필드를 통해 접근했습니다:

1. `main` 필드: CJS(CommonJS) 버전용
   - 예: `"main": "lib/index.cjs"`
2. `module`/`browser` 필드: ESM(ECMAScript Modules) 버전용
   - 예: `"module": "lib/index.esm.js"`
3. `types` 필드: TypeScript 타입 선언 파일용
   - 예: `"types": "lib/types/index.d.ts"`

### 2. 현대적 방식 (exports 필드)

`exports` 필드를 사용한 현대적인 설정:

```json
{
  "main": "dist/index.cjs", // 레거시 지원용
  "exports": {
    "import": {
      "default": "./dist/index.esm.js",
      "types": "./dist/types/index.d.ts"
    },
    "require": {
      "default": "./dist/index.cjs",
      "types": "./dist/types/index.d.ts"
    }
  }
}
```

### 3. exports 필드의 주요 장점

1. **모듈 캡슐화**
   - `exports`에 명시된 파일만 외부에서 접근 가능
   - 내부 모듈을 효과적으로 숨길 수 있음
2. **환경별 설정**
   - ESM, CJS 각각에 대한 진입점 별도 정의
   - 타입 정의 파일도 각 환경에 맞게 지정
3. **하위 호환성**
   - `main` 필드는 Node.js 10 이하 버전 지원용으로 유지
   - 최신 Node.js에서는 `exports` 필드가 우선순위

이러한 설정으로 패키지 사용자들이 환경에 맞는 올바른 버전의 코드를 사용할 수 있게 됩니다.

## npm 패키지 배포 시 파일 관리

### 1. files 필드의 역할

- `package.json`의 `files` 필드는 배포할 파일들을 지정
- `npm publish` 실행 시 포함될 파일들을 결정
- 기본적으로는 루트 디렉토리의 모든 파일이 포함됨

### 2. 항상 포함되는 파일들

무조건 패키지에 포함되는 파일들:

- `package.json`
- `README` 파일
- `LICENSE` 파일

### 3. 배포 전 확인 방법

```bash
npm publish --dry-run

```

- 실제 배포 전에 어떤 파일들이 포함될지 미리 확인 가능
- 민감한 정보(API 키, 인증정보 등)가 실수로 포함되지 않았는지 확인 가능

### 4. files 필드 설정 예시

```json
{
  "files": ["dist"]
}
```

- 빌드된 결과물이 있는 `dist` 폴더만 포함
- 이렇게 하면 실제 배포 시 필요한 파일들만 포함됨:
  - 번들된 파일들
  - 타입 선언 파일들
  - 기본 포함 파일들(package.json, README, LICENSE)

이러한 설정을 통해 불필요한 파일은 제외하고, 필요한 파일만 효율적으로 배포할 수 있습니다.

## npm 패키지 배포를 위한 계정 설정

### 1. npm 레지스트리 이해

- npm 레지스트리는 패키지들을 저장하고 공유하는 데이터베이스
- 주요 레지스트리 옵션들:
  - npmjs.com (기본/공식 레지스트리)
  - GitHub npm 레지스트리
  - AWS CodeArtifact
  - 기타 private 레지스트리들

### 2. 계정 생성 프로세스

1. npmjs.com 회원가입
   - 고유한 사용자명 필수
   - 이메일 주소
   - 비밀번호 설정
2. 보안 절차
   - 챌린지 해결 (보안 검증)
   - 이메일로 받은 OTP(일회용 비밀번호) 입력
3. 로그인 완료

## npm 패키지 배포 프로세스 정리

### 1. npm 터미널 로그인 절차

```bash
# 현재 로그인 상태 확인
npm whoami

# 로그인
npm login
# - 사용자명 입력
# - 비밀번호 입력
# - 이메일 입력
# - OTP 입력 (이메일로 받은 코드)

```

### 2. 패키지 설정

package.json 수정:

```json
{
  "name": "@username/package-name", // 스코프 패키지 이름
  "version": "1.0.0" // 버전 설정
}
```

### 3. 패키지 배포

```bash
npm publish --access public

```

- `-access public` 플래그는 스코프 패키지를 공개로 설정
- 기본적으로 스코프 패키지는 private으로 설정됨

### 4. 코드 관리

```bash
git add .
git commit -m "first package release"
git push origin main

```

### 💡 개선 가능한 부분들

1. **자동화 필요성**
   - 현재는 수동 프로세스
   - 배포 프로세스 자동화 도구 활용 가능
2. **문서화 개선**
   - Changelog 필요
   - Git 태그 관리
   - 릴리스 문서화
3. **향후 개선 방향**
   - 더 강력하고 자동화된 배포 프로세스 구현
   - 전문 도구들을 활용한 배포 관리
   - 체계적인 버전 관리
