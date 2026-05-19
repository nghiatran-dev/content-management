# Cấu hình (Configuration)

> Nuxt được cấu hình với các giá trị mặc định hợp lý để giúp bạn làm việc hiệu quả hơn.

Mặc định, Nuxt đã được cấu hình để đáp ứng hầu hết các trường hợp sử dụng. File [`nuxt.config.ts`](/docs/4.x/directory-structure/nuxt-config) có thể ghi đè hoặc mở rộng cấu hình mặc định này.

## Cấu hình Nuxt

File [`nuxt.config.ts`](/docs/4.x/directory-structure/nuxt-config) nằm ở thư mục gốc của dự án Nuxt và có thể ghi đè hoặc mở rộng hành vi của ứng dụng.

Một file cấu hình tối thiểu sẽ export hàm `defineNuxtConfig` chứa object cấu hình của bạn. Helper `defineNuxtConfig` có sẵn toàn cục mà không cần import.

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  // My Nuxt config
})
```

File này thường được nhắc đến trong tài liệu hướng dẫn, ví dụ như khi thêm script tùy chỉnh, đăng ký module hoặc thay đổi chế độ render.

<read-more to="/docs/4.x/api/configuration/nuxt-config">

Mọi tùy chọn đều được mô tả trong **Configuration Reference**.

</read-more>

<note>

Bạn không bắt buộc phải dùng TypeScript để xây dựng ứng dụng với Nuxt. Tuy nhiên, bạn nên sử dụng đuôi `.ts` cho file `nuxt.config`. Điều này giúp IDE hiển thị gợi ý để tránh lỗi chính tả và sai sót khi chỉnh sửa cấu hình.

</note>

### Ghi đè theo môi trường (Environment Overrides)

Bạn có thể cấu hình override theo từng môi trường với đầy đủ typing trong `nuxt.config`.

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

Để chọn môi trường khi chạy lệnh Nuxt CLI, chỉ cần truyền tên môi trường vào flag `--envName`, ví dụ: `nuxt build --envName staging`.

Để tìm hiểu thêm về cơ chế override này, hãy tham khảo tài liệu `c12` về [environment-specific configuration](https://github.com/unjs/c12?tab=readme-ov-file#environment-specific-configuration).

<video-accordion title="Xem video của Alexander Lichter về env-aware nuxt.config.ts" video-id="DFZI2iVCrNc">

</video-accordion>

<note>

Nếu bạn đang xây dựng layer, bạn cũng có thể sử dụng key `$meta` để cung cấp metadata mà bạn hoặc người dùng layer có thể sử dụng.

</note>

### Biến môi trường và token riêng tư

API `runtimeConfig` cho phép expose các giá trị như biến môi trường đến toàn bộ ứng dụng. Mặc định, các key này chỉ khả dụng ở phía server. Các key bên trong `runtimeConfig.public` và `runtimeConfig.app` (được Nuxt sử dụng nội bộ) cũng khả dụng ở phía client.

Các giá trị này nên được định nghĩa trong `nuxt.config` và có thể bị ghi đè bằng biến môi trường.

<code-group>

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  runtimeConfig: {
    // Các key private chỉ khả dụng phía server
    apiSecret: '123',
    // Các key bên trong public cũng được expose phía client
    public: {
      apiBase: '/api',
    },
  },
})
```

```ini [.env]
# Giá trị này sẽ ghi đè apiSecret
NUXT_API_SECRET=api_secret_token
```

</code-group>

Các biến này được expose đến toàn bộ ứng dụng thông qua composable [`useRuntimeConfig()`](/docs/4.x/api/composables/use-runtime-config).

```vue [app/pages/index.vue]
<script setup lang="ts">
const runtimeConfig = useRuntimeConfig()
</script>
```

<read-more to="/docs/4.x/guide/going-further/runtime-config">

</read-more>

## Cấu hình ứng dụng (App Configuration)

File `app.config.ts`, nằm trong thư mục source (mặc định là `app/`), được dùng để expose các biến public có thể xác định tại thời điểm build. Khác với `runtimeConfig`, các biến này không thể bị ghi đè bằng biến môi trường.

Một file cấu hình tối thiểu sẽ export hàm `defineAppConfig` chứa object cấu hình của bạn. Helper `defineAppConfig` có sẵn toàn cục mà không cần import.

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

Các biến này được expose đến toàn bộ ứng dụng thông qua composable [`useAppConfig`](/docs/4.x/api/composables/use-app-config).

```vue [app/pages/index.vue]
<script setup lang="ts">
const appConfig = useAppConfig()
</script>
```

<read-more to="/docs/4.x/directory-structure/app/app-config">

</read-more>

## `runtimeConfig` vs. `app.config`

Như đã đề cập ở trên, cả `runtimeConfig` và `app.config` đều được dùng để expose biến đến toàn bộ ứng dụng. Để xác định nên dùng cái nào, dưới đây là một số hướng dẫn:

* `runtimeConfig`: Token private hoặc public cần được chỉ định sau khi build bằng biến môi trường.
* `app.config`: Token public được xác định tại thời điểm build, cấu hình website như theme, tiêu đề và các config dự án không nhạy cảm.

<table>
<thead>
  <tr>
    <th>
      Tính năng
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
      Phía client
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
      Biến môi trường
    </td>

```
<td>
  ✅ Có
</td>

<td>
  ❌ Không
</td>
```

  </tr>

  <tr>
    <td>
      Reactive
    </td>

```
<td>
  ✅ Có
</td>

<td>
  ✅ Có
</td>
```

  </tr>

  <tr>
    <td>
      Hỗ trợ kiểu dữ liệu
    </td>

```
<td>
  ✅ Một phần
</td>

<td>
  ✅ Có
</td>
```

  </tr>

  <tr>
    <td>
      Cấu hình theo từng request
    </td>

```
<td>
  ❌ Không
</td>

<td>
  ✅ Có
</td>
```

  </tr>

  <tr>
    <td>
      Hot module replacement
    </td>

```
<td>
  ❌ Không
</td>

<td>
  ✅ Có
</td>
```

  </tr>

  <tr>
    <td>
      Kiểu JS không nguyên thủy (Non-primitive JS types)
    </td>

```
<td>
  ❌ Không
</td>

<td>
  ✅ Có
</td>
```

  </tr>
</tbody>
</table>

## File cấu hình bên ngoài

Nuxt sử dụng file [`nuxt.config.ts`](/docs/4.x/directory-structure/nuxt-config) làm nguồn cấu hình duy nhất (single source of truth) và bỏ qua việc đọc các file cấu hình bên ngoài. Trong quá trình build dự án, bạn có thể cần cấu hình những công cụ này. Bảng dưới đây hiển thị các cấu hình phổ biến và cách cấu hình chúng với Nuxt.

<table>
<thead>
  <tr>
    <th>
      Tên
    </th>

```
<th>
  File cấu hình
</th>

<th>
  Cách cấu hình
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
  Sử dụng key <a href="/docs/4.x/api/nuxt-config#nitro">
    <code>
      nitro
    </code>
  </a>
  trong <code>nuxt.config</code>
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
  Sử dụng key <a href="/docs/4.x/api/nuxt-config#postcss">
    <code>
      postcss
    </code>
  </a>
  trong <code>nuxt.config</code>
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
  Sử dụng key <a href="/docs/4.x/api/nuxt-config#vite">
    <code>
      vite
    </code>
  </a>
  trong <code>nuxt.config</code>
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
  Sử dụng key <a href="/docs/4.x/api/nuxt-config#webpack-1">
    <code>
      webpack
    </code>
  </a>
  trong <code>nuxt.config</code>
</td>
```

  </tr>
</tbody>
</table>

Dưới đây là danh sách một số file cấu hình phổ biến khác:

<table>
<thead>
  <tr>
    <th>
      Tên
    </th>

```
<th>
  File cấu hình
</th>

<th>
  Cách cấu hình
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
    Thông tin thêm
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
    Thông tin thêm
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
    Thông tin thêm
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
    Thông tin thêm
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
    Thông tin thêm
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
    Thông tin thêm
  </a>
</td>
```

  </tr>
</tbody>
</table>

## Cấu hình Vue

### Với Vite

Nếu bạn cần truyền option cho `@vitejs/plugin-vue` hoặc `@vitejs/plugin-vue-jsx`, bạn có thể cấu hình trong file `nuxt.config`.

* `vite.vue` cho `@vitejs/plugin-vue`. Xem [các option khả dụng](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue).
* `vite.vueJsx` cho `@vitejs/plugin-vue-jsx`. Xem [các option khả dụng](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue-jsx).

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

### Với webpack

Nếu bạn sử dụng webpack và cần cấu hình `vue-loader`, bạn có thể thực hiện bằng key `webpack.loaders.vue` bên trong file `nuxt.config`. Các option khả dụng được [định nghĩa tại đây](https://github.com/vuejs/vue-loader/blob/main/src/index.ts#L32-L62).

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

### Bật các tính năng Vue thử nghiệm

Bạn có thể cần bật các tính năng thử nghiệm của Vue như `propsDestructure`. Nuxt cung cấp cách dễ dàng để thực hiện điều đó trong `nuxt.config.ts`, bất kể bạn đang sử dụng builder nào.

```ts [nuxt.config.ts]twoslash
export default defineNuxtConfig({
  vue: {
    propsDestructure: true,
  },
})
```

#### Migration experimental `reactivityTransform` từ Vue 3.4 và Nuxt 3.9

Từ Nuxt 3.9 và Vue 3.4, `reactivityTransform` đã được chuyển từ Vue sang Vue Macros, cung cấp [Nuxt integration](https://vue-macros.dev/guide/nuxt-integration.html).

<read-more to="/docs/4.x/api/configuration/nuxt-config#vue-1">

</read-more>
