# InfinitySlideComponent
vue로 개발한 무한 슬라이드 컴포넌트입니다.  
전체 슬라이드가 보이는 상태에서 무한으로 슬라이드가 작동하는 것이 차별화 된 특징입니다.
  
갯수가 홀수인 상태에서 highlight 옵션이 활성화 될 경우 중앙 컨텐츠 강조가 되는 기능을 제공하고 있습니다. 

## 사용법
[예제코드](https://codepen.io/bardstale/pen/vYbQZJm)
```html
<template>
  <infinity-slide
          :slideId="`#infinity-slide`"
          :amount="7"
          :width="200"
          :height="100"
          :widthSmall="150"
          :heightSmall="120"
          :margin="5"
          :transition="0.3"
          :staticPath="`/img/slide`"
          :imgPatternName="'slideImg'"
          :highlight="true"
          @nextStartEmit="startMethod"
          @prevStartEmit="startMethod"
          @nextEndEmit="endMethod"
          @prevEndEmit="endMethod"
  />
</template>

<script>
  export default {
    methods: {
      startMethod({now,prev,next}){
        console.log(now);
      },
      endMethod({now,prev,next}){
        alert(now);
      }
    }
  };
</script>
```

## 속성
**slideId(String)** : 생성될 slide 엘리먼트의 id값.  
**amount(Number)** : 총 슬라이드 개수.  
**highlight(Boolean)** : 중앙 슬라이드 강조 옵션 사용 유무, default는 true. 옵션 사용 시 1번 슬라이드가 중앙으로 오며 양 옆으로 분할하여 배치됨.  
**width(Number)** : highlight 될 슬라이드의 너비. highlight가 false일 경우 일반 슬라이드의 너비.  
**height(Number)** : highlight 될 슬라이드의 높이. highlight가 false일 경우 일반 슬라이드의 높이.  
**widthSmall(Number)** : 강조되지 않은 슬라이드의 너비. highlight가 false일 경우 사용되지 않음.  
**heightSmall(Number)** : 강조되지 않은 슬라이드의 높이. highlight가 false일 경우 사용되지 않음.  
**margin(Number)** : 슬라이드 사이의 간격.  
**transition(Number)** : 슬라이드 이동 트랜지션 시간. 초 단위.  
**staticPath(String)** : 슬라이드에 사용될 이미지의 경로.  
**slideImg(String)** : 슬라이드에 사용될 이미지의 네이밍 패턴. ex)slide로 저장 시 'slide-' 패턴으로으로 읽어옴.(slide-1.png, slide-2.png)  

## 이벤트 메소드
공통적으로 디폴트 파라미터로 슬라이드 상태값을 담은 객체를 제공  
**{now,prev,next}** : 각각 현재, 이전, 다음의 index값을 나타냄.  
  
**nextStartEmit** : 다음 슬라이드로 이동 시 발생하는 이벤트.  
**prevStartEmit** : 이전 슬라이드로 이동 시 발생하는 이벤트.  
**nextEndEmit** : 다음 슬라이드로 이동 완료 시 발생하는 이벤트.  
**prevEndEmit** : 이전 슬라이드로 이동 완료 시 발생하는 이벤트.  
