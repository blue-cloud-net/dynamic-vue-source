
<template>
	<div ref="display"></div>
</template>
<script>
import randomStr from "../../utils/random_str.js";
import Vue from "vue/dist/vue";
export default {
	props: {
		code: {
			type: String,
			default: "",
		},
	},
	data() {
		return {
			html: "",
			js: "",
			css: "",
			id: randomStr(),
		};
	},
	mounted() {
		this.renderCode();
	},
	methods: {
		// 分割
		getSource(source, type) {
			const regex = new RegExp(`<${type}[^>]*>`);
			// 标签匹配
			let openingTag = source.match(regex);

			if (!openingTag) {
				return "";
			} else {
				openingTag = openingTag[0];
			}
			//返回的结果剔除了script等标签
			return source.slice(
				source.indexOf(openingTag) + openingTag.length,
				source.lastIndexOf(`</${type}>`)
			);
		},
		//具体化分割
		splitCode() {
			const script = this.getSource(this.code, "script").replace(
				/export default/,
				"return "
			);
			const style = this.getSource(this.code, "style");
			const template =
				'<div id="app">' + this.getSource(this.code, "template") + "</div>";

			this.js = script;
			this.css = style;
			this.html = template;
		},
		// 渲染，挂载
		renderCode() {
			this.splitCode();
			if (this.html !== "" && this.js !== "") {
				const parseStrToFunc = new Function(this.js)();

				parseStrToFunc.template = this.html;
				const Component = Vue.extend(parseStrToFunc);
				this.component = new Component().$mount();
				this.$refs.display.appendChild(this.component.$el);

				if (this.css !== "") {
					const style = document.createElement("style");
					style.type = "text/css";
					style.id = this.id;
					style.innerHTML = this.css;
					document.getElementsByTagName("head")[0].appendChild(style);
				}
			}
		},
		// 当display组件销毁时，也要手动销毁extend创建的实例以及上面的css
		destroyCode() {
			// 销毁style
			const $target = document.getElementById(this.id);
			if ($target) {
				$target.parentNode.removeChild($target);
			}
			if (this.component) {
				this.$refs.display.removeChild(this.component.$el);
				this.component.$destroy();
				this.component = null;
			}
		},
	},
	// 当this.code更新时，整个过程需要重来一次，所以要对code进行监听
	watch: {
		code() {
			this.destroyCode();
			this.renderCode();
		},
	},
	beforeDestroy() {
		this.destroyCode();
	},
};
</script>