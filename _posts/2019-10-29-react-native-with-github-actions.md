---
title: 使用 Github Actions 打包 React Native 应用报错 System limit for number of file watchers reached 的解决办法
tags: devops,js
---

# 使用 Github Actions 打包 React Native 应用报错 System limit for number of file watchers reached 的解决办法

Github Actions 限制了 `fs.inotify.max_user_watches=8192` 导致打包过程报错 `System limit for number of file watchers reached`
https://github.com/expo/expo-github-action/issues/20

解决办法是在打包之前修改设置：

```sh
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

```yaml
name: Build

on:
  push:
    branches:
      - dev
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Setup kernel for react native, increase watchers
        run: echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
      - uses: actions/checkout@v1
      - name: Use Node.js
        uses: actions/setup-node@v1
      - name: npm install
        run: npm install
      - name: set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Build with Gradle
        run: cd android && ./gradlew assembleRelease
```

完美解决。
