# TypeScript Website - Playground Code Structure Analysis

## Overview

이 문서는 TypeScript Website 저장소의 코드 구조를 분석하며, 특히 Playground 관련 컴포넌트에 대해 상세히 설명합니다.

## Repository Structure

### Monorepo Architecture
- **Package Manager**: pnpm workspaces
- **Structure**: 15개의 workspace 프로젝트
- **Build System**: TypeScript + esbuild
- **Main Website**: Gatsby 기반 정적 사이트

## Playground 관련 핵심 패키지들

### 1. Core Playground Components

#### `packages/playground/` - 메인 플레이그라운드 UI
```
playground/
├── src/
│   ├── index.ts              # 메인 setupPlayground 함수
│   ├── createElements.ts     # UI 요소 생성 (사이드바, 탭 등)
│   ├── createUI.ts          # UI 유틸리티 및 알림
│   ├── sidebar/             # 사이드바 플러그인들
│   │   ├── plugins.ts       # 플러그인 관리
│   │   ├── settings.ts      # 설정 플러그인
│   │   └── runtime.ts       # 코드 실행 로그
│   ├── monaco/              # Monaco 에디터 확장
│   ├── exporter.ts          # 코드 내보내기 (CodeSandbox 등)
│   ├── navigation.ts        # 네비게이션 관리
│   └── pluginUtils.ts       # 플러그인 유틸리티
├── package.json
└── README.md
```

**주요 기능:**
- 플러그인 시스템 (사이드바 확장)
- 컴파일러 옵션 UI
- 코드 내보내기 기능
- 예제 시스템 통합
- URL 기반 상태 관리

#### `packages/sandbox/` - 핵심 에디터 컴포넌트
```
sandbox/
├── src/
│   ├── index.ts             # 메인 Sandbox 클래스
│   ├── monaco/              # Monaco 에디터 설정
│   ├── compiler/            # TypeScript 컴파일러 통합
│   └── types.ts             # 타입 정의
└── package.json
```

**주요 기능:**
- Monaco 에디터 + TypeScript 통합
- 자동 타입 수집 (ATA)
- 컴파일러 설정 관리
- 가상 파일 시스템

#### `packages/playground-worker/` - WebWorker
```
playground-worker/
├── index.ts                 # TypeScript Language Service 래퍼
├── types.d.ts              # 워커 타입 정의
└── package.json
```

**주요 기능:**
- TypeScript Language Service를 WebWorker에서 실행
- `// @filename:` 구문 지원
- Monaco-TypeScript 바인딩 확장

### 2. Content & Examples

#### `packages/playground-examples/` - 코드 예제
```
playground-examples/
├── copy/                    # 다국어 예제들
│   ├── en/                 # 영어 예제
│   ├── ko/                 # 한국어 예제
│   └── [other-langs]/      # 기타 언어들
├── generated/              # 자동 생성된 예제 메타데이터
└── scripts/                # 예제 빌드 스크립트
```

#### `packages/playground-handbook/` - 플레이그라운드 문서
- 사용자 가이드 및 튜토리얼
- 플러그인 개발 가이드

### 3. Plugin Development

#### `packages/create-typescript-playground-plugin/` - 플러그인 템플릿
```
create-typescript-playground-plugin/
├── template/               # 플러그인 템플릿
│   ├── src/
│   │   └── index.ts       # 플러그인 엔트리포인트
│   ├── package.json
│   └── rollup.config.js
├── index.js               # npm init 스크립트
└── package.json
```

### 4. Website Integration

#### `packages/typescriptlang-org/` - 메인 웹사이트
```
typescriptlang-org/
├── src/
│   ├── pages/
│   │   └── play.tsx           # 플레이그라운드 페이지
│   ├── templates/
│   │   └── playgroundHandbook.tsx  # 핸드북 템플릿
│   ├── lib/
│   │   └── playgroundURLs.ts  # URL 유틸리티
│   └── copy/
│       └── en/playground.ts   # 플레이그라운드 텍스트
└── static/
    ├── js/playground/         # 빌드된 플레이그라운드 JS
    └── images/               # 플레이그라운드 이미지들
```

## Architecture Deep Dive

### Playground Setup Flow

1. **초기화** (`setupPlayground` in `packages/playground/src/index.ts`)
   ```typescript
   setupPlayground(sandbox, monaco, config, i18n, react)
   ```

2. **UI 구성**
   - 왼쪽 네비게이션 (예제, 핸드북)
   - 중앙 에디터 (Sandbox/Monaco)
   - 오른쪽 사이드바 (플러그인들)

3. **플러그인 시스템**
   ```typescript
   interface PlaygroundPlugin {
     id: string
     displayName: string
     willMount?: (sandbox, container) => void
     didMount?: (sandbox, container) => void
     modelChanged?: (sandbox, model, container) => void
   }
   ```

### Key Data Flow

```
사용자 입력 → Monaco Editor → Sandbox → TypeScript Compiler → 결과 표시
                    ↓
              Plugin System → 사이드바 UI 업데이트
                    ↓
              URL State → 브라우저 히스토리
```

### Plugin Architecture

플레이그라운드는 확장 가능한 플러그인 시스템을 사용합니다:

1. **기본 플러그인들**:
   - JavaScript 출력
   - Type Definition (.d.ts)
   - 에러 및 로그
   - AST 뷰어
   - 설정

2. **커스텀 플러그인**:
   - NPM에서 설치 가능
   - 로컬 개발 서버 연결 지원

## Key Features

### 1. URL-based State Management
```
https://typescriptlang.org/play?
  ts=4.9.4&                    # TypeScript 버전
  target=99&                   # 컴파일 타겟
  module=1&                    # 모듈 시스템
  #code/PRA...                 # Base64 + gzip 압축된 코드
```

### 2. Multi-file Support
```typescript
// @filename: utils.ts
export function helper() { }

// @filename: main.ts  
import { helper } from "./utils"
```

### 3. Export Integration
- CodeSandbox
- TypeScript AST Viewer
- Carbon (코드 이미지)
- 클립보드 복사

### 4. Example System
- 카테고리별 구성 (TypeScript, JavaScript, Types 등)
- 다국어 지원
- 검색 및 필터링

## Development Workflow

### Building the Playground
```bash
# 전체 프로젝트 빌드
pnpm build

# 플레이그라운드만 빌드
pnpm run --filter=@typescript/playground build

# 개발 서버 시작
pnpm start  # 전체 웹사이트 실행 (포트 8000)
```

### Testing
```bash
# 플레이그라운드 테스트
pnpm run --filter=@typescript/playground test

# 전체 테스트
pnpm test
```

## Extension Points

### 1. Custom Plugins
새로운 플러그인을 만들어 기능 확장:
```bash
npm init playground-plugin my-plugin
```

### 2. Examples
새로운 코드 예제 추가:
```
packages/playground-examples/copy/en/[category]/[example].md
```

### 3. Compiler Options
새로운 컴파일러 옵션 UI 추가는 `createConfigDropdown.ts`에서 수정

## Integration with Monaco Editor

플레이그라운드는 Monaco Editor를 기반으로 하며 다음과 같은 확장을 제공합니다:

1. **TypeScript Language Service** 통합
2. **Code Lens** - 파일명 표시
3. **Inlay Hints** - 타입 힌트
4. **Link Provider** - 예제 간 네비게이션
5. **Custom Actions** - 키보드 단축키

## Dependencies

주요 외부 의존성:
- `monaco-editor` - 코드 에디터
- `typescript` - 컴파일러
- `gatsby` - 웹사이트 프레임워크
- `react` - UI 라이브러리
- `esbuild` - 번들러

## Performance Considerations

1. **WebWorker 사용** - Language Service를 메인 스레드에서 분리
2. **Lazy Loading** - 플러그인과 예제의 지연 로딩
3. **Code Splitting** - 번들 크기 최적화
4. **Caching** - LocalStorage를 통한 상태 저장

## 플레이그라운드 상세 기술 분석

### Core Plugin System Implementation

플레이그라운드의 플러그인 시스템은 다음과 같이 구현됩니다:

#### 1. Plugin Interface
```typescript
interface PlaygroundPlugin {
  id: string                    // 고유 식별자
  displayName: string          // 탭에 표시될 이름
  shouldBeSelected?: () => boolean  // 초기 선택 여부
  
  // 라이프사이클 메서드들
  willMount?: (sandbox: Sandbox, container: HTMLDivElement) => void
  didMount?: (sandbox: Sandbox, container: HTMLDivElement) => void
  willUnmount?: (sandbox: Sandbox, container: HTMLDivElement) => void
  didUnmount?: (sandbox: Sandbox, container: HTMLDivElement) => void
  
  // 에디터 변경 이벤트 핸들러들
  modelChanged?: (sandbox: Sandbox, model: ITextModel, container: HTMLDivElement) => void
  modelChangedDebounce?: (sandbox: Sandbox, model: ITextModel, container: HTMLDivElement) => void
  
  data?: any                   // 플러그인 상태 저장
}
```

#### 2. 기본 플러그인들 (packages/playground/src/sidebar/)

**JavaScript 출력 (showJS.ts)**
```typescript
// TypeScript 코드를 JavaScript로 컴파일 결과 표시
// 실시간 컴파일 결과, 에러 표시, 코드 실행 기능
```

**타입 정의 (showDTS.ts)**
```typescript  
// .d.ts 파일 생성 및 표시
// TypeScript 타입 정의 파일 미리보기
```

**에러 표시 (showErrors.ts)**
```typescript
// 컴파일 에러, 경고, 제안사항 표시
// Monaco 에디터의 마커와 연동
```

**AST 뷰어 (ast.ts)**
```typescript
// TypeScript AST (Abstract Syntax Tree) 시각화
// 코드 구조 분석 및 디버깅 도구
```

#### 3. Plugin Management System

**플러그인 등록 및 활성화** (packages/playground/src/sidebar/plugins.ts):
```typescript
// NPM 플러그인 자동 발견
export const allNPMPlugins = [
  { id: "typescript-playground-presentation-mode", ... },
  { id: "playground-transformer-timeline", ... }
]

// 로컬스토리지 기반 플러그인 상태 관리
export const activePlugins = () => {
  const existing = customPlugins().map(module => ({ id: module }))
  return existing.concat(allNPMPlugins.filter(p => !!localStorage.getItem("plugin-" + p.id)))
}

// 개발용 로컬호스트 플러그인 연결
export const allowConnectingToLocalhost = () => {
  return !!localStorage.getItem("compiler-setting-connect-dev-plugin")
}
```

### UI Architecture Details

#### 1. Layout System (createElements.ts)
```
+------------------+--+------------------------+--+------------------+
|   Navigation     || |      Monaco Editor     || |    Sidebar       |
| (Examples, etc.) || |     (TypeScript)       || |   (Plugins)      |
|                  || |                        || |                  |
+------------------+--+------------------------+--+------------------+
   ↑ dragbar-left      ↑ editor-container         ↑ dragbar-right
```

**드래그 가능한 사이즈 조절**:
- 왼쪽/오른쪽 패널 크기 조절 가능
- LocalStorage에 크기 설정 저장
- 반응형 최소/최대 너비 제한

#### 2. Monaco Editor Integration

**TypeScript Language Service 설정**:
```typescript
// packages/sandbox/src/index.ts
export const createTypeScriptSandbox = (config: SandboxConfig, monaco: Monaco) => {
  // Monaco-TypeScript 언어 서비스 설정
  // 컴파일러 옵션 적용
  // 가상 파일 시스템 설정
  // ATA (Automatic Type Acquisition) 활성화
}
```

**Key Features**:
- **Code Lens**: 파일명 표시 (`// @filename: example.ts`)
- **Inlay Hints**: TypeScript 4.6+ 타입 힌트
- **Link Provider**: 예제 간 네비게이션
- **Custom Actions**: Ctrl+S (공유), Ctrl+Enter (실행)

### Advanced Features

#### 1. Multi-file Support
```typescript
// @filename: utils.ts
export function helper() { return "hello" }

// @filename: main.ts  
import { helper } from "./utils"
console.log(helper())
```
- Virtual File System (packages/typescript-vfs) 활용
- 파일 간 import/export 지원
- Language Service가 모든 파일 인식

#### 2. URL State Management
```typescript
// packages/sandbox/src/compilerOptions.ts
export const createURLQueryWithCompilerOptions = (sandbox: Sandbox) => {
  // 컴파일러 옵션을 URL 쿼리로 변환
  // 코드를 base64 + LZ-string 압축
  // 브라우저 히스토리 관리
}
```

**URL 구조**:
```
?ts=4.9.4&target=99&module=1&jsx=2#code/PRA...
├── ts=4.9.4           # TypeScript 버전
├── target=99          # 컴파일 타겟 (ES2022)
├── module=1           # 모듈 시스템 (CommonJS)
├── jsx=2              # JSX 처리 방식
└── #code/PRA...       # 압축된 코드
```

#### 3. Export System (exporter.ts)
```typescript
export const createExporter = (sandbox: Sandbox, monaco: Monaco, ui: UI) => {
  return {
    // CodeSandbox 프로젝트 생성
    openInCodeSandbox: () => { ... },
    
    // TypeScript AST Viewer 연동  
    openInTSAST: () => { ... },
    
    // Bug 리포트 생성
    openBugWorkbenchExample: () => { ... },
    
    // Carbon 코드 이미지 생성
    openInCarbon: () => { ... }
  }
}
```

### Performance Optimizations

#### 1. WebWorker Architecture
```typescript
// packages/playground-worker/index.ts
// TypeScript Language Service를 WebWorker에서 실행
// 메인 스레드 블로킹 방지
// Monaco-TypeScript 바인딩 확장
```

#### 2. Debounced Updates
```typescript
// 300ms 디바운스로 성능 최적화
sandbox.editor.onDidChangeModelContent(() => {
  const plugin = getCurrentPlugin()
  if (plugin.modelChanged) plugin.modelChanged(...)
  
  if (debouncingTimer) return
  debouncingTimer = true
  setTimeout(() => {
    plugin.modelChangedDebounce?.(...)
  }, 300)
})
```

#### 3. Lazy Loading
- 플러그인은 활성화시에만 로드
- 예제는 필요시에만 fetch
- TypeScript 버전은 동적 로딩

### Extension and Customization Points

#### 1. 새로운 플러그인 만들기
```bash
npm init playground-plugin my-custom-plugin
```

**플러그인 템플릿 구조**:
```typescript
import { PlaygroundPlugin, PluginUtils } from "@typescript/playground"

const plugin: PlaygroundPlugin = {
  id: "my-plugin",
  displayName: "My Custom Plugin",
  didMount: (sandbox, container) => {
    // 플러그인 UI 구성
    container.innerHTML = `<h3>My Plugin</h3>`
  },
  modelChanged: (sandbox, model) => {
    // 코드 변경시 로직
    const code = model.getValue()
    // 분석 결과 표시
  }
}

export default plugin
```

#### 2. 컴파일러 옵션 UI 확장
```typescript
// packages/playground/src/createConfigDropdown.ts
// 새로운 컴파일러 플래그 UI 추가
// 카테고리별 옵션 그룹핑
// 실시간 변경 적용
```

#### 3. 예제 시스템 확장
```markdown
<!-- packages/playground-examples/copy/en/[category]/new-example.ts -->
//// { "order": 1, "isJavaScript": false }

// 새로운 예제 코드
// 마크다운 + 프론트매터 형식
```

### Build and Deployment Process

#### 1. Development Workflow
```bash
# 전체 개발 서버 (포트 8000)
pnpm start

# 개별 패키지 빌드
pnpm run --filter=@typescript/playground build

# 플레이그라운드 빠른 테스트 빌드  
pnpm run --filter=@typescript/playground build-fast-test
```

#### 2. Production Build
```bash
# esbuild를 통한 번들링
# packages/typescriptlang-org/static/js/playground/ 출력
# Cache busting을 위한 파일명 해싱
```

#### 3. CI/CD Integration
- GitHub Actions를 통한 자동 배포
- TypeScript 버전 업데이트 자동화
- 다국어 번역 동기화

### Testing Infrastructure

#### 1. Test Coverage
- **Playground**: 기본 렌더링 테스트
- **Sandbox**: 포괄적인 테스트 스위트
  - Type Acquisition 테스트
  - Twoslash 지원 테스트  
  - 기본 컴파일러 옵션 테스트
  - 스냅샷 테스트 (7개)

#### 2. Test Commands
```bash
# 전체 테스트 실행
pnpm test

# 플레이그라운드 테스트만
pnpm run --filter=@typescript/playground test

# 샌드박스 테스트만  
pnpm run --filter=@typescript/sandbox test
```

### Examples System Deep Dive

#### 1. 예제 구조 (packages/playground-examples/)
```
copy/en/                     # 영어 예제들
├── JavaScript/              # JavaScript 예제
│   ├── JavaScript Essentials/
│   └── Working With the DOM/
├── TypeScript/              # TypeScript 기본 예제
│   ├── Primitives/         # 기본 타입들
│   ├── Classes/            # 클래스 시스템
│   └── Functions/          # 함수
├── 3-7/, 3-8/, 4-0/, ...   # 버전별 새 기능들
└── Playground/             # 플레이그라운드 특화 예제
```

#### 2. 예제 파일 형식
```typescript
//// { "order": 1, "isJavaScript": false, "compiler": { "target": 1 } }

// Any is the TypeScript escape clause...
const myObject = JSON.parse("{}");
```

**Frontmatter 옵션**:
- `order`: 카테고리 내 정렬 순서
- `isJavaScript`: JavaScript 예제 여부
- `compiler`: 특정 컴파일러 옵션

#### 3. 예제 빌드 프로세스
```bash
# 예제 메타데이터 생성
pnpm run --filter=playground-examples build

# 다국어 예제 생성 
# packages/playground-examples/generated/ 에 출력
```

### 국제화 (i18n) Support

#### 1. 다국어 지원 구조
```
packages/playground-examples/copy/
├── en/          # 영어 (기본)
├── ja/          # 일본어  
├── ko/          # 한국어
├── zh/          # 중국어
└── ...
```

#### 2. 번역 워크플로우
- 기본 영어 예제 작성
- microsoft/TypeScript-Website-Localizations 저장소에서 번역 관리
- CI/CD를 통한 자동 동기화

### Future Extension Possibilities

#### 1. 새로운 플러그인 아이디어
- **Performance Profiler**: 컴파일 성능 분석
- **Dependency Visualizer**: import/export 관계 시각화
- **TypeScript Migration Helper**: JavaScript → TypeScript 변환 도구
- **Code Quality Metrics**: 코드 복잡도 분석

#### 2. 플레이그라운드 개선 방향
- **Mobile Responsiveness**: 모바일 환경 최적화
- **Collaborative Editing**: 실시간 협업 기능
- **Version Control**: Git 통합
- **Package Manager**: NPM 패키지 직접 설치

#### 3. 통합 가능한 도구들
- **ESLint**: 코드 품질 검사
- **Prettier**: 코드 포맷팅
- **Webpack/Vite**: 번들링 시뮬레이션
- **Jest**: 테스트 실행 환경

### 개발자를 위한 실용적 가이드

#### 1. 로컬 개발 환경 설정
```bash
# 1. 저장소 클론
git clone https://github.com/microsoft/TypeScript-website
cd TypeScript-website

# 2. 의존성 설치
pnpm install

# 3. 개발 서버 시작
pnpm start  # http://localhost:8000

# 4. 플레이그라운드만 빌드
pnpm run --filter=@typescript/playground build
```

#### 4. 플러그인 개발 워크플로우
```bash
# 1. 플러그인 템플릿 생성
npm init playground-plugin my-plugin

# 2. 로컬 개발 서버 시작
cd my-plugin
npm run serve  # http://localhost:5000

# 3. 플레이그라운드에서 dev plugin 활성화
# localStorage에 'compiler-setting-connect-dev-plugin' 설정

# 4. 플레이그라운드 새로고침하여 플러그인 로드 확인
```

#### 5. 디버깅 팁
```javascript
// 브라우저 콘솔에서 사용 가능한 글로벌 변수들
window.ts        // TypeScript API
window.sandbox   // Sandbox 인스턴스  
window.playground // Playground 인스턴스
window.react     // React 라이브러리
window.reactDOM  // ReactDOM 라이브러리

// 예시: 현재 코드 가져오기
console.log(window.sandbox.getText())

// 예시: 컴파일 결과 확인
console.log(window.sandbox.getRunnableJS())
```

## 결론

TypeScript Website의 Playground는 잘 설계된 모듈형 아키텍처를 가지고 있으며, 다음과 같은 특징을 보입니다:

### 강점
1. **확장성**: 플러그인 시스템을 통한 무한 확장 가능
2. **성능**: WebWorker 활용으로 메인 스레드 블로킹 없음
3. **사용성**: 직관적인 UI와 풍부한 예제 시스템
4. **개발자 친화적**: 상세한 API와 명확한 분리된 관심사

### 활용 가능성
- TypeScript 학습 도구로 활용
- 새로운 언어 기능 실험 환경
- 코드 공유 및 협업 플랫폼
- 교육용 Interactive 도구

이 코드 구조 분석을 통해 TypeScript Playground의 전체적인 아키텍처와 개별 컴포넌트들의 역할을 이해할 수 있으며, 향후 확장이나 개선 작업의 기초 자료로 활용할 수 있습니다.