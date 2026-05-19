# 설정(Configuration)

> Nuxt는 생산성을 높일 수 있도록 합리적인 기본 설정과 함께 제공됩니다.

기본적으로 Nuxt는 대부분의 사용 사례를 지원하도록 설정되어 있습니다. [`nuxt.config.ts`](/docs/4.x/directory-structure/nuxt-config) 파일을 통해 이러한 기본 설정을 재정의하거나 확장할 수 있습니다.

## Nuxt 설정

[`nuxt.config.ts`](/docs/4.x/directory-structure/nuxt-config) 파일은 Nuxt 프로젝트의 루트에 위치하며, 애플리케이션의 동작을 재정의하거나 확장할 수 있습니다.

최소 구성 파일은 설정 객체를 포함한 `defineNuxtConfig` 함수를 export 합니다. `defineNuxtConfig` 헬퍼는 import 없이 전역으로 사용할 수 있습니다.

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  // My Nuxt config
})
```

이 파일은 커스텀 스크립트 추가, 모듈 등록, 렌더링 모드 변경 등과 같은 문서 예제에서 자주 언급됩니다.

<read-more to="/docs/4.x/api/configuration/nuxt-config">

모든 옵션은 **Configuration Reference**에서 설명되어 있습니다.

</read-more>

<note>

Nuxt 애플리케이션을 빌드할 때 반드시 TypeScript를 사용할 필요는 없습니다. 하지만 `nuxt.config` 파일에는 `.ts` 확장자를 사용하는 것을 강력히 권장합니다. 이렇게 하면 IDE에서 힌트를 제공받아 설정을 수정할 때 오타나 실수를 줄일 수 있습니다.

</note>

### 환경별 Override

`nuxt.config`에서 완전히 타입이 지정된 환경별 override 설정을 사용할 수 있습니다.

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  $production: {
    routeRules: {
      '/**': { isr: true },
    },
  },
  $development: {
    //
  },
  $env: {
    staging: {
      //
    },
  },
})
```

Nuxt CLI 명령 실행 시 특정 환경을 선택하려면 다음과 같이 `--envName` 플래그에 환경 이름을 전달하면 됩니다: `nuxt build --envName staging`.

이 override 메커니즘에 대해 더 알아보려면 `c12` 문서의 [environment-specific configuration](https://github.com/unjs/c12?tab=readme-ov-file#environment-specific-configuration)을 참고하세요.

<video-accordion title="Alexander Lichter의 env-aware nuxt.config.ts 영상 보기" video-id="DFZI2iVCrNc">

</video-accordion>

<note>

레이어를 작성하는 경우 `$meta` 키를 사용하여 레이어 사용자 또는 작성자가 활용할 수 있는 메타데이터를 제공할 수도 있습니다.

</note>

### 환경 변수 및 비공개 토큰

`runtimeConfig` API는 환경 변수와 같은 값을 애플리케이션 전반에서 사용할 수 있도록 제공합니다. 기본적으로 이 키들은 서버 측에서만 사용할 수 있습니다. `runtimeConfig.public` 및 `runtimeConfig.app`(Nuxt 내부에서 사용)의 키는 클라이언트 측에서도 사용할 수 있습니다.

이 값들은 `nuxt.config`에 정의되어야 하며, 환경 변수를 통해 override 할 수 있습니다.

<code-group>

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  runtimeConfig: {
    // 서버 측에서만 사용 가능한 비공개 키
    apiSecret: '123',
    // public 내부 키는 클라이언트 측에도 노출됩니다
    public: {
      apiBase: '/api',
    },
  },
})
```

```ini [.env]
# apiSecret 값을 override 합니다
NUXT_API_SECRET=api_secret_token
```

</code-group>

이 변수들은 [`useRuntimeConfig()`](/docs/4.x/api/composables/use-runtime-config) composable을 사용하여 애플리케이션 전반에서 접근할 수 있습니다.

```vue [app/pages/index.vue]
<script setup lang="ts">
const runtimeConfig = useRuntimeConfig()
</script>
```

<read-more to="/docs/4.x/guide/going-further/runtime-config">

</read-more>

## 앱 설정(App Configuration)

소스 디렉토리(기본값 `app/`)에 위치한 `app.config.ts` 파일은 빌드 시점에 결정되는 공개 변수를 노출하는 데 사용됩니다. `runtimeConfig` 옵션과 달리, 환경 변수를 사용하여 override 할 수 없습니다.

최소 구성 파일은 설정 객체를 포함한 `defineAppConfig` 함수를 export 합니다. `defineAppConfig` 헬퍼는 import 없이 전역으로 사용할 수 있습니다.

```ts [app/app.config.ts]
export default defineAppConfig({
  title: 'Hello Nuxt',
  theme: {
    dark: true,
    colors: {
      primary: '#ff0000',
    },
  },
})
```

이 변수들은 [`useAppConfig`](/docs/4.x/api/composables/use-app-config) composable을 사용하여 애플리케이션 전반에서 접근할 수 있습니다.

```vue [app/pages/index.vue]
<script setup lang="ts">
const appConfig = useAppConfig()
</script>
```

<read-more to="/docs/4.x/directory-structure/app/app-config">

</read-more>

## `runtimeConfig` vs. `app.config`

위에서 설명했듯이 `runtimeConfig`와 `app.config`는 모두 애플리케이션 전반에 변수를 노출하는 데 사용됩니다. 어떤 것을 사용해야 할지 판단하기 위한 가이드는 다음과 같습니다.

* `runtimeConfig`: 빌드 이후 환경 변수를 통해 지정되어야 하는 비공개 또는 공개 토큰
* `app.config`: 빌드 시점에 결정되는 공개 토큰, 테마 종류, 제목 등과 같은 웹사이트 설정 및 민감하지 않은 프로젝트 설정

<table>
<thead>
  <tr>
    <th>
      기능
    </th>

```
<th>
  <code>
    runtimeConfig
  </code>
</th>

<th>
  <code>
    app.config
  </code>
</th>
```

  </tr>
</thead>

<tbody>
  <tr>
    <td>
      클라이언트 측
    </td>

```
<td>
  Hydrated
</td>

<td>
  Bundled
</td>
```

  </tr>

  <tr>
    <td>
      환경 변수
    </td>

```
<td>
  ✅ 예
</td>

<td>
  ❌ 아니오
</td>
```

  </tr>

  <tr>
    <td>
      반응성(Reactive)
    </td>

```
<td>
  ✅ 예
</td>

<td>
  ✅ 예
</td>
```

  </tr>

  <tr>
    <td>
      타입 지원
    </td>

```
<td>
  ✅ 부분 지원
</td>

<td>
  ✅ 예
</td>
```

  </tr>

  <tr>
    <td>
      요청별 설정
    </td>

```
<td>
  ❌ 아니오
</td>

<td>
  ✅ 예
</td>
```

  </tr>

  <tr>
    <td>
      Hot module replacement
    </td>

```
<td>
  ❌ 아니오
</td>

<td>
  ✅ 예
</td>
```

  </tr>

  <tr>
    <td>
      Non-primitive JS 타입
    </td>

```
<td>
  ❌ 아니오
</td>

<td>
  ✅ 예
</td>
```

  </tr>
</tbody>
</table>

## 외부 설정 파일

Nuxt는 [`nuxt.config.ts`](/docs/4.x/directory-structure/nuxt-config) 파일을 단일 설정 기준(source of truth)으로 사용하며 외부 설정 파일은 읽지 않습니다. 프로젝트를 빌드하는 과정에서 이러한 설정이 필요한 경우가 있을 수 있습니다. 아래 표는 일반적인 설정 항목과, Nuxt에서 이를 설정하는 방법을 보여줍니다.

<table>
<thead>
  <tr>
    <th>
      이름
    </th>

```
<th>
  설정 파일
</th>

<th>
  설정 방법
</th>
```

  </tr>
</thead>

<tbody>
  <tr>
    <td>
      <a href="https://nitro.build" rel="nofollow">
        Nitro
      </a>
    </td>

```
<td>
  <del>
    <code>
      nitro.config.ts
    </code>
  </del>
</td>

<td>
  <code>nuxt.config</code>의 <a href="/docs/4.x/api/nuxt-config#nitro">
    <code>
      nitro
    </code>
  </a>
  키 사용
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://postcss.org" rel="nofollow">
        PostCSS
      </a>
    </td>

```
<td>
  <del>
    <code>
      postcss.config.js
    </code>
  </del>
</td>

<td>
  <code>nuxt.config</code>의 <a href="/docs/4.x/api/nuxt-config#postcss">
    <code>
      postcss
    </code>
  </a>
  키 사용
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://vite.dev" rel="nofollow">
        Vite
      </a>
    </td>

```
<td>
  <del>
    <code>
      vite.config.ts
    </code>
  </del>
</td>

<td>
  <code>nuxt.config</code>의 <a href="/docs/4.x/api/nuxt-config#vite">
    <code>
      vite
    </code>
  </a>
  키 사용
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://webpack.js.org" rel="nofollow">
        webpack
      </a>
    </td>

```
<td>
  <del>
    <code>
      webpack.config.ts
    </code>
  </del>
</td>

<td>
  <code>nuxt.config</code>의 <a href="/docs/4.x/api/nuxt-config#webpack-1">
    <code>
      webpack
    </code>
  </a>
  키 사용
</td>
```

  </tr>
</tbody>
</table>

다음은 기타 일반적인 설정 파일 목록입니다.

<table>
<thead>
  <tr>
    <th>
      이름
    </th>

```
<th>
  설정 파일
</th>

<th>
  설정 방법
</th>
```

  </tr>
</thead>

<tbody>
  <tr>
    <td>
      <a href="https://www.typescriptlang.org" rel="nofollow">
        TypeScript
      </a>
    </td>

```
<td>
  <code>
    tsconfig.json
  </code>
</td>

<td>
  <a href="/docs/4.x/directory-structure/tsconfig">
    자세히 보기
  </a>
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://eslint.org" rel="nofollow">
        ESLint
      </a>
    </td>

```
<td>
  <code>
    eslint.config.js
  </code>
</td>

<td>
  <a href="https://eslint.org/docs/latest/use/configure/configuration-files" rel="nofollow">
    자세히 보기
  </a>
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://prettier.io" rel="nofollow">
        Prettier
      </a>
    </td>

```
<td>
  <code>
    prettier.config.js
  </code>
</td>

<td>
  <a href="https://prettier.io/docs/configuration.html" rel="nofollow">
    자세히 보기
  </a>
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://stylelint.io" rel="nofollow">
        Stylelint
      </a>
    </td>

```
<td>
  <code>
    stylelint.config.js
  </code>
</td>

<td>
  <a href="https://stylelint.io/user-guide/configure/" rel="nofollow">
    자세히 보기
  </a>
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://tailwindcss.com" rel="nofollow">
        TailwindCSS
      </a>
    </td>

```
<td>
  <code>
    tailwind.config.js
  </code>
</td>

<td>
  <a href="https://tailwindcss.nuxtjs.org/tailwindcss/configuration/" rel="nofollow">
    자세히 보기
  </a>
</td>
```

  </tr>

  <tr>
    <td>
      <a href="https://vitest.dev" rel="nofollow">
        Vitest
      </a>
    </td>

```
<td>
  <code>
    vitest.config.ts
  </code>
</td>

<td>
  <a href="https://vitest.dev/config/" rel="nofollow">
    자세히 보기
  </a>
</td>
```

  </tr>
</tbody>
</table>

## Vue 설정

### Vite 사용 시

`@vitejs/plugin-vue` 또는 `@vitejs/plugin-vue-jsx`에 옵션을 전달해야 하는 경우 `nuxt.config` 파일에서 설정할 수 있습니다.

* `vite.vue`: `@vitejs/plugin-vue`용. [사용 가능한 옵션](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue) 참고
* `vite.vueJsx`: `@vitejs/plugin-vue-jsx`용. [사용 가능한 옵션](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue-jsx) 참고

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  vite: {
    vue: {
      customElement: true,
    },
    vueJsx: {
      mergeProps: true,
    },
  },
})
```

<read-more to="/docs/4.x/api/configuration/nuxt-config#vue">

</read-more>

### webpack 사용 시

webpack을 사용하고 `vue-loader`를 설정해야 하는 경우, `nuxt.config` 파일 내부의 `webpack.loaders.vue` 키를 사용하여 설정할 수 있습니다. 사용 가능한 옵션은 [여기](https://github.com/vuejs/vue-loader/blob/main/src/index.ts#L32-L62)에 정의되어 있습니다.

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  webpack: {
    loaders: {
      vue: {
        hotReload: true,
      },
    },
  },
})
```

<read-more to="/docs/4.x/api/configuration/nuxt-config#loaders">

</read-more>

### 실험적 Vue 기능 활성화

`propsDestructure`와 같은 Vue의 실험적 기능을 활성화해야 할 수 있습니다. Nuxt는 어떤 빌더를 사용하든 `nuxt.config.ts`에서 쉽게 설정할 수 있는 방법을 제공합니다.

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  vue: {
    propsDestructure: true,
  },
})
```

#### Vue 3.4 및 Nuxt 3.9에서 experimental `reactivityTransform` 마이그레이션

Nuxt 3.9 및 Vue 3.4부터 `reactivityTransform`은 Vue에서 Vue Macros로 이동되었으며, [Nuxt integration](https://vue-macros.dev/guide/nuxt-integration.html)을 제공합니다.

<read-more to="/docs/4.x/api/configuration/nuxt-config#vue-1">

</read-more>
