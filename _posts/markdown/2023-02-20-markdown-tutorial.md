---
title: 기본적인 마크다운(Markdown) 문법
date: 2023-02-20 14:37:39 +/-TTTT
categories: [Markdown]
tags: [markdown, chatgpt_used]     # TAG names should always be lowercase
# thumbnail: "/assets/img/default_thumbnail1.png"
---

본 문서는 chatGPT에게 "마크다운 문법을 자세히 알려줘" 에 대한 chatGPT의 내용을 기반으로 필자가 추가적인 정보를 덧붙이고 편집하여 만든 튜토리얼입니다. 
또한 지속적으로 필자가 마크다운을 사용하며 자주 실수하는 내용들에 대한 내용을 업데이트합니다.


# 마크다운
마크다운(markdown)은 문서를 쉽게 작성하고 서식을 지정하기 위한 경량 마크업 언어입니다. 이 문법을 사용하면 HTML과 같은 더 복잡한 마크업 언어를 사용하지 않고도 문서의 서식을 간단하게 지정할 수 있습니다.

다음은 마크다운에서 사용할 수 있는 주요 문법입니다.

# 제목

제목을 지정할 때는 `#` 기호를 사용합니다. `#`의 개수에 따라 다른 수준의 제목을 지정할 수 있습니다. 예를 들어, `#` 하나는 1단계 제목을, `##`는 2단계 제목을, `###`는 3단계 제목을 나타냅니다.

```markdown
# 1단계 제목
## 2단계 제목
### 3단계 제목
```

# 1단계 제목
## 2단계 제목
### 3단계 제목

# 리스트

리스트를 만들 때는 `*`나 `-` 기호를 사용합니다. `*`와 `-` 중에서 어느 것을 사용해도 상관 없습니다. 숫자 리스트를 만들 때는 숫자와 마침표(`.`)를 사용합니다.

```markdown
* 항목 1
* 항목 2
  * 하위 항목 1
  * 하위 항목 2

1. 순서 있는 항목 1

2. 순서 있는 항목 2
```

* 항목 1
* 항목 2
  * 하위 항목 1
  * 하위 항목 2

1. 순서 있는 항목 1

2. 순서 있는 항목 2


# 강조

단어나 구를 강조할 때는 해당 부분을 `*` 또는 `_`로 감싸면 됩니다. `*`와 `_` 중에서 어느 것을 사용해도 상관 없습니다. 강조한 부분을 굵게 만들려면 `**` 또는 `__`로 감싸면 됩니다.

```markdown
*기울임* _기울임_

**굵게** __굵게__
```

*기울임* _기울임_

**굵게** __굵게__

# 링크

링크를 추가할 때는 `[링크 텍스트](URL)`와 같이 작성합니다.

```markdown
[구글](https://www.google.com)
```

[구글](https://www.google.com)


# 이미지

이미지를 추가할 때는 `![대체 텍스트](이미지 URL)`와 같이 작성합니다.

** [대체 텍스트란?](#대체-텍스트alternative-text-alt-text)

```markdown
![고양이](https://hyunwoo1123.github.io/assets/img/markdown/cat.jpeg)
```

![고양이](https://hyunwoo1123.github.io/assets/img/markdown/cat.jpeg)


## 대체 텍스트(Alternative text, alt text)
대체 텍스트(Alternative text, alt text)는 웹 페이지에서 이미지를 불러올 수 없는 상황에서 대신 보여줄 수 있는 텍스트입니다. 대체 텍스트는 시각 장애인이나 이미지 로딩에 실패한 사용자 등이 이미지에 대한 정보를 이해하는 데에 도움을 주며, 검색 엔진이 웹 페이지의 내용을 이해하는 데에도 중요한 역할을 합니다.

이를 이용해 웹 페이지에서 이미지가 불러와지지 않을 경우, 해당 이미지의 대체 텍스트를 보여줄 수 있습니다.

아래는 마크다운에서 이미지 경로가 잘못되어 이미지 로드에 실패했을 때 대체 텍스트가 보여지는 예시입니다. 깨진 아이콘에 마우스를 가져다 대면 다체 텍스트 `고양이`를 확인할 수 있습니다.

```markdown
![고양이](https://blablabla.com/blabla/blabla)
```

![고양이](https://blablabla.com/blabla/blabla)


## 이미지 링크

링크가 포함된 이미지를 추가할 때는 `[![대체 텍스트](이미지 URL)](링크)`와 같이 작성합니다

```markdown
[![고양이](https://hyunwoo1123.github.io/assets/img/markdown/cat.jpeg)](https://www.google.com)
```

[![고양이](https://hyunwoo1123.github.io/assets/img/markdown/cat.jpeg)](https://www.google.com)


위 고양이 이미지를 클릭하면 구글 홈페이지로 연결됩니다.


# 인용

인용문을 추가할 때는 `>` 기호를 사용합니다. 인용문 안에 인용문을 추가하려면 `>` 기호를 연속해서 사용하면 됩니다.

```markdown
> 인용문
>> 중첩 인용문
```

> 인용문
>> 중첩 인용문


# 코드

코드 블록을 추가할 때는 \` 와 \`\`\`를 사용합니다. \`는 한 줄의 코드를 작성할 때 사용하고, \`\`\` 는 여러 줄의 코드를 작성할 때 사용합니다.

그리고 \`\`\`를 일반적으로 사용하는 것이지, 꼭 \` 를 3개 붙여쓸 필요는 없습니다. 필요에 따라 4개 이상으로 감싸도 됩니다.

**중요한 것은** \`\`\`과 그 위의 본문 사이에는 반드시 빈 줄이 있으면 안됩니다. 예를 들어

```python
def greet(name):
    print(f"Hello, {name}!")
```

이 코드를 코드 블록으로 추가할 때, 만약,

````markdown
```python

def greet(name):
    print(f"Hello, {name}!")
```
````

이렇게, \`\`\`python 라인과 내부 코드 사이에 빈 줄이 한 줄 이상 작성한 경우, 코드의 라인이 알맞게 보이지 않는 오류가 발생합니다.

\`\`\`python 줄 바로 다음 줄에 `def greet(name):` 가 작성되어야 합니다.

## 코드 블록에 제목 추가

위 코드 블록에는 해당 코드가 python으로 작성되었음이 명시되어있습니다.

위와 같이 코드 블록에 제목을 명시하려면, 시작하는 \`\`\` 뒤에 코드 블록의 제목을 명시하면 됩니다.

**중요한 것은** 제목을 전부 **소문자**로 작성해야 합니다.

````markdown
# 제목을 포함한 코드 블록

```python
def say_hello(name):
    print(f"Hello, {name}!")
```

````

위 경우 코드 블록 내에 \`\`\`를 표기해야 하기 때문에, 실제 마크다운은 아래와 같이, 4개의 \` 으로 감싸 작성되었습니다.

`````markdown
````markdown
# 제목을 포함한 코드 블록

```python
def say_hello(name):
    print(f"Hello, {name}!")
```

````
`````


# 수평선

마크다운에서는 수평선을 추가할 수 있습니다. 수평선은 문서 내에서 새로운 섹션(section)을 만들거나, 내용의 구분을 위해 사용됩니다. 수평선을 추가하는 방법은 아래와 같습니다.

1. 하이픈(`-`)을 연속해서 세 개 이상 작성
2. 별표(`*`)를 연속해서 세 개 이상 작성
3. 언더바(`_`)를 연속해서 세 개 이상 작성

세 가지 중 어떤 것을 사용해도 수평선으로 인식되며, 수평선 아래쪽에는 자동으로 개행이 추가됩니다.

아래는 세 가지 방법으로 수평선을 추가하는 예시입니다.

```markdown
---

***

___
```

---

***

___

위와 같이 어떻게 작성하더라도 수평선이 출력됩니다.