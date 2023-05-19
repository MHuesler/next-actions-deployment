# Configure NextJS auto-deployment using GitHub Actions

## On VPS
1. Install NodeJS
2. Install Nginx and configure port proxy where the Next app will be served
4. Install Pm2

## Actions Workflow
1. In GitHub Repo -> Actions -> Select NodeJS Workflow
2. Edit publish.yml

```
name: Node.js CI

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [18.x]
        
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    - run: pm2 reload homepage
```

3. Settings -> Actions -> Add Runner

## On VPS
1. Follow Commands from GitHub Add Runner
2. Start pm2 `pm2 start npm --name "next-js" --start`
3. In actions-runner folder
  3.1. `sudo ./svc.sh install`
  3.2. `sudo ./svc.sh start`
  3.3. `sudo ./svc.sh status`
