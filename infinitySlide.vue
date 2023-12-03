<template>
  <div class="slide-box slide-box__slide-wrap" :id="slideId.replace('#','')" :style="css_var">
    <div class="slide-box__slide-box" @click="clickSlideContents">
      <!--   중앙 하이라이트가 있을 경우 1을 기준으로 좌우 분배, 갯수가 홀수일 경우에만 가능  -->
      <template v-if="isHighlight">
        <!--   애니매이션을 위한 좌측 숨겨진 1개   -->
        <div :class="`slide-box__slide slide-box__slide--${divideCount+1}`" :data-slide-num="divideCount+1"></div>

        <!--   좌측 리스트   -->
        <template v-for="num in divideCount" :key="`slideList-${divideCount+num+1}`">
          <div v-if="divideCount+num+1 === amount" :class="`slide-box__slide slide-box__slide--${amount} slide-box__slide--prev`" :data-slide-num="amount"></div>
          <div v-else :class="`slide-box__slide slide-box__slide--${divideCount+num+1}`" :data-slide-num="divideCount+num+1"></div>
        </template>

        <!--  1번은 중앙    -->
        <div :class="`slide-box__slide slide-box__slide--1 slide-box__slide--now`" :data-slide-num="1"></div>

        <!--   우측 리스트   -->
        <template v-for="num in (divideCount)" :key="`slideList-${num+1}`">
          <div v-if="num+1 === 2" :class="`slide-box__slide slide-box__slide--${num+1} slide-box__slide--next`" :data-slide-num="num+1"></div>
          <div v-else :class="`slide-box__slide slide-box__slide--${num+1}`" :data-slide-num="num+1"></div>
        </template>

        <!--   애니매이션을 위한 우측 숨겨진 1개   -->
        <div :class="`slide-box__slide slide-box__slide--${divideCount+2}`" :data-slide-num="divideCount+2"></div>
      </template>

      <!--   중앙 하이라이트가 없을 경우 순서대로 나열   -->
      <template v-else>
        <!--   애니매이션을 위한 좌측 숨겨진 1개   -->
        <div :class="`slide-box__slide slide-box__slide--${amount} slide-box__slide--now`" :data-slide-num="amount"></div>

        <!--   전체 요소들   -->
        <div v-for="num in amount" :class="`slide-box__slide slide-box__slide--${num} slide-box__slide--now`" :data-slide-num="num" :key="`slideList-${num}`"></div>

        <!--   애니매이션을 위한 우측 숨겨진 1개   -->
        <div :class="`slide-box__slide slide-box__slide--1 slide-box__slide--now`" :data-slide-num="1"></div>
      </template>
    </div>
  </div>
  <button class="slide-box__slide-btn--allow slide-box__slide-btn--prev" :data-first="isFirst" :data-now="prev_num" @click="clickPrev"></button>
  <button class="slide-box__slide-btn--allow slide-box__slide-btn--next" :data-last="isLast" :data-now="next_num" @click="clickNext"></button>
</template>

<script>
let throttling_slide = false; // 슬라이드 쓰로틀링

export default {
  name: "InfinitySlide",
  emits:['nextStartEmit','prevStartEmit','nextEndEmit','prevEndEmit'],
  props:{
    slideId : { // 고유 id
      type : String,
      required : true,
    },
    amount : { // 슬라이드 수
      type : Number,
      required : true,
    },
    width : { //슬라이드 요소의 width
      type : Number,
      required : true,
    },
    height : { //슬라이드 요소의 height
      type : Number,
      required : true,
    },
    widthSmall : { //슬라이드 요소의 width (작은 요소)
      type : Number,
    },
    heightSmall : { //슬라이드 요소의 height (작은 요소)
      type : Number,
    },
    margin : { //슬라이드 요소의 좌, 우 margin(디자인 시안에서 1/2값)
      type : Number,
      required : true,
    },
    transition : { //슬라이드 트랜지션
      type : Number,
      required : true,
    },
    staticPath : { //리소스의 절대 경로
      type : String,
    },
    imgPatternName : { //이미지 패턴 명 (item-1, item-2에서 item을 의미)
      type : String
    },
    highlight : { //슬라이드 중앙 하이라이트 유무(홀수일 경우에만 작동)
      type : Boolean,
      default : true
    },
  },
  data() {
    return {
      prev_num : 1,
      next_num : this.amount,
      isFirst : 'true',
      isLast : 'false',
      defaultWidth : this.width,
      defaultHeight : this.height,
    }
  },
  computed: {
    css_var() {
      return {
        "--view" : this.amount,
        "--width" : `${this.width}px`,
        "--height" : `${this.height}px`,
        "--widthSmall" : `${this.isHighlight? this.computedSmallWidth : this.width}px`,
        "--heightSmall" : `${this.isHighlight? this.computedSmallHeight : this.height}px`,
        "--margin" : `${this.margin}px`,
        "--transition" : `${this.transition}s`,
      }
    },
    computedSmallWidth(){
      return this.widthSmall || this.defaultWidth;
    },
    computedSmallHeight(){
      return this.heightSmall || this.defaultHeight;
    },
    divideCount(){
      return Math.floor(this.amount/2);
    },
    isHighlight(){
      //amount가 짝수면 highlight 기능 off.
      return this.amount%2===0? false : this.highlight;
    }
  },
  mounted() {
    let slideNodeList = document.querySelectorAll(`${this.slideId} .slide-box__slide`);
    [...slideNodeList].forEach(ele=>{
      ele.style.backgroundImage = `url('${process.env.VUE_APP_PUBLIC_PATH}${this.staticPath}/${this.imgPatternName}-${ele.dataset.slideNum}.png')`;
      ele.style.backgroundSize = 'cover';
    });
  },
  methods: {
    async clickSlideContents(event){
      // highlight가 false일 경우 의미가 없으므로 동작 수행 없이 return.
      if(this.isHighlight === false) return;
      // 클릭 시 작동
      let slideNum = event.target.dataset.slideNum;
      if(slideNum){
        let moveCount = 0;
        let centerNumber = this.divideCount;
        let slideNodeList = [...document.querySelectorAll(`${this.slideId} .slide-box__slide`)]
        slideNodeList.shift();
        slideNodeList.pop();
        moveCount = slideNodeList.indexOf(event.target);

        if(moveCount < centerNumber){
          while (moveCount !== centerNumber) {
            await this.clickPrev('',Boolean(moveCount !== centerNumber-1));
            moveCount++;
          }
        }else if(moveCount > centerNumber){
          while (moveCount !== centerNumber) {
            await this.clickNext('',Boolean(moveCount !== centerNumber+1));
            moveCount--;
          }
        }
      }
    },
    slideThrottling(type,dontNeedEmit) {
      // 쓰로틀링
      if (throttling_slide) return false;

      throttling_slide = true;
      setTimeout(() => {
        let slideNodeNow = document.querySelector(`${this.slideId} .slide-box__slide--now`);
        let slideNodeNext = document.querySelector(`${this.slideId} .slide-box__slide--next`);
        let slideNodePrev = document.querySelector(`${this.slideId} .slide-box__slide--prev`);
        if(type==='next'){
          dontNeedEmit || this.$emit('nextEndEmit',{
            now:slideNodeNow.dataset.slideNum,
            next: slideNodeNext && slideNodeNext.dataset.slideNum,
            prev: slideNodePrev && slideNodePrev.dataset.slideNum,
          });
        }else{
          dontNeedEmit || this.$emit('prevEndEmit',{
            now:slideNodeNow.dataset.slideNum,
            next:slideNodeNext && slideNodeNext.dataset.slideNum,
            prev:slideNodePrev && slideNodePrev.dataset.slideNum,
          });
        }
        throttling_slide = false;
      }, this.transition*1000); //prop 받은 트랜지션의 시간 간격을 두고 버튼 실행
      return true;
    },
    clickPrev(e,dontNeedEmit) {
      // 슬라이드 쓰로틀링 실행
      if (!this.slideThrottling('prev',dontNeedEmit)) return;

      //슬라이드 요소
      let slideNode = document.querySelector(`${this.slideId} .slide-box__slide-box`);
      let slideNodeList = document.querySelectorAll(`${this.slideId} .slide-box__slide`);
      let slideNodeNow = document.querySelector(`${this.slideId} .slide-box__slide--now`);
      let slideNodePrev = document.querySelector(`${this.slideId} .slide-box__slide--prev`);
      let slideNodeNext = document.querySelector(`${this.slideId} .slide-box__slide--next`);

      //버튼 dataset에 바인딩 된 변수 변경
      this.prev_num = this.prev_num - 1;
      this.next_num = this.next_num - 1;

      if(this.prev_num === 1) this.isFirst = 'true';
      else this.isFirst = 'false';
      if(this.next_num == this.amount+1) this.isLast = 'true';
      else this.isLast = 'false';

      //slide 이동
      slideNode.insertBefore(slideNodeList[this.amount+1], slideNode.firstChild);

      //highlight가 true일때 view 슬라이드(now, prev, next) 처리
      if(this.isHighlight){
        slideNodeNow.previousElementSibling.classList.add('slide-box__slide--now');
        slideNodeNow.classList.remove('slide-box__slide--now');
        slideNodeNext.previousElementSibling.classList.add('slide-box__slide--next');
        slideNodeNext.classList.remove('slide-box__slide--next');
        slideNodePrev.previousElementSibling.classList.add('slide-box__slide--prev');
        slideNodePrev.classList.remove('slide-box__slide--prev');
      }

      //좌측 숨겨진 요소 변경
      let firstSlideNum = slideNode.firstChild.nextElementSibling.dataset.slideNum-1;
      firstSlideNum = firstSlideNum === 0? this.amount : firstSlideNum;
      slideNode.firstChild.dataset.slideNum = firstSlideNum;
      slideNode.firstChild.classList = `slide-box__slide slide-box__slide--${firstSlideNum} ${!this.isHighlight? 'slide-box__slide--now':''}`;
      slideNode.firstChild.style.backgroundImage = `url('${process.env.VUE_APP_PUBLIC_PATH}${this.staticPath}/${this.imgPatternName}-${firstSlideNum}.png')`;

      //start emit 발동
      dontNeedEmit || this.$emit('prevStartEmit',{
        now:slideNodeNow.previousElementSibling?.dataset?.slideNum,
        next: slideNodeNext && slideNodeNext.previousElementSibling.dataset.slideNum,
        prev: slideNodePrev && slideNodePrev.previousElementSibling.dataset.slideNum,
      });

      return new Promise(resolve => setTimeout(resolve, this.transition*1000));
    },
    clickNext(e,dontNeedEmit) {
      console.log('?')
      // 슬라이드 쓰로틀링 실행
      if (!this.slideThrottling('next',dontNeedEmit)) return;

      //슬라이드 요소
      let slideNode = document.querySelector(`${this.slideId} .slide-box__slide-box`);
      let slideNodeList = document.querySelectorAll(`${this.slideId} .slide-box__slide`);
      let slideNodeNow = document.querySelector(`${this.slideId} .slide-box__slide--now`);
      let slideNodePrev = document.querySelector(`${this.slideId} .slide-box__slide--prev`);
      let slideNodeNext = document.querySelector(`${this.slideId} .slide-box__slide--next`);

      //버튼 dataset에 바인딩 된 변수 변경
      this.prev_num = this.prev_num + 1;
      this.next_num = this.next_num + 1;

      if(this.prev_num === 1) this.isFirst = 'true';
      else this.isFirst = 'false';
      if(this.next_num == this.amount-1) this.isLast = 'true';
      else this.isLast = 'false';

      //slide 이동
      slideNode.insertBefore(slideNodeList[0], slideNode.lastChild.nextSibling);

      //view 슬라이드(now, prev, next) 처리
      if(this.isHighlight){
        slideNodeNow.nextElementSibling.classList.add('slide-box__slide--now');
        slideNodeNow.classList.remove('slide-box__slide--now');
        slideNodePrev.classList.remove('slide-box__slide--prev');
        slideNodeNext.classList.remove('slide-box__slide--next');
      }

      //총 갯수와 보여질 갯수가 같을 경우 우측 숨겨진 요소 변경
      let lastSlideNum = slideNode.lastChild.previousElementSibling.dataset.slideNum*1+1;
      lastSlideNum = lastSlideNum === this.amount+1? 1 : lastSlideNum;
      slideNode.lastChild.dataset.slideNum = lastSlideNum;
      slideNode.lastChild.classList = `slide-box__slide slide-box__slide--${lastSlideNum} ${!this.isHighlight? 'slide-box__slide--now':''}`;
      slideNode.lastChild.style.backgroundImage = `url('${process.env.VUE_APP_PUBLIC_PATH}${this.staticPath}/${this.imgPatternName}-${lastSlideNum}.png')`;

      //start emit 발동
      dontNeedEmit || this.$emit('nextStartEmit',{
        now:slideNodeNow.nextElementSibling?.dataset?.slideNum,
        next:slideNodeNext && slideNodeNext.nextElementSibling.dataset.slideNum,
        prev:slideNodePrev && slideNodePrev.nextElementSibling.dataset.slideNum,
      });

      //next 시 z-index에 따른 자연스러운 애니매이션을 위한 트랜지션
      setTimeout(()=>{
        if(this.isHighlight) {
          slideNodeNext.nextElementSibling.classList.add('slide-box__slide--next');
          slideNodePrev.nextElementSibling.classList.add('slide-box__slide--prev');
        }
      },200);

      return new Promise(resolve => setTimeout(resolve, this.transition*1000));
    },
  }
}
</script>

<style lang="scss">
$width : var(--width);
$height : var(--height);
$widthSmall : var(--widthSmall);
$heightSmall : var(--heightSmall);
$margin : var(--margin);
$view : var(--view);
$transition : var(--transition);
.slide-box{
  position: absolute;
  left: 0;
  top: 0;
  overflow: hidden;
  &__slide{
    &:nth-child(n){
      position: relative;
      float: left;
      transition: $transition;
      width: $widthSmall;
      height: $heightSmall;
      margin: calc($height/6) $margin 0 $margin;
      filter: brightness(0.1);
      cursor: pointer;
      &:before{
        position: absolute;
        width: $widthSmall;
        height: $heightSmall;
        top:0;
        left: 0;
        content: '';
        transition: $transition;
        opacity: 0;
      }
    }

    &:first-child{
      margin-left: calc($width * -1); // 화면에서 사라질 정도의 margin
    }

    &--now{
      width: $width !important;
      height: $height !important;
      margin-top: 0 !important;
      filter: brightness(1) !important;
      z-index: 10;
      cursor: default !important;
      &:before{
        position: absolute;
        width: $width !important;
        height: $height !important;
        top:0;
        left: 0;
        content: '';
        transition: .9s;
        opacity: 1  !important;
      }
    }
    &--prev{
      z-index: 9;
    }
    &--next{
      z-index: 8;
    }

    &-wrap{
      position: absolute;
      width: calc(($widthSmall * ($view)) + ($width - $widthSmall) + ($margin * $view * 2) + ($margin *2)); // 실제로 눈에 보여질 크기의 width 계산. (각 요소의 너비(margin 포함)*보여질 요소갯수)
      height: $height;
      left : 0;
      top : 50%;
      overflow: hidden;
      z-index: 5;
    }
    &-box{
      width: calc(($widthSmall * $view) + ($width * 2.1)); // 전체 슬라이드가 들어갈 정도의 넉넉한 크기
      height: 100%;
    }
    &-btn{
      &--next{
        position: absolute;
        right : 0;
        top : 50%;

        &[data-last='true'] {
          //background-color: darkgreen;
        }
      }
      &--prev{
        position: absolute;
        left : 0;
        top : 50%;

        &[data-first='true'] {
          //background-color: darkgreen;
        }
      }
      &--allow{
        cursor: pointer;
        z-index: 11;
        width: 46px;
        height: 72px;
      }
    }
  }
}
</style>
