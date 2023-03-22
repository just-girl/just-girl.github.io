---
title: Container 的布局表现行为
tags: ["Flutter"]
categories: Flutter
date: 2021-06-24
grammar_cjkRuby: true
copyright: ture
---

因为 Container 包含多个子组件，并且每个子组件都有自己的布局表现，所以Container的布局表现有些复杂。

<!-- more -->

简单来说，Container 会按照以下**顺序**进行尝试：
1. 根据 alignment；
2. 根据子组件调整自己的大小；
3. 根据自身的 width 、height 和  constraints；
4. 尝试扩大自身的大小以适应父组件；
5. 尽可能的缩小。

特别的：
1. 如果没有子组件、没有 height 、没有 width 、没有 constraints 、父组件没有提供 constraints，则 Container 的尺寸会尽可能的小；
2. 如果没有子组件、没有 alignment 、但提供了 height 、width 、或者 constraints，则 Container 会在给定的自身约束和父约束的前提下的尺寸会尽可能的小；
3. 如果没有子组件、没有 alignment 、没有 height 、没有 width 、没有 constraints，但父组件提供了 constraints，则 Container 的尺寸会和父组件的尺寸一样；
4. 如果有 alignment 、父组件没有提供 constraints，则 Container 的尺寸会和子组件的尺寸一样；
5. 如果有 alignment 、父组件提供了 constraints，则 Container 的尺寸会和父组件的尺寸一样，然后根据 alignment 放置子组件；
6. 如果没有 alignment 、没有 height 、没有 width 、没有 constraints，则子组件、Container、父组件三者的尺寸一样大。

