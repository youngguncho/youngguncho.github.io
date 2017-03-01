---
layout: post
title:  "Initial Jekyll setup!"
date:   2017-02-27 02:07:58 +0900
tags: [jekyll]
description: >
  Jekyll을 이용해서 github.io blog를 생성하는 방법
---

# Create blog using Jekyll

우선 기본 정리만

###### 루비, 지킬 설치
* 
```
sudo apt-get update
sudo apt-get install ruby-full
sudo gem install jekyll
```

### 블로그를 생성할 repository clone
* 
```
git clone https://github.com/<username>/<username>.github.io.git 
```

### Jekyll 기본 테마 설정
* Repository root 폴더에서 jekyll 명령어를 통한 기본 테마 설정, _config, Gemfile등이 생성된다.
```
jekyll new .
```

### Theme 적용 
* 여기선 지킬 테마 중 [Hydejacked](https://github.com/qwtel/hydejack)를 이용하여 테마를 셋업.
* github fork를 이용하는 방법도 있지만 기존 github.io 리포를 미리 만들었다고 보고 theme을 다운받아서 사용한다. 
다운 받은 theme의 내용을 모두 github reop로 카피

### _config.yml 확인
* 블로그의 틀을 잡는 기본적인 configuration 정보를 담고 있는 파일이다. user, url, 폰트 등을 설정할 수 있다. 
  처음 블로그에 테마를 적용하는 경우에는 테마별로 jekyll dependency가 있는 지 확인해야한다. 
  hydejacked의 경우 jekyll-sitemap, jekyll-paginate 이렇게 두 가지 dependency가 필요하다. 

### Gemfile 셋업
* 위에서 언급한 dependency를 설치할 경우에는 Gemfile에서 일괄적으로 설치하도록 하는 것이 간편하다. (뒤의 jekyll serve에서도 마찬가지)
Gemfile에서 플러그인으로 명시된 부분이 있는데 (line 22: group :jekyll_plugins do) 해당하는 부분에 dependency 들을 아래와 같이 명시한다. 
```
# If you have any plugins, put them here!
group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.6"
   gem "jekyll-sitemap"
   gem "jekyll-paginate"
end
```

### Gem install & serve
* Gemfile 셋업을 완료했다면 이제 필요 dependency install과 localhost에서 블로그 테스트를 해본다. 
우선 bundle install을 통해 필요 파일들을 설치한다.
```
bundle insatll
```
* 설치가 끝나면 config를 확인한다. 
여기서 중요한 부분은 url과 baseurl인데 관련 내용은 [링크](https://byparker.com/blog/2014/clearing-up-confusion-around-baseurl/)를 확인하고, url에는 본인 블로그 주소를 입력하고 baseurl을 비워둔다. 그리고 localhost에 블로그 만들어서 확인한다.
```
bundle exec jekyll serve
```

* 그리고 웹브라우저에서 localhost:4000 를 들어가보면 테마가 적용된 블로그가 생성된 것을 확인할 수 있다. 만약 home 어쩌고 하면서 warning이 나오면 불필요한 index.md 파일이 생긴 것이기 때문에 삭제해준다.

### Push to my github.io
* localhost에서 확인이 끝나면 이제 블로그로 push를 한다. 
```
git add -A
git commit -m "Initial blog test"
git push origin master
```    