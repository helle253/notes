---
id: tkdutmzpz7rzne28r9m680h
title: Dendron
desc: ''
updated: 1687532115827
created: 1687532011688
---
Dendron uses Antd for its component styling.

In order to create a custom style, we gotta override the theme. Antd's v5 release drops less support in favor of tokens, but Dendron wants a css file. [You can use this npm package](https://www.npmjs.com/package/@emeks/antd-custom-theme-generator) to generate an antd-friendly css file. 

[References Antd less variable source](https://github.com/ant-design/ant-design/blob/80110b54242552f78eaa93b818d4dd0cbed65f89/components/style/themes/default.less#L31)
