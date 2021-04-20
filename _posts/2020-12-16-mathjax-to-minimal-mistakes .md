---
layout: post
title: "Apply mathjax to Minimal Mistakes theme "
date: 2020-12-16 23:53:30 -0400
categories: tips
use_math: true
---

# Minimal Mistakes에서 mathjax 사용하기

출처 : <https://www.janmeppe.com/blog/How-to-add-mathjax-to-minimal-mistakes/>

​           <https://mkkim85.github.io/blog-apply-mathjax-to-jekyll-and-github-pages/>





## How to apply Mathjax to Minimal mistakes

Minimal mistakes theme을 fork 해온후에 Mathjax를 적용시켜보자.



#### Step 1. Set markdown engine to kramdown

---

_config.yml 파일에서 다음 부분을 찾아 다음과 같이 바꿔준다. 아마 기본값으로 되어있을 것이다.

```yml
# Conversion
markdown: kramdown
highlighter: rouge
lsi: false
excerpt_separator: "\n\n"
incremental: false
```



#### Step 2. _includes/mathjax_support.html 파일 생성

---

_includes 디렉토리에 mathjax_support.html 파일을 만든 후에 다음의 내용을 입력한다.

```html
<script type="text/javascript" async
	src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML">
</script>

<script type="text/x-mathjax-config">
   MathJax.Hub.Config({
     extensions: ["tex2jax.js"],
     jax: ["input/TeX", "output/HTML-CSS"],
     tex2jax: {
       inlineMath: [ ['$','$'], ["\\(","\\)"] ],
       displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
       processEscapes: true
     },
     "HTML-CSS": { availableFonts: ["TeX"] }
   });
</script>
```



#### Step 3. _layouts/defalut.html 수정

---

다음 부분을 head 사이에 끼워넣는다.
{% raw %}

```html
{% if page.use_math %}   
 {% include mathjax_support.html %}
{% endif %}
```
{% endraw %}



#### Step 4. 올릴 _posts/yourfile.md 파일에 use_math: true 적용

---

```yml
---
title: "Hough Transform 코드 분석"
date: 2020-12-10 14:42:30 -0400
categories: hough transform
use_math: true
---
```

앞에 use_math: true를 추가해준다.



#### Step 5. 적용

---

##### 글 사이사이에 수식 끼워넣기

```latex
이렇게 $\rho$의 범위는 $-\rho_{max}\leq \rho\leq \rho_{max}$ 이다.
```

이렇게 $\rho$의 범위는 $-\rho_{max}\leq \rho\leq \rho_{max}$ 이다.

##### 수식블록 사용

Ctrl+Shift+M을 누르거나 $$+Enter를 치면 수식 블록이 생긴다.

```latex
$$
y=ax+b ↔ r=acos\theta + bsin\theta
$$
```

$$
y=ax+b ↔ r=acos\theta + bsin\theta
$$



수식은 <https://www.codecogs.com/latex/eqneditor.php> 여기에서 쓰고 복붙하면 편하다.



