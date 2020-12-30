---
title: [BOJ]2146번: 다리 만들기
author: cotchan
date: 2020-12-30 22:41:21 +0800
categories: [Algorithm, BOJ]
tags: [boj]
---

## 문제링크

+ [문제링크](https://www.acmicpc.net/problem/2146)

---


## 해결 방법

BFS 문제입니다.    
컴포넌트들을 제외한 `바다 칸`을 대상으로 `BFS`를 돌려서 서로 다른 두 컴포넌트가 만나도록 합니다.         
1. 2차원 배열에 대해 BFS를 돌려 컴포넌트의 갯수를 구합니다.
2. 구한 `컴포넌트들의 가장자리 칸`(바다와 연결되어 있는 칸)만 뽑아내어 큐에 넣어줍니다.
3. 2번에서 구한 칸들을 대상으로 4방향 BFS를 돌립니다.
4. BFS를 돌리는 와중에 서로 다른 두 컴포넌트가 만난다면 그 거리가 최단거리가 됩니다. 
 

---

+ [코드](https://github.com/cotchan/algorithm/blob/main/cpp/BOJ/BOJ2146.cc)