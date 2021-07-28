---
title: "두 에디터 이야기"
date: 2021-07-27T00:04:36+09:00
---

## 프롤로그

얼마 전에 저희는 서비스 중인 [마림바](https://marimba.team/)에서 사용하고 있는 텍스트 에디터의 엔진을 [Quill](https://quilljs.com/)에서 [ProseMirror](https://prosemirror.net/)로 교체하였습니다. [마림바](https://marimba.team/)의 텍스트 에디터는 일반적인 서비스의 텍스트 에디터와 다르게 파워포인트와 같이 보드 내에 존재하는 다양한 오브젝트가 각각의 에디터를 가지고 있는 형태로 사용하고 있는데, 이러한 특성 때문에 에디터는 상황에 따라 크기나 표현할 수 있는 스타일의 제한 등 다양한 제약조건을 가지고 사용할 수 있어야 했고, 캔버스와도 긴밀하게 상호작용하며 움직일 수 있어야 했습니다. 저희는 [Quill](https://quilljs.com/)을 이용하여 [마림바](https://marimba.team/)의 첫번째 버전을 구현하였고, 이 때문에 [Quill](https://quilljs.com/)은 이식받은 인공장기 만큼이나 매우 복잡한 형태로 저희 서비스와 결합되어 있는 상태였습니다.

![Text Editor of Marimba](/blog/images/editor/editor-introduction.png)

사실 [마림바](https://marimba.team/)가 텍스트 에디터를 사용하는 특수성을 감안하지 않더라도 일반적으로 운영 중인 서비스의 주요 모듈을 통째로 교체하는 일은 위험도가 높기에 금기시되는 일입니다. 이 때문에 개발팀에서도 많은 고민이 있었고 교체 가능성을 알아보기 위해서 정말 많은 고민을 거쳐서 다양한 사전 검증, 실증, 개발 및 테스트 과정을 거치면서 에디터를 교체하였는데, 이 글에서는 에디터를 교체하기로 결정하게 된 이유와 교체 과정에 대해서 이야기해보고자 합니다.

## Quill의 이야기

[Quill](https://quilljs.com/)은 2014년 샌프란시스코에서 탄생했습니다. [Quill](https://quilljs.com/)의 부모는 Jason Chen이라는 개발자였는데, Jason Chen은 스탠포드 대학을 졸업한 뒤 젊은 나이에 실시간으로 여러 사람이 동시에 편집할 수 있는 텍스트 에디터를 개발하는 Stypi라는 회사를 창업하였고, 이 회사는 세일즈포스에 2012년 인수됩니다.

Jason Chen과 그의 팀은 세일즈포스에 인수된 뒤에도 Stypi의 개발을 계속했고, Stypi의 핵심기술 몇 가지를 다시 개발하면서 일부를 오픈소스 프로젝트로 공개하게 됩니다. 이 중 텍스트 에디터의 역할을 담당하고 있던 모듈은 [Quill](https://quilljs.com/)이란 이름으로 탄생하게 되고, 당시 비교할만한 오픈소스 에디터가 흔치 않았기에 [Quill](https://quilljs.com/)은 많은 프로젝트의 관심을 받으며 성공적으로 세상에 데뷔합니다.

Jason Chen은 세일즈포스에서 일하면서 회사가 커지면 커질 수록 필요한 정보들이 제대로 문서화되어서 공유되지 않는다는 사실을 알게 되었고, 이러한 문제점을 해결하자는 의지로 2016년 세일즈포스에서 나와 기업을 위한 위키 서비스인 [Slab](https://slab.com/)을 창업하게 됩니다.

[Slab](https://slab.com/)은 문서를 정리하는 서비스였고, 당연히 텍스트 에디팅이 핵심 기능으로 들어가 있었으며, 물론 [Quill](https://quilljs.com/)을 에디터로 사용하였습니다. 이러한 지원에 힘입어 Quill은 지속적으로 버전업을 거듭하며 마침내 2018년 1.0 버전을 릴리즈 하였고, 슬랙, 링크드인, USA Today 등 많은 기업에서 메인 에디터로 채택되며 입지를 다지게 되었습니다.

작년 겨울, 저희 개발팀이 [Quill](https://quilljs.com/)을 [마림바](https://marimba.team/)의 메인 텍스트 에디터로 채택했을때 [Quill](https://quilljs.com/)은 이미 오픈소스 텍스트 에디터계의 슈퍼스타였습니다. [Quill](https://quilljs.com/)의 훌륭한 설계와 폭넓은 확장성은 저희의 복잡한 요구사항을 잘 수용해주었고, 저희는 [Quill](https://quilljs.com/)을 기반으로 다양한 기능을 구현할 수 있었습니다. 하지만 서비스를 베타 오픈하고 다양한 사용자의 요구사항을 받고 서비스를 개선해나가면서 저희는 생각보다 빨리 [Quill](https://quilljs.com/)의 한계를 느끼게 되고 더 늦기 전에 에디터의 교체를 고려해야 하지 않을까 하는 고민을 하기 시작합니다.

## Rich Text Editor의 기본적인 구조

만약 오픈소스를 사용하지 않고 텍스트 에디터를 직접 구현해야 한다면 어디에서부터 시작해야 할까요? 가장 기본적인 형태는 div 영역을 만든 뒤 [contenteditable](https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/contenteditable) 옵션을 활성화하는 것입니다.

```html
<div contenteditable>
Editing Area...
</div>
```

이렇게 생성된 영역은 키인(Key In)이 가능하고 사용자의 입력에 따라 변경 가능한 영역이 됩니다. 여기에는 키보드를 통합 문자 입력과 커서 이동, 마우스 이벤트에 따른 커서와 포커스 이동이 모두 구현되어 있는데, 이 정도로도 들인 노력에 비해 많은 기능을 하고는 있지만 ```<TextArea>```를 대체할 만큼 훌륭해 보이지는 않습니다.

저희가 이야기하는 텍스트 에디터는 일반적으로 Bold, Underline, Italic이나 폰트 색상의 변경, 순서가 있거나 없는 목록의 생성 등 조금 더 복잡한 형태, 즉 리치 텍스트(Rich Text)를 다룰 수 있는 리치 텍스트 에디터를 지칭하는 경우가 많습니다. 이러한 에디터들은 특정 영역을 블럭으로 선택하고 Bold 버튼을 누르면 선택된 글자들은 굵게 표시되고, ```1.```을 입력한 뒤 스페이스를 누르면 순서 있는 목록이 자동으로 생성되고 엔터를 칠때마다 번호가 증가하는 기능들을 지원해야 쓸만하다는 평가를 받을 수 있습니다.

만약 텍스트 에디터를 직접 구현한다면 어떤 영역에 Bold 처리를 할 때, 해당 영역을 ```<b>``` 태그로 감싸주거나, 순서있는 목록을 만들어야 할 때 ```<ol>``` 태그와 ```<li>``` 태그를 조합하여 표현해주는 식으로 기능을 구현할 수 있을 겁니다. 즉, 키보드 및 마우스 이벤트를 핸들링하여 이를 적절한 DOM으로 변환하는 역할을 수행하도록 해주어야 하는데, 조금만 생각해보면 이 작업이 결코 쉬운 작업이 아니라는 것을 알 수 있습니다. 사용자가 선택한 영역은 다행히도 Window 객체의 ```getSelection``` 함수를 이용해서 가져올 수 있지만 이 좌표를 이용해서 DOM을 조작하여 원하는 출력을 보여주는 것은 또 다른 이야기가 됩니다.

만약 제목을 표시하기 위해 특정 영역에 ```<h1>``` 태그를 감싸야하는 경우는 어떨까요? 일반적으로 텍스트 에디터들은 선택한 영역이 없더라도 ```<h1>```태그는 커서가 위치한 줄 전체를 감싸는 방식으로 동작합니다. 그렇다면 커서의 좌표를 기준으로 어떻게 해당 줄 전체를 감싸기 위한 시작점과 종료점을 찾을 수 있을까요? 단축키는 어떤 방식으로 처리해야 할까요? 되돌리기(Undo) 기능은 어떻게 구현할까요? 문서는 어떤 식으로 모델화하고 메모리에 유지할까요? 아니면 순수한 DOM을 계속 조작하는 방식으로 만들어야 할까요? 이러한 구현들이 1초에서 수십 번씩 전달되는 이벤트를 처리하면서도 성능을 잃지 않도록 만들 수 있을까요?

사실 대부분의 기능들은 많은 시간 고민하면 어떻게든 구현할 수 있는 기능들입니다. 다만 투자해야 하는 시간이 매우 많기에 일반적으로는 오픈소스 텍스트 에디터를 사용하여 해당 기능들을 서비스에 통합시키는 선택을 합니다. 하지만 저희가 최초에 [Quill](https://quilljs.com/)을 선택하여 서비스를 구현할 때 ```Canvas``` 기반의 애플리케이션에 사용자의 이벤트를 기반으로 DOM을 직접 편집하는 에디터를 통합시키는 과정은 난관이 많았습니다. 저희 서비스는 기본적으로 협업을 위한 화이트보드 서비스이고, 일반적인 상식으로 화이트보드와 활자는 그렇게 잘 어울리는 조합은 아닙니다. 이는 애플리케이션 레벨에서도 비슷한 문제를 가지고 왔는데, 생각없이 캔버스와 통합한 에디터는 화이트보드에 붙인 A4 용지처럼 어설프게 보일 위험이 있었기 때문입니다.

단순하게 생각하면 텍스트 에디터도 DOM으로 표현되고, 캔버스도 DOM의 일부로 표현되므로 캔버스와 에디터를 같이 표시하는 것은 큰 문제가 없습니다. 하지만 에디터가 중심의 되는 애플리케이션에 캔버스 기능이 보조적으로 통합하는 것과 달리 캔버스가 중심이 되는 애플리케이션에 에디터를 통합하는 것은 고려할 점이 많았습니다. 우선 [마림바](https://marimba.team/)에서 사용하는 보드의 캔버스 위에는 텍스트 외에도 다양한 선, 도형, 구조화된 패드, 스티키 노트, 유튜브 영상 등이 자유롭게 배치될 수 있었고, 스케일과 화면 이동이 자유로워야 하고, 다른 사용자와 공동으로 편집이 가능해야 했기 때문에 이러한 객체들은 모두 모델로 변환되어 저장 및 전송할 수 있어야 했습니다. 이러한 처리를 위해서 저희는 [Konva](https://konvajs.org/)를 사용하여 캔버스 객체들을 다루었는데, 만약 에디터가 캔버스 내부가 아닌 외부에 존재한다면 에디터는 이러한 처리를 모두 독자적으로 처리하면서도 캔버스의 동작과 동기화되어야 했습니다.

여기에서 발생하는 또 다른 문제는 성능 문제였습니다. 일반적인 사용 시나리오에서 하나의 보드에는 적어도 몇 개에서 수십 개의 텍스트 노드가 동시에 존재할 수 있고, 각각의 몇 글자 정도의 짧은 내용만 담고 있는 경우에서 수천 자 분량의 장문을 담고 있는 케이스가 모두 있을 수 있었는데,  이 노드들 역시 [CSS Transform](https://developer.mozilla.org/ko/docs/Web/CSS/transform)을 이용하여 캔버스의 움직임에 따라서 확대, 축소, 이동을 할 수는 있었지만 캔버스에 비해서 월등히 느리게 움직였고 결과적으로 보드 전체의 성능을 저하시키는 문제를 가지게 되었습니다. 사실 테스트해본 결과로는 수십 개가 아닌 하나의 텍스트 에디터만 DOM 형태로 존재하더라도 라이프사이클을 고려하지 않았을 경우 사양에 따라서는 눈에 보일 정도의 성능 저하를 우려해야 할 정도였습니다.

가장 좋은 것은 DOM이 아닌 Canvas에서 직접 동작하는 에디터를 통합시키는 것이었습니다. 하지만 단순한 Plain Text 형태가 아닌 다양한 스타일과 서식이 존재하는 문서를 다룰 수 있는 Canvas 기반의 에디터는 찾아보기 어려웠습니다. 그래서 저희가 선택한 방법은 [Quill](https://quilljs.com/)을 기반으로 에디터 기능을 구현하여 통합시키되, 사용자가 에디팅과 관련된 인터랙션을 주었을 때만 에디터를 새로 생성해서 DOM에 올린 뒤 텍스트를 편집하고, 다른 곳을 클릭하는 등 사용자 인터랙션에 의해 편집이 종료되는 시점에 에디터의 DOM이 가지고 있는 내용, 즉, HTML 문서를 파싱하여 Canvas 내의 그래픽 객체로 변환하고 에디터는 제거하는 방식이었습니다. 하나의 브라우저에서는 동시에 최대 하나의 에디터만 활성화할 수 있었기에 이 방식은 단순하면서도 확실하게 DOM 기반 에디터의 풍부한 기능을 활용하면서도 성능 저하를 일으키지 않을 수 있는 방식이었습니다.

물론 어떤 종류의 텍스트 에디터도 Canvas 객체로의 변환 기능을 제공하지는 않았기에, 에디터와 캔버스 간의 상호 변환을 구현하는 것은 개발팀의 고통스러운 시간이 들어가는 일이었고 이 가운데 일부 기능들은 구현의 어려움으로 인하여 제외되어야 했습니다. 하지만 최종적으로 구현된 에디터는 어느 정도 목표했던 모습에 가까웠고 저희는 텍스트 에디터를 포함한 많은 기능들을 가다듬어서 서비스를 오픈하게 됩니다.

## Quill의 한계와 모듈 교체에 대한 고민

![Usage of Structured Pad](/blog/images/editor/editor-pad-usage.png)

저희가 [마림바](https://marimba.team/)를 오픈하면서 가장 큰 차별점으로 내세운 것 중 하나는 매우 자유로운 형태로 사용할 수 있는 화이트보드에 구조화된 형태의 데이터를 정리할 수 있도록 도와주는 패드(Pad)를 제공한다는 점인데 구현의 관점에서 패드는 캔버스 위에 존재하는 객체들의 그룹이었지만, 멘탈 모델을 고려하면 패드는 다수의 텍스트 데이터의 집합으로 해석할 수도 있었습니다. 서식과 스타일, 링크나 목록 등 다양한 요소들을 가진 텍스트는 그 자체로도 구조화된 모델의 특성을 가지고 있었지만 패드는 그러한 모델들을 다시 한번 구조화하는 역할을 가지고 있었던 것입니다. 때문에 [마림바](https://marimba.team/)에서 텍스트 에디터의 사용 빈도는 꽤 높은 편이었고, 서비스를 오픈하고 개선시켜나가면서 텍스트 편집 기능에 대한 요구사항이나 변경이 꽤 많이 발생할 수밖에 없었습니다.

사실 에디터의 내용을 캔버스 객체로 전환하는 것이나 외부에서 붙여넣기 한 텍스트를 저희 모델에 맞게 변환하는 과정들은 대부분 Quill의 외부에서 처리하는 기능들이었기에 저희가 Quill의 내부, 즉 에디터 자체를 개조해서 사용한 것은 아니었습니다. 전체 디자인에 맞게 UI를 수정하거나, URL 자동 변환 등 필요한 기능을 위해 일부 모듈을 추가하는 등의 작업을 한 정도였습니다. 하지만 서비스를 개선하는 과정에서 에디터 자체에 대한 변경이 필요한 경우가 점점 생겨나기 시작했는데, 예를 들면 다음과 같은 기능들이었습니다.

- 사용자가 외부에서 더블 클릭을 하여 에디터에 진입할 때 클릭한 위치에 편집 커서를 위치하는 기능
- 순서가 있는 목록과 없는 목록을 혼용하여도 전체 목록 맥락에서 순서를 유지하기
- 순서 있는 목록의 시작 숫자를 선택하기
- 목록의 추가 / 삭제 동작을 커스터마이징 하기
- 서식과 스타일의 다양한 확장
- 사용자 멘션 / 코멘트 기능 등의 추가
- 성능과 안정성의 개선

[Quill](https://quilljs.com/)은 기본적으로 문서 모델을 매우 단순하게 가져가고 있으며, 모델에 대한 임의의 조작이 필요할 때는 [Delta](https://quilljs.com/docs/delta/)라는 개념을 이용하여 처리하고 있습니다. 이는 쉽게 생각하면 에디터 내의 문서는 기본적으로 DOM 객체 형태로 다루어지면서도 조작이 필요할 때는 각 글자의 인덱스를 기준으로 글자를 추가하거나, 삭제하거나, 스타일을 변경하는 등의 동작을 수행하는 것입니다. 이렇게 문서를 다루는 [Quill의 API](https://quilljs.com/docs/api/)는 매우 깨끗하고 단순하게 정리되어 있기 때문에 일반적인 용도에서 [Quill](https://quilljs.com/)을 텍스트 에디터로 활용하는 것은 좋은 선택이 될 수 있습니다. 하지만 이러한 종류의 오픈소스들이 그렇듯이, 단순하고 사용하기 쉬운 구조는 특정 목적을 위해 확장하기 어렵다는 단점이 있었고, 저희가 추가하거나 개선하고 싶었던 기능들도 API가 없어서 포기해야 하는 경우가 점점 생겨나기 시작했습니다.

이러한 경험이 쌓이기 시작하자 저희는 고갈된 베스핀 간헐천과 같은 상태가 되어버린 [Quill](https://quilljs.com/)에 계속 의지하다가는 텍스트 에디터가 서비스 개선의 발목을 잡을 수 있겠다는 판단을 하게 되었고 더 늦기 전에 에디터를 교체하는 것이 가능할지, 대안은 있을지에 대해 검토하기 위해 정찰을 보내게 되었습니다.

대안이 될만한 텍스트 에디터를 찾는 과정에서 가장 먼저 고려했던 것은 역시나 캔버스 기반의 텍스트 에디터를 인하우스 방식으로 직접 구현하는 방식이었습니다. 별도의 연결점을 만들지 않고도 저희 서비스에 에디터를 완전히 통합할 수 있었고 원하는 기능을 얼마든지 추가할 수 있다는 장점이 있었지만, 앞에서 말했듯이 리치 텍스트를 지원하는 에디터를 바닥에서부터 만드는 것은 철근 콘크리트로 궤도 엘리베이터를 만들자는 소리만큼이나 막막한 일이었습니다.

다른 대안을 찾기 위해 [Slate](https://www.slatejs.org/), [Draft](https://draftjs.org/), [ProseMirror](https://prosemirror.net/) 등의 오픈소스가 고려되었고, 앞에서 말했듯이 저희의 선택은 [ProseMirror](https://prosemirror.net/) 였습니다. [Slate](https://www.slatejs.org/)는 에디터라기보다는 에디터 프레임워크에 가까웠고, 저희가 원하는 방식으로 확장할 수 있다는 장점이 있었지만 많은 부분을 새로 구현해야 하는 부담이 있었습니다. Draft는 리액트 기반의 에디터였기에 리액트 기반인 저희 애플리케이션과 통합하기에 유리하다는 장점이 있었고, 이미 저희가 [Quill](https://quilljs.com/) 이전에 에디터로 사용하기 위해 탐색 개발을 수행하다가 저희 서비스의 요구사항을 모두 충족하지 못해서 변경했던 이력이 있었습니다. [ProseMirror](https://prosemirror.net/) 는 가장 상세한 문서 모델을 가지고 있었으며, 풍부한 생태계와 API를 가지고 있다는 장점이 있었지만 문서 모델이나 UI의 처리에 DOM을 너무 적극적으로 활용하기에 리액트와 캔버스를 기반으로 한 저희 애플리케이션과 통합이 쉬울까 하는 의문점이 있었습니다.

여러 검토를 통해 저희는 우선 [ProseMirror](https://prosemirror.net/) 를 최우선으로 고려하기로 결정하였는데, 가장 큰 이유는 저희가 텍스트 에디터를 전환하려는 목적 자체가 Quill이 제공하는 API에 한계가 있어서 기능 확장이 어려웠다는 점이었고, [ProseMirror](https://prosemirror.net/) 는 이런 측면에서 정반대 위치에 있는 오픈소스 에디터였기 때문입니다. 실제로 [Quill](https://quilljs.com/)은 사이트에서 [다른 텍스트 에디터와 Quill을 비교하는 문서](https://quilljs.com/guides/comparison-with-other-rich-text-editors)를 작성하여 공개하고 있는데 [ProseMirror와의 비교](https://quilljs.com/guides/comparison-with-other-rich-text-editors/#prosemirror)에는 다음과 같이 기술하고 있었습니다.

> ProseMirror favors broad exposure of API methods, configurations and variables. Quill treats developers as users and designs an organized API surface, judicious in what to expose, sometimes hiding confusing methods or creating new ones that unify several internal operations.

> ProseMirror는 API, 설정, 변수들을 광범위하게 노출하는 것을 선호한다. Quill은 개발자를 사용자라고 생각하고 API를 설계하였으며, 어떤 것들을 노출할지 결정하며, 개발자를 혼란스럽게 할 수 있는 메소드는 숨겨두며 여러 내부 동작을 통합할 수 있는 새로운 API를 만들어서 제공하기도 한다.

이는 [ProseMirror](https://prosemirror.net/)에 비해서 [Quill](https://quilljs.com/)이 사용하기 쉽고 개발자 친화적임을 어필하는 문장이고 사실이었지만, 가능하다면 에디터의 모든 것을 뽑아내서 활용하고 이를 기반으로 새로운 기능을 저희 뜻대로 만들고 싶었던 저희는 오히려 [ProseMirror](https://prosemirror.net/) 로의 전환에 확신을 더해주는 문장이기도 했습니다. 이 시점에서 저희의 고민은 에디터를 정말 전환해야 할까 혹은 어떤 에디터를 사용해야 할까에서 React를 정식으로 지원하지 않고 독자적인 라이프사이클에 따라 이벤트와 UI를 처리하던 [ProseMirror](https://prosemirror.net/) 를 어떻게 React 기반인 저희 서비스와 통합할 수 있을까로 옮겨지게 되었습니다.

## ProseMirror 통합하기

[ProseMirror](https://prosemirror.net/)를 리액트에서 사용하는 방법은 [react-prosemirror](https://github.com/hubgit/react-prosemirror)나 또 다른 [react-prosemirror](https://github.com/tgecho/react-prosemirror)와 같이 클래스 형태로 제공되는 컴포넌트를 사용하는 방식이 있었고, [Remirror](https://remirror.io/)나 [Atalskit](https://atlaskit.atlassian.com/packages/editor/editor-core)과 같이 [ProseMirror](https://prosemirror.net/)를 이용하여 개발된 다른 에디터를 사용하는 방법이 있었습니다. 하지만 전자의 경우 업데이트가 몇 년 전에 끝난 형태였기 때문에 부담스러웠으며, 후자의 경우 다양한 기능을 제공해주기는 하지만 저희는 저희 나름대로 다양한 요구사항을 만족시키기 위해 [ProseMirror](https://prosemirror.net/)를 이리저리 마개조 할 생각이었기에 다양한 기능으로 잘 포장된 에디터를 사용하는 것은 또 그 나름의 부담이 있었습니다.

[ProseMirror](https://prosemirror.net/)와 리액트를 통합하기 위해 다양한 방법을 실험해본 저희가 최종적으로 선택한 방식은 [use-prosemirror](https://github.com/dminkovsky/use-prosemirror)를 사용하는 방식이었습니다. 최근에 개발된 [use-prosemirror](https://github.com/dminkovsky/use-prosemirror)는 Hook 방식으로 [ProseMirror](https://prosemirror.net/)를 사용할 수 있도록 해주는 오픈소스로 [ProseMirror](https://prosemirror.net/)를 그대로 활용하면서도 리액트 라이프 사이클과 [ProseMirror](https://prosemirror.net/)의 라이프 사이클, 상태 변화를 완벽히 일치시켜주는 Hook이었습니다.

[ProseMirror](https://prosemirror.net/)는 실제 DOM 위에 존재하는 에디터를 [EditorView](https://prosemirror.net/docs/ref/#view.EditorView)라고 정의하고 있으며, 이 [EditorView](https://prosemirror.net/docs/ref/#view.EditorView)에 특정한 이벤트가 발생하면 이를 [Transaction](https://prosemirror.net/docs/ref/#state.Transaction) 형태로 포장하여 커서를 이동시키거나 문서 모델을 변경하는 등의 과정을 수행합니다. 이때, 에디터의 상태는 [EditorState](https://prosemirror.net/docs/ref/#state.Editor_State)라는 모델에 저장되며 이 EditorState는 불변 객체로 변경이 생기면 새로운 State가 생성되고 이 State를 [EditorView](https://prosemirror.net/docs/ref/#view.EditorView)에 반영하여 변경이 일어난 부분에 한정하여 UI 갱신하는 형태의 라이프 사이클을 가지고 있습니다. 이러한 컨셉은 리액트의 그것과 매우 유사한데, 실제로 [ProseMirror](https://prosemirror.net/)의 문서를 살펴보면 리액트의 컨셉을 많이 차용하여 만들었다고 적혀있습니다. [use-prosemirror](https://github.com/dminkovsky/use-prosemirror)는 일반적인 리액트 Hook과 동일하게 에디터 생성을 위한 초기 설정을 받아서, 리액트에서 사용 가능한 State를 반환해줍니다. 이 State가 바로 [EditorState](https://prosemirror.net/docs/ref/#state.Editor_State)이기 때문에 개발자는 리액트 레벨에서 [ProseMirror](https://prosemirror.net/) 내부의 State에 접근할 수 있게 됩니다.

```javascript
import {useProseMirror, ProseMirror} from 'use-prosemirror';

return function ProseMirrorEditor() {
    const [state, setState] = useProseMirror({schema});

    return <ProseMirror state={state} onChange={setState} />;
};
```

이러한 방식은 [ProseMirror](https://prosemirror.net/)의 API에 직접 접근하면서도 상태에 대해서는 리액트 레벨에서 처리하고 싶었던 개발팀의 요구사항에 잘 맞았기 때문에 몇 가지 실증을 거친 뒤, 이 방식이 가장 적합한 방식이라는 확신과 함께 [use-prosemirror](https://github.com/dminkovsky/use-prosemirror)를 이용하여 에디터를 통합하는 작업에 들어가게 됩니다. 

기본적으로 [ProseMirror의 문서 모델](https://prosemirror.net/docs/guide/#doc)은 사용자가 정의한 노드(Node)를 기반으로 한 트리 구조이며, 이는 DOM과 유사하고 실제로 각 노드의 정의도 DOM과 일대일 혹은 다대일로 매핑되는 형태로 이루어집니다.

```javascript
paragraph: {
    content: "inline*",
    toDOM() { return ["p", 0] },
    parseDOM: [{tag: "p"}]
}
```

또한 [ProseMirror](https://prosemirror.net/)는 문서 모델의 트리가 지나치게 깊어지거나 복잡해지는 것을 방지하기 위해서 ```<b>``` 태그나 ```<span>``` 태그와 같이 블럭을 차지하지 않는 규격은 마크(Mark)라는 형태의 인라인(Inline) 객체로 표현하도록 하고 있는데, 이는 다음과 같은 형태로 표현할 수 있습니다.

```javascript
link: {
    attrs: {href: {}},
    toDOM(node) { return ["a", {href: node.attrs.href}, 0] },
    parseDOM: [{tag: "a", getAttrs(dom) { return {href: dom.href} }}],
}
```

이러한 노드와 마크 정의를 모으면 에디터에서 실제로 사용하고자 하는 문서 모델을 독자적으로 정의할 수 있게 되는데, 이를 [ProseMirror](https://prosemirror.net/)에서는 [스키마(Schema)](https://prosemirror.net/docs/guide/#schema)라고 지칭하고 있습니다. 저희의 경우 텍스트 노드에서는 목록, 링크, 헤더 등 모든 서식을 다 지원해야 하지만 도형 내에서는 링크가 지원되지 않는다든지, 스티키 노트에서는 아무런 서식도 적용해서는 안된다든지 하는 식으로 에디터가 사용되는 맥락에 따라 다른 문서 모델을 가져야 하는 요구사항이 있었습니다. 이는 상황에 따라 메뉴에 표시되는 아이콘을 제어하는 식으로 구현할 수도 있었지만 사용자가 단축키를 이용하여 기능을 호출하거나 외부에서 서식이 있는 텍스트를 클립보드에 복사해서 붙여넣는 등의 동작을 했을 때 등 다양한 경로로 문서 모델을 활용할 수 있기 때문에 각각의 객체에 따라 다른 스키마를 정의해서 에디터에 적용해주면 해당 스키마에 정의되지 않은 서식들은 어떤 경로로 입력되더라도 모두 일반 텍스트(Plain Text) 형태로 변형되어 에디터에 적용되게 됩니다. 이 방식은 '에디터를 초기화할 때 전달한 스키마를 위배하는 서식은 존재하지 않는다'라는 가정을 할 수 있게 해 주기 때문에 개발자 입장에서는 많은 부분을 배제하고 개발할 수 있도록 해주며, 추후 지원해야 하는 서식이 늘어나더라도 코드에 큰 변경 없이 스키마를 변경하는 수준에서 대응할 수 있게 해 줍니다.

개발 과정에서 저희가 가장 먼저 시작한 일은 [Quill](https://quilljs.com/)을 이용하여 표현하던 문서 서식들을 [ProseMirror](https://prosemirror.net/) 스키마로 모두 옮기는 것이었는데, 하나의 서식이라도 [ProseMirror](https://prosemirror.net/)에서 구현이 불가능하다면 에디터 전환이 불가능했기 때문입니다. 저희가 운영 중인 서비스가 아니었다면 기능을 빼거나 변경하는 형태로 개발을 이어나갈 수도 있었겠지만, 운영중인 서비스에서 기능이 빠지는 것은 좋은 결정이 될 수 없을뿐더러, 기존 데이터와의 호환성 문제 때문에 애초에 저희에게 가능한 선택지가 아니었습니다. 때문에 체크박스가 있는 Todo List와 같이 [Quill](https://quilljs.com/)에는 있으나 [ProseMirror](https://prosemirror.net/)에는 없는 서식을 추가적인 스키마 정의와 코드 추가로 구현이 가능할까에 대해서 알아보는 것이 가장 큰 관건이었고, 스키마와 CSS, 그리고 일부 이벤트 핸들링 코드를 추가하면 커스텀 노드를 만드는 것이 가능하겠다는 판단이 생겨서 기존 기능의 구현은 가능하다는 판단을 내리게 되었습니다.

그래서 [ProseMirror](https://prosemirror.net/)를 애플리케이션에 통합시키고 요구사항을 구현하는 과정은 저희 요구사항에 맞춘 스키마를 정의하는 작업에서부터 시작하여 요구사항에 맞추어 이벤트를 캐치하여 [에디터 상태](https://prosemirror.net/docs/guide/#state)를 조작하는 플러그인을 개발하는 작업으로 이어졌습니다. [ProseMirror](https://prosemirror.net/)는 상태를 조회하거나 조작하고 변형하기 위한 API가 매우 풍부하게 제공되고 있으며, 기본적으로는 상태가 바뀌는 동작은 트랜잭션 형태의 객체로 만들어서 전달하여 반영합니다. 이러한 방식은 대부분의 에디터가 동일하게 사용하는 방식이기 때문에 이해하기가 크게 어렵지는 않았지만, Delta를 사용하여 비교적 단순하게 문서 모델을 조작할 수 있었던 [Quill](https://quilljs.com/)과 달리 [ProseMirror](https://prosemirror.net/)는 노드를 쪼개거나 합치거나 상위 노드로 올리거나 하위 노드로 내리는 등의 레벨로도 조작이 가능했기에 실제로 기능 구현을 위해 에디터 상태를 조작하는 과정을 꽤나 험난했습니다. 예를 들어서 다음 코드는 [ProseMirror](https://prosemirror.net/)에서 제공하는 리스트 노드 컨트롤을 위한 [prosemirror-schema-list 모듈](https://github.com/ProseMirror/prosemirror-schema-list)의 리스트 분리 코드로, 예를 들어서 목록을 편집하다가 사용자가 텍스트의 중간에서 엔터를 쳤을 때 커서 뒤쪽의 텍스트를 이용해서 다음 목록 아이템을 만들어주는 코드입니다.

```javascript
export function splitListItem(itemType) {
  return function(state, dispatch) {
    let {$from, $to, node} = state.selection
    if ((node && node.isBlock) || $from.depth < 2 || !$from.sameParent($to)) return false
    let grandParent = $from.node(-1)
    if (grandParent.type != itemType) return false
    if ($from.parent.content.size == 0 && $from.node(-1).childCount == $from.indexAfter(-1)) {

      if ($from.depth == 2 || $from.node(-3).type != itemType ||
          $from.index(-2) != $from.node(-2).childCount - 1) return false
      if (dispatch) {
        let wrap = Fragment.empty, keepItem = $from.index(-1) > 0

        for (let d = $from.depth - (keepItem ? 1 : 2); d >= $from.depth - 3; d--)
          wrap = Fragment.from($from.node(d).copy(wrap))
        wrap = wrap.append(Fragment.from(itemType.createAndFill()))
        let tr = state.tr.replace($from.before(keepItem ? null : -1), $from.after(-3), new Slice(wrap, keepItem ? 3 : 2, 2))
        tr.setSelection(state.selection.constructor.near(tr.doc.resolve($from.pos + (keepItem ? 3 : 2))))
        dispatch(tr.scrollIntoView())
      }
      return true
    }
    let nextType = $to.pos == $from.end() ? grandParent.contentMatchAt(0).defaultType : null
    let tr = state.tr.delete($from.pos, $to.pos)
    let types = nextType && [null, {type: nextType}]
    if (!canSplit(tr.doc, $from.pos, 2, types)) return false
    if (dispatch) dispatch(tr.split($from.pos, 2, types).scrollIntoView())
    return true
  }
}
```

이 코드는 현재 커서의 위치와 해당 위치의 노드가 가지고 있는 상태를 파악하고, 현재 목록이 가지는 아이템을 가져와서 새로 생성할 아이템을 만들어주며 특정 영역을 잘라내고 다른 영역에 새로 생성한 아이템으로 감싸는 형태로 붙여넣는 과정을 담고 있는데, 이러한 복잡한 처리가 가능하다는 것이 [ProseMirror](https://prosemirror.net/)의 장점이지만 이러한 처리를 위해 복잡한 계산이 필요하다는 점이 [ProseMirror](https://prosemirror.net/)의 단점이었습니다. 특히나 저희 서비스가 목록을 다루는 방식이 [ProseMirror](https://prosemirror.net/)에서 기본적으로 제공하는 방식과 다소 달라서 위 코드를 수정하거나 다시 짜야한다는 사실을 알게 되었을 때는 콘크리트 속에서 수영하는 듯한 기분이 들었습니다. 위 코드에서는 일반적으로 개발할 때 잘 안 쓰는 -2 라든지 -3과 같은 인덱스 조작이 보이는데, 이는 [ProseMirror](https://prosemirror.net/) 문서 모델이 노드를 트리 형태로 가지고 있지만 동시에 인덱스 기반으로도 탐색이 가능하기 때문에 현재 위치의 상위 노드 혹은 상위 노드의 앞쪽 형제 노드의 자식 노드 등을 지칭하는 의미를 함축하고 있었고, 이러한 의미를 파악하고 동작을 수정하는 것은 어느 정도 적응의 시간이 필요했습니다. 다만 [Atlassian에서 공개한 ProseMirror 라이브러리인 prosemirror-utils](https://github.com/atlassian/prosemirror-utils)와 같은 오픈소스의 도움을 받을 수도 있었고, 이미 많은 동작들이 구현되어 모듈 형태로 제공되기 때문에 [ProseMirror DevTools](https://github.com/d4rkr00t/prosemirror-dev-tools)와 같은 개발 도구의 도움을 받아서 동작 구조를 파악해나갈 수 있었습니다. 이러한 과정을 통해 문서 모델에 어느 정도 적응을 한다면 [ProseMirror](https://prosemirror.net/)는 어떠한 종류의 이벤트라도 핸들링하여 어떠한 종류의 모델 조작도 가능하기 때문에 다소 복잡한 인터랙션이 필요한 저희 서비스 같은 경우라면 [ProseMirror](https://prosemirror.net/)를 사용하는 것이 옳았다는 결론을 내릴 수 있었고, 이 시점에 [ProseMirror](https://prosemirror.net/)로의 에디터 전환을 확정하고 일정 계획을 세워서 개발을 진행하게 되었습니다.

## 날아가는 서비스의 엔진을 교체하기

일정상 계획된 [ProseMirror](https://prosemirror.net/)의 반영 시점은 베타 서비스를 오픈한 지 3개월이 약간 안 되는 시점으로 이미 사용자들이 [Quill](https://quilljs.com/)을 이용해 작성한 컨텐츠가 운영계에 꽤 쌓여있는 상황이었습니다. 따라서 [Quill](https://quilljs.com/)에서 작성한 컨텐츠가 [ProseMirror](https://prosemirror.net/)의 스키마와 맞지 않아서 무시되거나 이전과 다르게 표시되는 일이 발생하지 않도록 하는 것이 저희 입장에서는 무엇보다 중요한 일이었습니다. 기본적으로 데이터베이스에 저장된 텍스트 컨텐츠는 HTML 형태이며, [Quill](https://quilljs.com/)이나 [ProseMirror](https://prosemirror.net/)의 문서 모델을 직렬화하여 저장한 것은 아니기 때문에, 상호간의 호환성은 어느 정도는 보장이 되는 상태였습니다. 특히나 [ProseMirror](https://prosemirror.net/)의 스키마를 [Quill](https://quilljs.com/)에서 사용한 서식들에 맞추어서 작성하였고 반영 이전에는 별도의 서식을 추가하거나 하지는 않았기 때문에 기본적인 호환성은 어느정도 확보한 상태였습니다.

하지만 호환이 완벽히 이루어지지는 않았고 몇 가지 문제가 발생하였는데, 가장 대표적인 문제는 서로 목록 서식을 다루는 방식이 달랐던 것입니다. 예를 들어서 [ProseMirror](https://prosemirror.net/)는 ```<li>``` 태그 내의 컨텐츠를 ```paragraph``` 노드로 취급하기 때문에 기본적으로 ```<li><p>목록 아이템</p></li>``` 형태로 데이터를 저장하였고, [Quill](https://quilljs.com/)은 ```<li>목록 아이템</li>``` 형태로 텍스트를 바로 저장하였습니다. 또한 [Quill](https://quilljs.com/)은 중첩된 목록을 허용하지 않고 모든 목록을 플랫한 형태로 표현하면서 ```<li ql-indent='2'>```와 같은 형태로 목록에 인덴트를 준 뒤 CSS를 이용하여 중첩된 목록을 표현하는 방식을 사용하였고, 체크박스 형태의 목록은 체크를 하거나 체크를 해제할 때마다 목록이 합쳐지고 분리되는 등 몇 가지 표준과 다른 모델을 사용하였고, [ProseMirror](https://prosemirror.net/)는 일반적인 형태의 목록 표현을 사용하였습니다. 당연하지만 ql-indent와 같은 속성으로 표현된 목록은 [ProseMirror](https://prosemirror.net/)에서 지원하지 않는 형태였고, [Quill](https://quilljs.com/)에서 작성된 중첩된 목록은 모든 인덴트가 무시되며 [ProseMirror](https://prosemirror.net/)에서는 플랫한 형태로 표현되었습니다.

이러한 문제점을 해결하기 위하여 데이터를 일괄 변환하거나, 프론트 엔드 레벨에서 이전 데이터를 볼 때마다 변환을 시도하는 등의 몇 가지 옵션이 고려되었는데, 저희는 리스크를 줄이기 위하여 [ProseMirror](https://prosemirror.net/)의 목록 스키마에 ```ql-indent``` 속성을 추가하고 CSS를 이용하여 인덴트를 추가하는, [Quill](https://quilljs.com/)과 같은 접근 방식을 사용하였습니다. [Quill](https://quilljs.com/)이 [ProseMirror](https://prosemirror.net/) 방식의 스키마를 해석하는 것은 어려운 일이었지만 [ProseMirror](https://prosemirror.net/)는 스키마 정의를 유연하게 할 수 있었기 때문에 [Quill](https://quilljs.com/)이 사용하는 형태로 데이터 모델을 정의하고 관련된 플러그인을 작성하는 것이 비교적 자유로웠기 때문입니다. 이는 [Quill](https://quilljs.com/) 이나 다른 에디터에서 [ProseMirror](https://prosemirror.net/)로의 전환은 가능한 일이었지만, 그 반대 방향으로의 전환은 매우 어려운 일이라는 결론으로 이어질 수 있었습니다.

하지만 문제는 [ProseMirror](https://prosemirror.net/)의 경우 스키마에 정의되지 않은 HTML 형식을 만나면 모두 무시하고 일반적인 텍스트로 해석하는 특징이 있는데, 이는 문서 모델에 예외가 존재하지 않는다는 점에서 분명히 큰 장점이 되기도 하지만, 저희처럼 기존 데이터가 있는 상황에서는 자칫 이전 에디터에서는 표현했지만 새 에디터에서는 표현되지 않는 데이터가 있을 경우 사용자가 해당 데이터를 클릭하여 에디팅을 하는 순간 [ProseMirror](https://prosemirror.net/)가 그 데이터의 서식을 제거하고, 실시간으로 저장되는 서비스의 특성상 바로 저장해버려서 데이터가 유실될 가능성이 분명 존재했습니다.

그래서 저희는 개발계에 이전 버전의 에디터를 사용하는 프론트 엔드와 새 버전의 에디터를 사용하는 프론트 엔드를 동시에 배포한 뒤, 같은 데이터베이스를 바라보게 만들어서 이전 버전에서 작성한 컨텐츠들이 새 에디터에서 잘 보이는지 계속 검증하고, 일치하지 않는 부분들의 스키마를 추가하거나 일부 데이터 컨버팅 함수를 추가하는 형태로 반영을 준비하였습니다. 하지만 사용자들이 실제로 사용하는 데이터에 접근할 수는 없었기 때문에 저희가 생각하지 못한 케이스, 즉 서식을 독창적인 형태로 사용하거나 예상하지 못한 양의 데이터가 존재한다든지 하는 등의 위험은 존재하고 있었기 때문에 실제 반영이 이루어지고 나서는 한동안 긴장을 놓칠 수 없었습니다.

새 버전의 에디터를 포함한 서비스가 배포된 뒤에는, 다양한 형태로 테스트를 진행하였습니다. 가장 먼저 이전 버전에서 작성한 문서들을 열어보고 편집하고 수정한 뒤 다시 저장하는 등의 기본적인 동작에서부터 새로 추가된 기능들이나 변경된 부분들을 테스트해보고, 의도와 다르게 동작하는 부분이 없는지 등에 대하여 검증을 진행하였습니다. 특히나 저희는 공동 편집이 가능한 화이트보드를 서비스하고 있기 때문에 다양한 사용자들이 동시에 사용하는 더 복잡한 시나리오가 존재할 수 있어서 그런 부분에 있어서 개발팀 전체가 반영 후 서비스 점검을 마치고 오픈할 때까지, 그리고 오픈 한 이후에도 한동안 지속적인 테스트를 수행하였습니다.

테스트가 종료되고 새로운 버전이 서비스되는 중에도, 문제가 생길 수 있는 부분에 대한 대비는 계속하고 있었습니다. 검증 과정에서 사용하였던 이전 버전의 에디터를 배포한 개발계를 계속 유지하면서 문제가 생긴 케이스에 대한 검증을 진행할 수 있는 환경을 준비해두었으며, 혹시나 새로운 에디터로 접근한 데이터가 깨지거나 복구 불가능한 상태로 변경될 위험에 대비하여 데이터 백업을 준비해두어 문제가 발생하였을 때 언제라도 롤백할 수 있게 대비하고 있었고, 다행히도 백업 데이터를 사용하는 일은 일어나지 않았습니다.

사실 이미 오픈해서 사용자들이 사용하고 있는 서비스의 많은 부분을 차지하는 코어 모듈을 완전히 교체하는 일은 정상적인 개발팀의 의사 결정 과정에서는 거의 일어나지 않을 일이고 그만큼이나 위험한 일입니다. 소프트웨어 개발팀은 언제나 위험과 가치, 고생과 보람의 사이에서 최선의 선택을 하기 위해서 고민하고 있는데, 저희 역시 에디터를 교체하면서 어떻게 하면 위험을 최소화하면서 필요한 것을 얻을 수 있을까에 대해서 고민을 많이 했습니다. 어느 정도 에디터의 교체가 안정적으로 이루어진 지금 시점에 저희 개발팀은 에디터에 새로운 기능들과 서식, 스타일들을 추가하기 위해 요구사항을 식별하고 사용자의 피드백을 받아서 기능과 인터랙션을 개선하는 작업을 진행하고 있습니다. 최초에 에디터를 교체하고자 하는 수요 자체가 추가 기능의 개발을 쉽게 할 수 있도록 더 많은 API와 커스터마이징 요소를 지니고 있는 에디터 엔진을 이식하는 것이었으므로 교체가 완료된 다음에는 본격적으로 기능 개발을 시도할 여지가 생겨났고, 실제로 개선작업이 이전에 비해서 훨씬 수월해진 것을 느낄 수 있었습니다.