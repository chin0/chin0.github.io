---
title: "오늘 푼 리버싱 문제"
categories: reversing
comments: true
---

## jit in my pants
매우 자명하게도 jit의 동작방식을 생각해보면 mprotect와 memcpy를 호출하는 쪽에서 jit코드를 복사하는 부분이 있을것임.(내가 개발자라면 그렇게 함.)

일단 jit코드 memcpy에서 복사하고 실행권한 주는부분 브포걸어서 바로 코드 얻을수 있고 복호화 루틴 분석해보면 걍 xor 5 -1 하는거라서 그냥 복호화 해주면 댐

### todo
- rust로 myjit같은 가벼운 jit library 만들어보기

## momo
1줄요약: 눈물과 고통의 movfuscator
열심히 보고 복호화하면 된다.
https://github.com/xoreaxeaxeax/movfuscator
