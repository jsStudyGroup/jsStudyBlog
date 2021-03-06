---
bg: "vue.jpg"
layout: post
title:  "Vue.js Instance"
crawlertitle: "vue.js Instance"
summary: "Vue.js Instance Life cycle"
date:   2018-02-05 14:00 +0900
categories: posts
tags: 'Vue.js'
author: codeMonkey
---

### Vue.js - Instance ###

**Vue 포스트는 node, npm, git, vue-cli, webpack을 알고 있으며, 설치 했다는 가정하에 설명 합니다. 참고 서적은 [Do it! Vue.js 입문](http://www.yes24.com/24/Goods/58206961?Acode=101&) 입니다.**

---


#### VUE 인스턴스 라이프 사이클 ####
- Vue 인스턴스의 유효범위는 new Vue로 선택한 el 선택자 안에서가 인스턴스 유효범위 입니다.
	``` html
	<div id="app">
		{{ msg }} // 뷰 인스턴스 유효범위
	</div>

	<script>
		new Vue({
			el: '#app', // 뷰 인스턴스 유효범위 지정
			data: {
				msg: 'hello world!'
			}
		})
	</script>
	```

- Vue 인스턴스 라이프 사이클 
	![Vue.js Instance Life cycle](/jsStudyBlog/assets/images/Vue-instance-lifecycle-Page-1.jpg)

#### Vue 라이프 사이클은 크게 4 가지로 나눌 수 있다. ####

##### 1. Creation 컴포넌트 초기 #####
- 처음 실행되는 부분이며 이 단계는 컴포넌트가 DOM에 추가되기 전
- 서버 렌더링에서도 지원되는 훅
- 클라이언트, 서버 모두에서 처리해야 할 일이 있다면 이 단계에서 하면 된다.
- **beforCreate(){ ... }**
	- data와 events(vm.$on, vm.$once, vm.$off, vm.$emit) 세팅 되기 전.
- **created(){ ... }**
	- data와 events가 활성화되어 접근 가능. (this.data, thisfetchData())
	- 템플릿과 가상돔은 마운트 및 렌더링되지 않은 상태 (template 속성 접근 X)
	- data 속성과 methods 속성에 접근할 수 있는 첫 라이프 사이클 단계 
	- 서버 데이터를 요청하여 받아오는 로직을 수행하기 좋음

##### 2. Mounting DOM 삽입 단계 #####
- 초기 렌더링 직전, component에 직접 접근가능 (서버 렌더링 X)
- **beforeMount(){ ... }**
	- render 직전에 실행 된다. (DOM 그리기 전)
- **mounted(){ ... }**
	- el 속성에서 지정한 화면 요소에 인스턴스가 부착되고 나면 호출 되는 단계
	- template 속성에 정의한 화면 DOM에 접근 가능 하지만 모든 컴포넌트가 mounted 것은 보장하지 않음.
	- vm.$nextTick() 사용하면 전체가 render 된 상태를 보장한다.
```javascript
export default {
	mounted() {
		console.log(this.$el.textContent) // can use $el
		this.$nextTick(function () {
			// 모든 화면이 렌더링된 후 실행
		})
	}
}
```

##### 3. Updating #####
- component에서 사용되는 반응형 속성들이 변경되거나 re-render되면 실행된다.
- el 속성에서 지정한 화면 요소에 인스턴스가 부착되고 나면 인스턴스에 정의한 속성이 화면에 치환된다.
- 치환된 값은 Reactivity(반응성) 제공을 위해 $watch 속성으로 감시 (데이터 관찰)
- **beforeUpdate(){ ... }**
	- wtach data가 변경되면 가상 DOM으로 화면을 다시 그리기 전에 호출 되는 단계
	- 변경 예정인 새 데이터에 접근 가능
	- 이 단계에 값을 변경하는 로직을 넣더라도 화면에는 아직 그려지지 않음
- **updated(){ ... }**
	- 데이터 변경된 후 가상 DOM으로 화면을 그리고 난 후
	- 이 단계에서 데이터 값을 변경하면. 무한루프에 빠질 수 있다고 함 (computed, watch 속성을 사용하여 값을 변경가능)
	- 데이터 값을 갱신하는 로직은 가급적 beforeUpdate에 추가, updated에서는 변경 데이터의 DOM 관련된 로직 추가 추천
	- vm.$nextTick() 사용하여 전체가 render 된 상태를 보장

##### 4. Destruction 해체 #####
- **beforeDestroy(){ ... }**
	- 해체 직전에 호출
	- event listener, reactive subscription 제거 시
	- 아직 인스턴스에 접근이 가능하여 인스턴스 데이터 삭제하기 좋음
- **destroyed(){ ... }**
	- 해체 후 호출
	- 모든 bind 해제 되고, 모든 event listener제거 하위 Vue 인스턴스 또한 삭제

#### Vue 라이프 사이클 Demo ####
```javascript
<script>
	new Vue({
		el : '#app',
		data : {
			msg : 'hello world!'
		},
		beforeCreate : function(){
			console.log('beforeCreate!');
		},
		created : function(){
			console.log('created!');
		},
		mounted : function(){
			console.log('mounted!');
			// this.msg = 'update msg!' data의 값이 바뀌면 updated()가 실행된다.
		},
		updated : function(){
			console.log('updated!'); //이 부분은 처음에 실행되지 않음
		}
	})
</script>
```