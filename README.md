# CD note

直接建置一個太麻煩了，只寫一些筆記

基本上會有 preview, staging, production 三個環境來對應產品的測試，灰度和生產環境。

灰度和生產的環境配置應相同。

Something like:

```dotenv

# .env.production

NODE_ENV=production
DEPLOYMENT_ENV=prod # 👈 detect deployment env
API_HOST=https://api.product.com

# .env.staging

NODE_ENV=production
DEPLOYMENT_ENV=staging
API_HOST=https://api.product.com

# .env.preview

NODE_ENV=development
DEPLOYMENT_ENV=preview
API_HOST=https://preview-api.product.com

```

部署應滿足緩存，測試和回滾的功能。

產品在部署時應要有內容滿足可定錨到 commit 精確度，可以透過在 build 的時候取得 commit hash 並置入生成的 html 中。

For example:

```html
<head>
  <meta name="commit-hash" content="{{ COMMIT_HASH }}" />
</head>
```

Production 和 Staging 可以把 build result 直接放入 S3 並由 cloudfront 做快取層，
基於它們應該是共識只有單一存在的環境(不像 open 的 PR 可以是多個的，main 和 release branch 都只會有一個)。

Preview 基於僅能在 PR 做部署，所以會透過建置 Docker image 來組成環境並部署。

Something like:

```yml
- uses: khan/pull-request-comment-trigger@v1.1.0
    id: check
    with:
        trigger: "/deploy-preview"
        reaction: rocket
    env:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"

- name: Checkout
    uses: actions/checkout@v3
    if: steps.check.outputs.triggered == 'true'

- name: Build Docker Image
    # ...
```

可以用 on push，merge to specific branch, tag 去 trigger production publish。
