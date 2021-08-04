
<template>
	<div>
		<h3>动态渲染 .vue 文件的组件—— Display</h3>
		<textarea v-model="code"></textarea>
		<dis :code="code"></dis>
		<!-- <p>{{code}}</p> -->
		<!-- <dis v-model="code"></dis> -->
		<!-- <dis>{{code}}</dis> -->
	</div>
</template>
<script>
import dis from "./components/display/display.vue";
import defaultCode from "./components/display/code.js";
export default {
	data() {
		return {
			code:defaultCode
		};
	},
	components: {
		dis,
	},
};
</script>