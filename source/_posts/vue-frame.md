---
title: vue frame
date: 2019-09-15 21:04:13
tags: 
 -vue
categories: 
 -vue官网文档小试
---
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
		<title>vue学习</title>
	<body>
		<div id="app">
			<u>消息显示</u><br><br>
			{{ message }}	
		</div>
		<div id="app2">
			<u>绑定事件</u><br><br>
			<span v-bind:title="message2">
				悬停显示
			</span>
		</div>
		<div id="app3">
			<u>条件判定</u><br>app3.seen = false不可见
			<p v-if="seen">
				now you can seen me 
			</p>
		</div>
		<div id="app4">
			<u>循环</u>app4.todos.push({text:"something"})添加元素
			<ol>
				<li v-for="todo in todos">
					{{todo.text}}
				</li>
			</ol>
		</div>
		<div id="app5">
			<u>事件</u>
			<p>{{message3}}</p>
			<button v-on:click="reverseMessage">Reverse</button>
		</div>
		<div id="app6">
			<br><u>v-model指令，实现表单输入和应用状态之间的双向绑定</u><br/>
			<p>{{message6}}</p>
			<input v-model="message6" />
		</div>
	</body>
	<script>
		var app = new Vue({
			el:"#app",
			data:{
				message:"Hello Vue!ss...."
			}
		});
		var app2 = new Vue({
			el:"#app2",
			data:{
				message2:"hold on ,I will stay~~信息查看于"+new Date().toLocaleString()
				}
		});
		var app3 = new Vue({
			el:"#app3",
			data:{
				seen:"true"
			}
		});
		var app4 = new Vue({
			el:"#app4",
			data:{
				todos : [
					{text:"study js"},
					{text:"study vue"},
					{text:"do something cool"}
				]
			}
		});
		var app5 = new Vue({
			el:"#app5",
			data:{
				message3:"I wants reverse!"
			},
			methods:{
				reverseMessage:function(){
					this.message3 = this.message3.split('').reverse().join('');
				}
			}
		});
		var app6 = new Vue({
			el:"#app6",
			data:{
				message6:"Restful API"
			}
		});
	</script>
</html>
