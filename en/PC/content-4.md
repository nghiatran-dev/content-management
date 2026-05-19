# Feature Overview

CONTENT 4.md
Content Management là chức năng để Creator quản lý tài liệu theo cây nội dung:
- Tạo document, category (folder), package version.
- Viết/chỉnh sửa nội dung bằng **Text editor** hoặc **Markdown editor**.
- Thiết lập đa ngôn ngữ cho menu/title/content.
- Import tài liệu từ GitHub/GitLab.

## Điều kiện truy cập

- User đã đăng nhập.
- User có quyền **Creator**.
- **Route gốc (entry + redirect):** `/admin/content-management` (xem mục `1.1`).
- **Route màn hình chính (viewer + cây document):** `/admin/content-management/{menuId}` và `/admin/content-management/{menuId}/{docId}` (cùng một page Nuxt `[[docId]].vue`; `docId` optional).
- **Route tạo mới:** `/admin/content-management/{menuId}/create` (query `editor`, xem mục `1.1`).
- **Route chỉnh sửa:** `/admin/content-management/{menuId}/{docId}/edit`.

## 1) Init Screen (Content Management / Enter)

![](./images/Init%20screen.png)

- Sau khi đã chọn menu GNB và có `menuId` trên URL, hệ thống gọi API để load LNB (danh sách document).
- Nếu chưa có document trong menu đó, hiển thị Page Empty ở khu vực nội dung trung tâm và gợi ý bấm `+ 등록`.
- LNB có các nhóm chính:
  - `콘텐츠 등록` (dropdown tạo mới).
  - `Git 연동 관리` (trạng thái ON/OFF).
  - Danh sách document/category/package version.
- `콘텐츠 등록` mở các lựa chọn:
  - `문서 등록하기`
  - `카테고리 등록하기`
  - `문서 패키지 버전 등록하기`
- Khi chọn `문서 등록하기`, mở modal `문서 작성`.

### Modal `문서 작성`

- Có 2 hướng tạo tài liệu:
  - `Git 에서 불러오기`
  - `템플릿 선택`
- Template hiện tại có khóa/mở theo điều kiện. Trạng thái đang khả dụng để bắt đầu là `빈 문서`.
- Với template khả dụng, hover sẽ hiển thị 2 lựa chọn:
  - `마크다운` → điều hướng tới `/admin/content-management/{menuId}/create?editor=markdown` (xem mục `1.1`).
  - `텍스트에디터` → điều hướng tới `/admin/content-management/{menuId}/create?editor=text-editor` (xem mục `1.1`).

### 1.0) Page Empty (không có document trong menu đã chọn)

![](./images/Content-management-empty-state.png)

- Điều kiện hiển thị:
  - User đang ở route menu `/admin/content-management/{menuId}`.
  - Sau khi gọi API load LNB (danh sách document), response **rỗng** (chưa có tài liệu nào trong menu đó).
- URL vẫn là `/admin/content-management/{menuId}` (chưa có segment `docId`).
- Khu vực main hiển thị Page Empty gồm:
  - Tiêu đề hướng dẫn đăng ký/khởi tạo tài liệu.
  - 4 bước flow onboarding (`문서 등록` → `운영팀 검수` → `승인 후 즉시 게시` → `검수없이 수정 반영`).
  - Nút CTA `+ 등록` để bắt đầu tạo tài liệu mới.
- Sau khi user tạo thành công ít nhất một document, điều hướng theo rule ở mục `1.1` (refetch LNB; nếu có document thì URL detail `/admin/content-management/{menuId}/{docId}` theo document đầu list; `docId` có thể là id hoặc slug tùy product/API).

### 1.0.1) View detail (có ít nhất 1 document trong list)

![](./images/View%20document.png)

- Điều kiện hiển thị:
  - User đã có `menuId` trên URL và API LNB trả về **ít nhất 1** document.
- Hệ thống chọn **document đầu tiên** trong list làm document đang xem.
- URL trên trình duyệt chuyển thành:
  - `/admin/content-management/{menuId}/{docId}`  
  - Trong đó `docId` là **id** (hoặc slug nếu API/route chốt) của document đầu tiên trong list.
- Nội dung document được load và render vào `ContentViewPanel`.
- LNB highlight document đang active; TOC bên phải theo nội dung document đang mở.

### Hiển thị theo locale hệ thống (LNB & RNB)

- Khi hiển thị **danh sách document trên LNB** (tên node, nhãn menu trong cây, v.v.) hoặc **dữ liệu của một document trên RNB** (khu vực xem chi tiết / preview nội dung, tiêu đề, TOC gắn với bản xem đó), UI **bắt buộc** dùng ngôn ngữ theo **locale hiện tại của hệ thống** (locale đang active trong app, ví dụ qua i18n / user preference).
- Nếu API trả về đa ngôn ngữ, FE map field theo locale đang chọn; nếu thiếu bản dịch cho locale đó, áp dụng rule fallback đã thống nhất (ví dụ: locale mặc định của product hoặc chuỗi gốc).

## 1.1) App routes cho Content Management (Nuxt file-based routing)

Cấu trúc thư mục `app/pages/admin/content-management/` và vai trò từng file (bỏ qua thư mục `markup/` nếu chỉ là sandbox/dev):

| File (relative `pages/admin/content-management/`) | URL (pattern) | Vai trò chính | LNB | RNB |
| --- | --- | --- | --- | --- |
| `index.vue` | `/admin/content-management` | Entry: redirect tới menu depth2 (GNB) đầu tiên có quyền; modal/popup khi chưa có menu | Không | Không |
| `[menuId]/[[docId]].vue` | `/admin/content-management/:menuId`<br>`/admin/content-management/:menuId/:docId?` | Màn viewer chính: load cây document (LNB), load chi tiết document, empty/loading/skeleton, `ContentViewPanel` + TOC | Có | Có |
| `[menuId]/create.vue` | `/admin/content-management/:menuId/create` | Tạo document: form + editor đã chọn, lưu draft/tạo mới | Có | Có |
| `[menuId]/[docId]/edit.vue` | `/admin/content-management/:menuId/:docId/edit` | Sửa document: load detail, editor, cập nhật/lưu | Không | Có |

`[[docId]]` là tham số **optional**: cùng một component phục vụ cả URL chỉ có `menuId` và URL có thêm `docId`.

### Flow người dùng ↔ file render (ví dụ)

| Hành động | Route ví dụ | File render |
| --- | --- | --- |
| Bấm depth1 Content Management (GNB) | `/admin/content-management` | `index.vue` |
| Hệ thống chọn menu depth2 đầu tiên | `/admin/content-management/start-guide` | `[menuId]/[[docId]].vue` |
| Auto chọn document đầu list | `/admin/content-management/start-guide/sdk-install` | `[menuId]/[[docId]].vue` |
| Bấm document khác trên LNB | `/admin/content-management/start-guide/auth-guide` | `[menuId]/[[docId]].vue` |
| Bấm tạo document (Create) | `/admin/content-management/start-guide/create?editor=markdown` | `[menuId]/create.vue` |
| Bấm Edit | `/admin/content-management/start-guide/sdk-install/edit` | `[menuId]/[docId]/edit.vue` |

### `index.vue` — entry & redirect

- `GET /admin/content-management` hoặc `GET /admin/content-management/`  
  - Map tới `app/pages/admin/content-management/index.vue`.
  - **Không gọi lại API để load list menu GNB** tại route này nếu list đã có trong workspace (ví dụ `AdminLayout.vue` + `useAdminNavbar.ts`); nếu spec product yêu cầu luôn fetch lại thì bổ sung riêng.
  - Nếu đã có danh sách menu GNB (depth2 / content management menus):
    - **redirect** sang `/admin/content-management/{menuId}`.
    - `menuId` lấy từ **item đầu tiên** sau khi sort: **có quyền access trước, không có quyền access sau**.
  - Nếu chưa có list hoặc list rỗng:
    - Modal chọn menu GNB: `관리할 GNB 메뉴를 선택해주세요.` (first-time).
    - Nếu workspace chưa có GNB: popup yêu cầu tạo GNB.
  - Trạng thái:
    - **loading** navbar/menu: chưa redirect/modal.
    - **loaded nhưng empty**: mở modal/popup GNB.

### `[menuId]/[[docId]].vue` — viewer chính

- `GET /admin/content-management/{menuId}`  
  - Sau khi có `menuId` trên URL, load LNB (cây document).
  - **Không có document** → empty state ở main (mục `1.0`); URL giữ `/admin/content-management/{menuId}`.
  - **Có document** → có thể tự chọn document đầu list và **replace** URL thành `/admin/content-management/{menuId}/{docId}` (mục `1.0.1`).
- `GET /admin/content-management/{menuId}/{docId}`  
  - Load chi tiết document tương ứng, render viewer + TOC (RNB).

### `[menuId]/create.vue` — tạo mới

- `GET /admin/content-management/{menuId}/create`  
  - Màn tạo document trong menu hiện tại; LNB + RNB theo spec layout.
- Phân loại editor bằng **query string**:
  - Markdown: `.../create?editor=markdown`
  - Text editor: `.../create?editor=text-editor`
- Sau khi tạo xong (thành công): điều hướng về viewer, ví dụ `/admin/content-management/{menuId}` hoặc thẳng tới `/admin/content-management/{menuId}/{docId}` của document vừa tạo (tùy UX).

### `[menuId]/[docId]/edit.vue` — chỉnh sửa

- `GET /admin/content-management/{menuId}/{docId}/edit`  
  - Màn soạn thảo; **ẩn LNB**, giữ **RNB** (TOC / phụ trợ) theo spec.
  - Loại editor (markdown vs text) **không** bắt buộc đưa lên URL; lấy từ detail document/backend.
  - API load/save vẫn trong namespace:
    - `GET /api/menus/{menuId}/documents/{documentIdOrSlug}`
    - `PUT /api/menus/{menuId}/documents/{documentIdOrSlug}`  
    (`docId` trên URL có thể map tới `documentIdOrSlug` nếu API hỗ trợ.)

### Reload (F5) và phiên đăng nhập

- F5 tại `/admin/content-management`: vào lại `index.vue`, lặp flow redirect/modal.
- Session hết hạn: middleware auth đưa về login; sau login có thể quay lại URL cũ qua `callbackUrl`/`redirect`.

### Xóa document (delete)

- **Không bắt buộc đổi URL** khi xóa: refetch list, cập nhật selection, hoặc điều hướng nội bộ tùy UX.

### Mapping Create / View / Edit / Delete theo route

- **Create**
  - Mở `/admin/content-management/{menuId}/create?editor=markdown` hoặc `...?editor=text-editor`.
  - `POST /api/menus/{menuId}/documents` (tạo node document/category/package theo flow).
  - Sau khi tạo: `navigateTo` về viewer (URL không còn segment `/create` và không còn query `editor` trừ khi product chủ đích giữ).

- **View (read)**
  - `/admin/content-management/{menuId}/{docId}`  
  - `GET /api/menus/{menuId}/documents/{docId}` (hoặc slug nếu API cho phép).

- **Edit**
  - `/admin/content-management/{menuId}/{docId}/edit`  
  - `GET` load, `PUT` lưu.

- **Delete**
  - `DELETE /api/menus/{menuId}/documents` với body `ids`.
  - Không ràng buộc đổi cấu trúc route theo spec này.

### Layout màn soạn thảo (Create / Edit)

- **Create** (`/admin/content-management/{menuId}/create`): có LNB và RNB.
- **Edit** (`/admin/content-management/{menuId}/{docId}/edit`): không có LNB; vẫn có RNB (TOC). Các mục `2)` và `3)` mô tả đầy đủ vùng LNB/RNB — trên route **Edit**, bỏ qua phần LNB trong mô tả bố cục.

## 2) Write Document - Text Editor

![](./images/Text%20Editor.png)

- Bố cục gồm các vùng chính:
  - Header: logo + hành động `작성 취소` và `등록하기`.
  - Thanh tab ngôn ngữ (ví dụ: 한국어, 영어, ...), có locale đang active.
  - LNB: cây document/category/package version.
  - Main editor:
    - `문서명` (required)
    - `문서 제목` (required)
    - `문서 컨텐츠` (required, rich text editor)
  - RNB: khu vực `목차` (TOC).
- TOC gồm:
  - Internal TOC: tự sinh từ heading trong nội dung.
  - External link TOC: người dùng tự thêm/chỉnh/sửa/xóa link.
- Footer action gồm:
  - `미리보기`
  - `템플릿`
  - `다국어 설정`

## 3) Write Document - Markdown Editor

![](./images/Markdown%20Editor.png)

- Cùng flow nhập liệu như Text Editor (header, tab ngôn ngữ, LNB, RNB).
- Khác biệt chính:
  - Vùng content chia 2 panel:
    - Bên trái: Markdown source editor.
    - Bên phải: Preview render.
  - TOC chỉ dùng theo heading nội dung (không có external TOC riêng như rich text).
- Footer vẫn có `미리보기`, `템플릿`, `다국어 설정`.

## 4) Translate Popup (`다국어 설정`)

![](./images/Translate%20Popup.png)

- Bấm `다국어 설정` trên màn hình viết tài liệu sẽ mở modal.
- Thành phần modal:
  - Cột trái: danh sách document/menu để chọn nguồn dữ liệu dịch.
  - Khối source (ngôn ngữ đang active tại màn hình editor):
    - `메뉴`
    - `문서제목`
    - `문서컨텐츠`
  - Khối target:
    - Tab các ngôn ngữ còn lại.
    - Mỗi tab có các field tương ứng `메뉴`, `문서제목`, `문서컨텐츠`.
  - Nút dịch từng ngôn ngữ (icon mũi tên giữa source/target).
  - Nút `AI 전체 번역` để dịch toàn bộ ngôn ngữ target.
- Footer modal:
  - `AI번역내역` (xem lịch sử dịch AI)
  - `저장하기`

## 5) Import GitLab/GitHub (Content Management / Import GitLab)

- Flow import được kích hoạt từ nhánh `Git 에서 불러오기` trong modal `문서 작성`.
- Modal import gồm 2 bước:

### Bước 1: Xác thực nguồn Git

#### 1.1. URL field — validate rules

Field `URL` áp dụng 5 rule theo priority order (chỉ hiển thị rule fail đầu tiên). **Validation chỉ chạy sau khi user click button `유효성 확인`** — không trigger reactive khi user đang gõ.

| # | Rule | Error message | Check |
|---|---|---|---|
| 1 | Required | `Git URL을 입력해주세요.` | Client |
| 2 | Bắt đầu bằng `http://` hoặc `https://` | `https 또는 http 형식만 지원합니다.` | Client |
| 3 | Format đúng `{host}/{owner}/{repository}` (phải có cả owner và repository) | `올바른 Git URL 형식이 아닙니다.` | Client |
| 4 | Không chứa branch/path bổ sung sau `{owner}/{repository}` (ví dụ `/tree/main`, `/-/blob/dev`) | `Git URL에 Branch 또는 Path 정보가 포함되어 있습니다. 제거 후 다시 시도해주세요.` | Client |
| 5 | URL tồn tại trên server | `존재하지 않는 Git URL 입니다.` | Server (API call) |

- Không hạn chế host — `github.com`, `gitlab.com`, `git.sginfra.net`, hoặc bất kỳ Git host nào server có thể truy cập đều được. API sẽ tự fail nếu URL không tồn tại → trả về rule 5.
- Trailing `.git` và `/` ở cuối URL được auto-strip; không gây lỗi.
- URL không parse được bằng `new URL()` cũng map sang error `존재하지 않는 Git URL 입니다.` (gom với rule 5).

#### 1.2. Verify URL flow

- Button `유효성 확인` enable khi input URL có giá trị.
- Click `유효성 확인` → chạy rule 1-4 client-side; nếu fail → hiển thị error tương ứng, không call API.
- Pass client → call `GET /api/gitlab/branches?gitlabUrl=...`:
  - **Public repo**: hiện badge `Public` (xanh dương). Branch selectbox load danh sách branch từ API, auto-select branch đầu tiên, enable.
  - **Private/Internal repo**: hiện badge `Private` (xanh lá). Hiện row `Access Token` để user nhập token. Branch selectbox vẫn disable, chờ token verify xong.
  - **URL không tồn tại** (API trả lỗi): hiển thị `존재하지 않는 Git URL 입니다.`
- Sau verify thành công, button đổi thành `URL 수정`, input URL chuyển sang readonly.

#### 1.3. Access Token flow (chỉ private/internal)

- Button `유효성 확인` cạnh Access Token enable khi token field có giá trị.
- Click → call lại `GET /api/gitlab/branches` với header `X-Private-Token: <token>`:
  - **Thành công**: hiển thị message xanh `정상적으로 확인되었습니다. 연동할 Branch 를 선택해주세요.`, button đổi thành `수정하기`, token input readonly. Branch selectbox load branches, auto-select branch đầu, enable. Button `다음` enable.
  - **Thất bại**: hiển thị error đỏ `Access Token 인증에 실패했습니다. 토큰을 확인해주세요.`. Branch vẫn disable.

#### 1.4. Edit URL / Edit Token (URL dirty tracking)

- Click `URL 수정` → input URL editable; badge/Access Token row/Branch giữ nguyên giá trị.
- Khi user gõ URL **khác** với `verifiedUrl` (URL đã verify trước đó):
  - Ẩn badge `Public`/`Private`.
  - Ẩn toàn bộ Access Token row (input + button + success message).
  - Clear Branch selection về placeholder, disable.
  - Button đổi về `유효성 확인`.
- Khi user gõ URL **trùng** với `verifiedUrl`: badge + Access Token row reappear (giữ trạng thái cũ). Riêng Branch đã clear → user phải chọn lại.
- Click `수정하기` → Access Token input editable, success message ẩn, button đổi về `유효성 확인`. Token value giữ nguyên.

#### 1.5. Branch selectbox

- Luôn hiển thị.
- Disable khi: chưa verify URL, hoặc repo private nhưng chưa verify token, hoặc URL đang dirty.
- Khi enable, load list từ API, auto-select branch đầu tiên.
- Placeholder `연동할 Branch를 선택해주세요.` khi disable hoặc empty.

#### 1.6. Import Type

- Hiện tại UI đang ẩn (commented-out). Sẽ có 2 lựa chọn khi enable:
  - `전체 불러오기 (Full Sync)`
  - `변경된 문서만 불러오기 (Changed Only)`

#### 1.7. Button `다음`

- Enable khi: URL verified + URL không dirty + (public hoặc token verified) + branch đã chọn.
- Click → sang Bước 2.

### Bước 2: Chọn file cần import

- Hiển thị cây file/folder từ repository.
- Hỗ trợ:
  - Chọn toàn bộ.
  - Chọn theo folder (cascade xuống file con).
  - Chọn từng file riêng lẻ.
- File có thể có trạng thái:
  - `신규` (new)
  - `변경` (modified)
- Action cuối:
  - `이전` (quay lại bước URL)
  - `불러오기` (thực thi import các file đã chọn)

## 6) API mapping cho Content Management

Các API dưới đây đều ở namespace `/api/menus/{menuId}/documents...`. `menuId` là ID của menu đang được chọn ở LNB (content tree).

### 6.1. Quản lý cây document/category/package

- **Tạo node mới (document / category / package)**  
  - `POST /api/menus/{menuId}/documents`  
  - Body: theo `createNodeTreeSchema`, cho phép tạo 1 hoặc nhiều node con (document, folder, package version) dưới một parent.  
  - Sử dụng khi:
    - Chọn `문서 등록하기` và hoàn tất form tạo document đầu tiên.
    - Tạo thêm category / package version trong cây nội dung.

- **Lấy cây document theo menu**  
  - `GET /api/menus/{menuId}/documents`  
  - Trả về danh sách node đã filter theo quyền/visibility, sau đó được build thành cây `DocumentTreeNode`.  
  - Sử dụng để:
    - Render LNB (content tree) sau khi vào trang hoặc sau khi tạo/xóa/sort node.

- **Xóa node (nhiều document cùng lúc)**  
  - `DELETE /api/menus/{menuId}/documents`  
  - Body: `{ ids: string[] }` (schema `deleteDocumentsSchema`).  
  - Sử dụng khi:
    - User xóa một hoặc nhiều document/category/package từ cây nội dung.

- **Sắp xếp lại thứ tự/cấu trúc cây**  
  - `PATCH /api/menus/{menuId}/documents/sort`  
  - Body: danh sách `nodeTrees` (schema `sortTreeNodesSchema`) thể hiện thứ tự & quan hệ cha–con mới.  
  - Sử dụng khi:
    - Kéo-thả document/category/package để đổi vị trí hoặc re-parent.

### 6.2. Nội dung document

- **Lấy chi tiết nội dung document**  
  - `GET /api/menus/{menuId}/documents/{documentIdOrSlug}`  
  - Dùng cho:
    - Khi chọn một document bất kỳ trong LNB để load nội dung vào Text editor / Markdown editor (bao gồm các bản dịch).

- **Cập nhật nội dung document**  
  - `PUT /api/menus/{menuId}/documents/{documentIdOrSlug}`
  - Body: theo `updateDocumentContentSchema` (mảng `translations` + `summary`).  
  - Dùng khi:
    - Bấm `등록하기` hoặc Save trong màn hình soạn thảo (Text editor / Markdown editor) để lưu lại nội dung và bản dịch.

### 6.3. Visibility & Permission

- **Cập nhật visibility của document**  
  - `PATCH /api/menus/{menuId}/documents/{documentIdOrSlug}/visibility`  
  - Body: `{ visibility: 'public' | 'private' | ... }` (`updateTreeNodeVisibilitySchema`).  
  - Dùng trong:
    - Màn hình chi tiết document khi Creator thay đổi trạng thái hiển thị (ví dụ: chỉ nội bộ, public, v.v).

- **Gán quyền truy cập document cho user**  
  - `POST /api/menus/{menuId}/documents/{documentIdOrSlug}/permissions`  
  - Body: theo `addTreeNodePermissionsSchema` (danh sách user hoặc group được cấp quyền).  
  - Dùng khi:
    - Creator/관리자 cấu hình chỉ một số người được truy cập/chỉnh sửa document.

- **Thu hồi quyền truy cập document**  
  - `DELETE /api/menus/{menuId}/documents/{documentIdOrSlug}/permissions`  
  - Body: theo `removeTreeNodePermissionsSchema`.  
  - Dùng khi:
    - Xóa bớt user/group khỏi danh sách được cấp quyền document.

### 6.4. Review & History

- **Cập nhật trạng thái review của document**  
  - `PATCH /api/menus/{menuId}/documents/{documentIdOrSlug}/review-status`  
  - Body: `{ status: ReviewStatus }` (`updateDocumentReviewStatusSchema`).  
  - Dùng khi:
    - Creator gửi review, reviewer approve/reject, hoặc cập nhật các trạng thái workflow khác (request, approved, reviewed...).

- **Xem lịch sử chỉnh sửa document**  
  - `GET /api/menus/{menuId}/documents/{documentIdOrSlug}/histories`  
  - Query: theo `documentHistoriesSearchQuerySchema` (filter theo type, date, paging...).  
  - Dùng khi:
    - Mở màn lịch sử/phiên bản document để xem các lần sửa trước đó, compare change log.

### 6.5. GitLab integration (Import GitLab flow)

- **Lấy branch + metadata visibility của repo**
  - `GET /api/gitlab/branches?gitlabUrl={url}`
  - Header optional: `X-Private-Token: <access-token>` (cho private/internal repo).
  - Response: `{ visibility: 'public' | 'internal' | 'private', branches: Array<{ name, webUrl, default }>, errorCode? }`.
    - `errorCode = 'MISSING_TOKEN'`: gọi unauth nhưng repo cần token (private/internal hoặc URL sai).
    - `errorCode = 'INVALID_TOKEN'`: token sai/hết hạn.
  - Dùng cho:
    - Verify URL repo tồn tại (Bước 1.2).
    - Phân biệt public vs private (chuyển sang flow Access Token nếu private).
    - Validate Access Token (Bước 1.3).
    - Load danh sách branch cho selectbox.

- **Lấy file tree / nội dung file của repo**
  - `GET /api/gitlab/files?gitlabUrl={url}&branch={branch}&paths={paths}&hasContent={bool}`
  - Header optional: `X-Private-Token`.
  - Không truyền `paths`/`hasContent=false`: trả về toàn bộ tree (chỉ `.md`).
  - Truyền `paths[]` + `hasContent=true`: trả về nội dung từng file (markdown đã normalize, image link đã re-upload lên CDN).
  - Dùng cho:
    - Bước 2: hiển thị cây file để user chọn.
    - Khi user bấm `불러오기`: fetch content từng file đã chọn rồi register vào hệ thống.

## Ghi chú cập nhật requirement

- Đã chỉnh các điểm mô tả chưa đúng/chưa rõ:
- Sửa link ảnh markdown sang cú pháp đúng `![](...)`.
- Chuẩn hóa tên màn hình `Markdown` (thay cho typo `mardown`).
- Bổ sung đầy đủ flow `Git 에서 불러오기 / Import GitLab`.
- Làm rõ cấu trúc layout theo từng màn hình và các action chính của user.
- Mapping chi tiết các API `/api/menus/{menuId}/documents...` cho từng nhóm hành vi (tree, content, visibility/permission, review, history).
- Route Content Management (Nuxt, cập nhật spec): entry `index.vue`; viewer + `docId` optional → `[menuId]/[[docId]].vue`; create → `[menuId]/create.vue` + query `editor`; edit → `[menuId]/[docId]/edit.vue`; bảng LNB/RNB mục `1.1`; delete không ràng buộc đổi URL.
- LNB (list document) và RNB (dữ liệu document đang xem) hiển thị theo locale hệ thống đang active.
- Bổ sung chi tiết flow xác thực Git URL (5 rule client-side validate + API check), phân biệt public/private repo, flow Access Token, behavior khi sửa URL (dirty tracking).
- Bổ sung API mapping cho GitLab integration (mục 6.5): `GET /api/gitlab/branches` (verify + validate token + load branch), `GET /api/gitlab/files` (load file tree + content).