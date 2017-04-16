---
layout: post
title:  "Add tags to the blog and posts"
date:   2017-04-16 02:07:58 +0900
tags: [Blog]
description: >
  Hydejack Theme Blog에서 포스트와 Tag 관리하는 방법
---

github.io 에서 블로그를 작성하다보면 생각보다 까다로운 부분이 각 포스트에 태그를 할당하고 블로그에서 분류되도록 하는 것이다. 각 테마별로 태그 시스템이 조금씩 다른데 현재 블로그의 테마는 [Hydejack](https://github.com/qwtel/hydejack) 이므로 해당 테마에서 사용하는 태그 추가 방법에 대해서 살펴본다.

## 블로그에 새로운 태그 추가하기

1. `_data/tags.yaml` 에 태그 추가하기

    태그 이름, 색, 설명 등을 추가할 수 있다.
    ```yml
    mytag:
      name: mytag
      color: '#949667'
      description: >
        Some explanation.
    ```

2. `tag` 폴더에 위의 yaml에서 사용한 '태그 이름'과 같은 이름의 파일 만들기

    Example `/tag/mytag.md` :
    ```md
    layout: blog-by-tag
    tag: mytag
    permalink: /tag/mytag/
    ```
3. 사이드 바에 Tag List를 만들기 위해 `_config.yml`의 `sidebar_tags`에 리스트 추가
    ```yml
    sidebar_tags: [mytags, othertag]
    ```

4. 이제 각 포스트에서 `tags` key를 명시해주면 태그를 지정할 수 있다.

    각 포스트에서 상단
    ```md
    layout: post
    title: Introducing My New Tag
    tags: [mytag, othertag]
    ```
