<template>
    <div id="app">
        <header class="header">MY TODOS</header>
        <div class="content">
            <span class="icon-down el-icon-arrow-down" 
            v-show="todoLists.length>0" 
            @click="selectAllTodos">
            </span>
            <input type="text" class="todos_add" placeholder="What needs to be done?" 
            @keyup.enter="addTodo($event.target)" 
            ref="currentInput">
            <ul class="content_todoLists">
                <li v-for="(list,index) in todoLists" :key="index" class="content_todoList" 
                @mouseover="list.isActive = true" 
                @mouseleave="list.isActive=false"
                v-show="defaultShow || (whichShow?list.isChecked:!list.isChecked)">
                    <input type="checkbox" class="checkBox" v-model="list.isChecked">
                    <div class="content_todoList_main" @dblclick="toEdit(list)" v-show="!list.isEditing" :class="{deleted:list.isChecked}">
                        {{list.value}}
                    </div>
                    <input type="text" class="content_todoList_main main_input" 
                    v-model="list.value" 
                    v-show="list.isEditing" 
                    v-todo-focus="list.value"
                    @blur="unEdit(list)">
                    <span class="el-icon-close content_todoList_delete" :class="{show: list.isActive}" @click="deleteTodo(index)"></span>
                </li>
            </ul>
            <div class="data" v-show="todoLists.length>0">
                <div class="data_times" v-show="times === 0">
                    <span>{{times}}</span>&nbsp;items left
                </div>
                <div class="data_times" v-show="times > 0">
                    <span>{{times}}</span>&nbsp;items left
                </div>
                <div class="data_status">
                    <a href="#" :class="{active:index === dataStatusIndex}" v-for="(item,index) in dataStatus" @click="switchStatus(index)" :key="index">
                        {{item}}
                    </a>
                </div>
                <div class="data_clearTodos" @click="clearTodos" v-show="times < todoLists.length">
                    <a href="#">clear completed</a>
                </div>
                <div class="data_clearTodos" @click="clearTodos" v-show="times === todoLists.length">
                    <a href="#"></a>
                </div>
            </div>
        </div>
        <footer class="info">
			<p>Double-click to edit a todo</p>
			<p>Written by <a href="http://twitter.com/lukeed05">Lanyiran</a></p>
			<p>Refactored by <a href="https://github.com/xorgy">Aaron Muir Hamilton</a></p>
			<p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
		</footer>
    </div>
</template>

<script>
export default {
    data() {
        return {
            todoLists: [],
            dataStatus: ["All", "Active", "Completed"],
            dataStatusIndex: 0,
            whichShow: true,
            defaultShow: true
        }
    },
    computed: {
        times() { //使用计算属性计算待办todos的次数 

//ES6:  let  (const和let很像,一般用const声明常量)   
//      1.不存在变量提升
//      2.暂时性死区——只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响
//      3.不允许重复声明
//      4.块级作用域——被{}包裹的，外部不能访问内部
//      注意： 
//          1. for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域，所以我们可以在循环体内部访问到i的值。 
//          2. let和var全局声明时，var可以通过window的属性访问而let不能。

            let todoArr = this.todoLists;
            let times = 0;

//ES6:  for...of...  和  for...in... 差不多
            for (let arr of todoArr) {
                if (arr.isChecked === false) {
                    times++;
                }
            }
            return times;
        }
    },
    methods: {
        toEdit(obj) { //使添加的todo可编辑
            obj.isEditing = true
        },
        unEdit(obj) { //使添加的todo不可编辑
            obj.isEditing = false
        },
        addTodo(e) { //添加todo
            var val = e.value
            if (val === "") {
                return
            } //如果输入内容为空则立即返回
            this.todoLists = this.todoLists.concat({ //使用concat这个api添加todo
                value: val, //输入内容
                isEditing: false, //是否在编辑状态
                isActive: false, //删除X图标是否激活
                isChecked: false //是否已完成
            })
            this.$refs.currentInput.value = "" //按下enter添加todo之后把输入框value清零
            window.localStorage.setItem("content", JSON.stringify(this.todoLists)) //使用localStorage以JSON格式存储数据
        },
        deleteTodo(index) { //删除todo
            this.todoLists.splice(index, 1)
            window.localStorage.setItem("content", JSON.stringify(this.todoLists)) //以json格式存储数据
        },
        switchStatus(index) { //试下下方三个状态切换，略麻烦
            this.dataStatusIndex = index
            if (this.dataStatus[index] === "Active") {
                this.defaultShow = false
                this.whichShow = false
            } else if (this.dataStatus[index] === "Completed") {
                this.defaultShow = false
                this.whichShow = true
            } else if (this.dataStatus[index] === "All") {
                this.defaultShow = true
            }
        },
        
//ES6:  => 箭头函数

        clearTodos() { //清空已完成的todoLists
            this.todoLists = this.todoLists.filter(todo => todo.isChecked === false)
            window.localStorage.setItem("content", JSON.stringify(this.todoLists)) //以json格式存储数据
        },
        selectAllTodos() { //设置所有todo为已完成
            this.todoLists.map(todo => todo.isChecked = todo.isChecked ? false : true)
        },
        directives: { //自定义focus指令
            "todo-focus": function (el, binding) {
                if (binding.value) {
                    el.focus()
                }
            }
        },
        created() {
            let myStorage = window.localStorage.getItem('content')
            this.todoLists = JSON.parse(myStorage) || [] //因为todoLists初始值是null,使用或运算符，如果为null设为空数组
        }
    }
}
</script>

<style scoped>
    * {
        padding: 0;
        margin: 0;
        box-sizing: border-box;
        font: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    }

    input {
        outline: none;
    }

    ul,
    li,
    ol {
        list-style: none;
    }

    #app {
        width: 800px;
        height: 900px;
        margin: 0 auto;
        text-align: center;
        background-color: rgb(245, 245, 245);
        padding: 24px 0;
    }

    #app .header {
        top: -155px;
        width: 100%;
        font-size: 100px;
        font-weight: 100;
        text-align: center;
        color: rgba(175, 47, 47, 0.15);
    }

    .content {
        width: 72%;
        margin: 0 auto;
        box-shadow: 0 3px 3px 2px rgba(0, 0, 0, 0.25);
        position: relative;
    }

    .icon-down {
        position: absolute;
        font-size: 24px;
        top: 16px;
        left: 16px;
        cursor: pointer;
    }

    #app .content .todos_add {
        width: 100%;
        height: 56px;
        padding: 16px 56px;
        font-size: 24px;
        border: 1px solid transparent;
    }

    .content_todoLists {
        position: relative;
        z-index: 3;
    }

    .content_todoList {
        display: flex;
        flex-direction: row;
        border-top: 1px solid #ccc;
        font-size: 24px;
        padding: 8px;
        background-color: white;
        align-items: center;
    }

    .checkBox {
        width: 20px;
        height: 20px;
        margin-left: 10px;
    }

    .content_todoList_main {
        flex: 1;
        text-align: left;
        margin-left: 16px;
        font-size: 20px;
        padding: 6px 0;
    }

    .main_input {
        position: relative;
        z-index: 1;
    }

    .content_todoList_delete {
        position: absolute;
        right: 16px;
        color: rgb(252, 55, 55);
        font-weight: 500;
        display: none;
        cursor: pointer;
    }

    .show {
        display: block;
    }

    .deleted {
        text-decoration-line: line-through;
        color: #bbb;
    }

    .show:hover {
        color: rgb(255, 0, 0);
        font-weight: 700;
    }

    ::-moz-placeholder {
        color: rgb(221, 218, 218);
    }

    ::-webkit-input-placeholder {
        color: rgb(221, 218, 218);
    }

    :-ms-input-placeholder {
        color: rgb(221, 218, 218);
    }

    .data {
        display: flex;
        justify-content: space-between;
        padding: 8px;
        font-size: 14px;
        font-weight: 300;
        color: rgb(145, 145, 145);
        box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2),
	            0 8px 0 -3px #f6f6f6,
	            0 9px 1px -3px rgba(0, 0, 0, 0.2),
	            0 16px 0 -6px #f6f6f6,
	            0 17px 2px -6px rgba(0, 0, 0, 0.2);
    }

    a {
        text-decoration: none;
        color: rgb(145, 145, 145);
    }

    .data_times {
        width: 73px;
    }

    .data_clearTodos {
        width: 142px;
    }

    .data_status a {
        display: inline-block;
        border: 1px solid transparent;
        border-radius: 2px;
        padding: 1px 4px;
        margin: 0 2px;
    }

    .data_status a:hover {
        border-color: #bbb;
    }

    .data_clearTodos a:hover {
        text-decoration-line: underline;
    }

    .active {
        box-shadow: 0 0 1px black;
    }
    .info {
        margin: 65px auto 0;
        color: #bfbfbf;
        font-size: 10px;
        text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
        text-align: center;
        line-height: 1.4em;
    }
</style>