<template>
    <h1 id="main">ToDo App </h1>
    <form @submit.prevent="">
        <label>Login or Create </label>
        <div>
            <div style="float: left; width: 68%">
                <input
                        autocomplete="off"
                        id="input-id"
                        name="newTodo"
                        style="width: 80%"
                >
            </div>
            <div style="float: right; width: 30%; margin-right: 2%">
                <div style="float: left; width: 50%; margin-right: 10%">
                    <button v-on:click="createButton()">Create</button>
                </div>
                <div style="float: right; width: 40%">
                    <button v-on:click="loginButton()">Login</button>
                </div>
            </div>
        </div>
        <h3 hidden="true" id="text-error" style="text-align: center; color: rgba(188,16,0,0.91)"></h3>
    </form>


    <form @submit.prevent="addTodo()">
        <label>New ToDo </label>
        <input
                autocomplete="off"
                disabled="true"
                id="input-to-do"
                maxlength="40"
                name="newTodo"
                v-model="newTodo"
        >
        <button disabled="true" id="button-to-do">Add ToDo</button>
    </form>
    <h2>ToDo List</h2>
    <ul>
        <li
                :key="index"
                v-for="(todo, index) in todos">
            <span>{{ todo.text }}</span>
            <button @click="removeTodo(index, todo)">Remove</button>
        </li>
    </ul>
    <h4 v-if="todos.length === 0">Empty list.</h4>
</template>

<script>
	import axios from "axios"
	import {ref} from 'vue';

	localStorage.setItem('todos', JSON.stringify([]));
    const newTodo = ref('');
    const todosData = JSON.parse(localStorage.getItem('todos'));
    const todos = ref(todosData);

    export default {
        name: 'App',
        methods: {
            createButton: function () {
                axios.post("http://localhost:3000/create?username=" + document.getElementById('input-id').value)
                    .catch(() => {
                        document.getElementById('text-error').hidden = false;
                        document.getElementById('text-error').textContent = "Such username is already registered!";
                        document.getElementById('main').textContent = "ToDo App";
                        document.getElementById('button-to-do').disabled = true;
                        document.getElementById('input-to-do').disabled = true;
                    })
                    .then((response) => {
                        if (response.status === 200) {
                            document.getElementById('button-to-do').disabled = false;
                            document.getElementById('input-to-do').disabled = false;
                            document.getElementById('main').textContent = "ToDo App (Signed as " + document.getElementById('input-id').value + ")";
                            document.getElementById('text-error').hidden = true;

                            todos.value.splice(0, todos.value.length);
                        }
                    });
            },
            loginButton: function () {
                axios.post("http://localhost:3000/login?username=" + document.getElementById('input-id').value)
                    .catch(() => {
                        document.getElementById('text-error').hidden = false;
                        document.getElementById('text-error').textContent = "There is no such username!";
                        document.getElementById('main').textContent = "ToDo App";
                        document.getElementById('button-to-do').disabled = true;
                        document.getElementById('input-to-do').disabled = true;
                    })
                    .then((response) => {
                        if (response.status === 200) {
                            document.getElementById('button-to-do').disabled = false;
                            document.getElementById('input-to-do').disabled = false;
                            document.getElementById('main').textContent = "ToDo App (Signed as " + document.getElementById('input-id').value + ")";
                            document.getElementById('text-error').hidden = true;

                            axios.get("http://localhost:3000/todos?username=" + document.getElementById('input-id').value)
                                .then(response => {
                                    localStorage.setItem('todos', JSON.stringify([]));
                                    todos.value.splice(0, todos.value.length);
                                    response.data.forEach(todo => {
                                        todos.value.push({
                                            _id: todo._id,
                                            text: todo.text
                                        });
                                    })
                                })
                        }
                    });
            },
            show: function () {
                return localStorage.getItem('enable')
            }
        },
        setup() {
            function addTodo() {
                axios.post("http://localhost:3000/todo?username=" + document.getElementById('input-id').value + "&text=" + newTodo.value)
                    .then(response => {
                        console.log("ADD ID: " + response.data.id);
                        if (newTodo.value) {
                            todos.value.push({
                                _id: response.data.id,
                                text: response.data.text
                            });
                            newTodo.value = '';
                        }
                    });
                saveData();
            }

            function removeTodo(index, todo) {
                axios.delete("http://localhost:3000/todo?todoId=" + todo._id);
                todos.value.splice(index, 1);
                saveData();
            }

            function saveData() {
                const storageData = JSON.stringify(todos.value);
                localStorage.setItem('todos', storageData);
            }

            return {
                todos,
                newTodo,
                addTodo,
                removeTodo,
                saveData
            }
        }
    }
</script>

<style lang="scss">
    $border: 2px solid rgba(
    $color: white,
    $alpha: 0.35,
    );
    $size1: 6px;
    $size2: 12px;
    $size3: 18px;
    $size4: 24px;
    $size5: 48px;
    $backgroundColor: #27292d;
    $textColor: white;
    $primaryColor: #a0a4d9;
    $secondTextColor: #1f2023;
    body {
        margin: 0;
        padding: 0;
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        background-color: $backgroundColor;
        color: $textColor;

        #app {
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            padding: 20px;

            h1 {
                font-weight: bold;
                font-size: 28px;
                text-align: center;
            }

            form {
                display: flex;
                flex-direction: column;
                width: 100%;

                label {
                    font-size: 14px;
                    font-weight: bold;
                }

                input,
                button {
                    height: $size5;
                    box-shadow: none;
                    outline: none;
                    padding-left: $size2;
                    padding-right: $size2;
                    border-radius: $size1;
                    font-size: 18px;
                    margin-top: $size1;
                    margin-bottom: $size2;
                }

                button:disabled {
                    height: $size5;
                    box-shadow: none;
                    outline: none;
                    padding-left: $size2;
                    padding-right: $size2;
                    border-radius: $size1;
                    font-size: 18px;
                    margin-top: $size1;
                    margin-bottom: $size2;
                    background-color: rgba(182, 187, 252, 0.21);
                }

                input {
                    background-color: transparent;
                    border: $border;
                    color: inherit;
                }
            }

            button {
                cursor: pointer;
                background-color: $primaryColor;
                border: 1px solid $primaryColor;
                color: $secondTextColor;
                font-weight: bold;
                outline: none;
                border-radius: $size1;
            }

            h2 {
                font-size: 22px;
                border-bottom: $border;
                padding-bottom: $size1;
            }

            ul {
                padding: 10px;

                li {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    border: $border;
                    padding: $size2 $size4;
                    border-radius: $size1;
                    margin-bottom: $size2;

                    span {
                        cursor: pointer;
                    }

                    .done {
                        text-decoration: line-through;
                    }

                    button {
                        font-size: $size2;
                        padding: $size1;
                    }
                }
            }

            h4 {
                text-align: center;
                opacity: 0.5;
                margin: 0;
            }
        }
    }
</style>
