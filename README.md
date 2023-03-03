# gh-action-poki
### Github Action to upload game to Poki
## Setup
### 1. [Install @poki/cli on your project](https://github.com/poki/poki-cli)
```
npx @poki/cli init --game xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx --build-dir dist
```
```
npx @poki/cli upload
```
> browser window will open to authenticate
### 2. Get contents of `auth.json`
###### Windows
```
type %LOCALAPPDATA%\Poki\auth.json
```
###### Linux
```
cat $XDG_CONFIG_HOME/poki/auth.json
```
###### macOS
```
cat $HOME/.config/poki/auth.json
```
### 3. [Add a secret to your Github repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository)
> Repository settings > Secrets and variables > Actions > **New repository secret**
  * **Name:** `POKI_AUTH`
  * **Secret:** (paste contents of `auth.json` here)
### 4. [Create a workflow](https://docs.github.com/en/actions/quickstart)
  * Add `.github/workflows` folder with new file `poki.yml` (or some other name) inside
###### Example workflow
```yaml
name: Build game, upload to Poki, notify on Discord
on:
  workflow_dispatch:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x]
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - name: Install dependencies
        shell: sh
        run: npm i
      - name: Create build
        shell: sh
        run: npm run build
      - name: Poki Upload
        id: poki_upload
        uses: djlsa/gh-action-poki@v1
        with:
          auth: '${{ secrets.POKI_AUTH }}'
      - name: Discord notify
        uses: sarisia/actions-status-discord@v1
        if: always()
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }}
          description: ${{ steps.poki_upload.outputs.preview_url }}
```