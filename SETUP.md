# lhoris 님의 개인 Wiki 초기 구축 기록

## 목표

Obsidian으로 글을 작성하고, Quartz로 정적 웹사이트를 생성한 뒤, GitHub Pages에 배포할 개인 LLM Wiki 저장소를 구축한다.

### 1. Get Started

Quartz requires **at least [Node v22](https://nodejs.org)** and npm v10.9.2 to function correctly. Ensure you have these installed on your machine before continuing.

```bash
# 1. Clone the Quartz repository
git clone https://github.com/jackyzha0/quartz.git lhoris.github.io
cd lhoris.github.io
 
# 2. Install dependencies
npm i
 
# 3. Initialize your site (choose a template, set your base URL, import content)
npx quartz create
 
# 4. Install plugins referenced by your chosen template
npx quartz plugin install --from-config
 
# 5. Preview your site locally
npx quartz build --serve
```

Your site is now running at [http://localhost:8080](http://localhost:8080). From here:

- Write content in the content/ folder
- Push to GitHub with npx quartz sync
- Deploy to GitHub Pages, Cloudflare, Netlify, or Vercel

### 2. GitHub Pages

In your local Quartz, create a new file quartz/.github/workflows/deploy.yml.

`quartz/.github/workflows/deploy.yml`

```yml
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      - main
 
permissions:
  contents: read
  pages: write
  id-token: write
 
concurrency:
  group: "pages"
  cancel-in-progress: false
 
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v6
        with:
          node-version: 24
      - name: Cache dependencies
        uses: actions/cache@v5
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      - name: Cache Quartz plugins
        uses: actions/cache@v5
        with:
          path: .quartz/plugins
          key: ${{ runner.os }}-plugins-${{ hashFiles('quartz.lock.json') }}
          restore-keys: |
            ${{ runner.os }}-plugins-
      - name: Install Dependencies
        run: npm ci
      - name: Install Quartz plugins
        run: npx quartz plugin install
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public
 
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 3. Setting Github Repository
Settings → Pages 에서 Build and deployment → Source → **GitHub Actions** 선택

```text
Build and deployment

Source
○ Deploy from a branch
● GitHub Actions
```

### 4. Git

```bash
git branch -M main

git remote remove origin

# 선택 사항: Quartz 원본 upstream 제거
git remote remove upstream

git remote add origin https://github.com/lhoris/lhoris.github.io.git

git add .
git commit -m "chore: initialize quartz wiki"
git push -u origin main
```

### 5. 배포 확인

브라우저에서 아래 주소로 접속한다.

> [https://lhoris.github.io](https://lhoris.github.io)