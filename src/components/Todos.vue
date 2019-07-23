<template>
    <div>
        <h1 class="title">My Todos</h1>
        <h1 class="email">{{userEmail}}</h1>
        <section class="todoapp">
            <div v-if="loading">
                <h1>Loading... just a sec</h1>
            </div>
            <div v-else>
                <header class="header">
                    <input class="new-todo" autofocus autocomplete="off"
                    :placeholder="this.inputPlaceholder" v-model="newTodo" @keyup.enter="addTodo">
                </header>
                <section class="main" v-show="todos.length" v-cloak>
                    <input class="toggle-all" type="checkbox" v-model="allDone">
                    <ul class="todo-list">
                        <li v-for="todo in filteredTodos" class="todo" :key="todo.id"
                            :class="{ completed: todo.completed, editing: todo == editedTodo }">
                            <div class="view">
                                <input class="toggle" type="checkbox" v-model="todo.completed"
                                @change="completeTodo(todo)">
                                <label @dblclick="editTodo(todo)">{{todo.title}}</label>
                                <button class="destroy" @click="removeTodo(todo)"></button>
                            </div>
                            <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo == editedTodo"
                            @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)"
                            @keyup.esc="cancelEdit(todo)">
                        </li>
                    </ul>
                </section>
                <footer class="footer" v-show="todos.length" v-cloak>
                    <span class="todo-count">
                        <strong>{{remaining}}</strong> {{remaining | pluralize}} left
                    </span>
                    <ul class="filters">
                        <li><a href="#/all" @click="setVisibility('all')" :class="{ selected: visibility == 'all' }">All</a></li>
                        <li><a href="#/active" @click="setVisibility('active')" :class="{ selected: visibility == 'active' }">Active</a></li>
                        <li><a href="#/completed" @click="setVisibility('completed')" :class="{ selected: visibility == 'completed' }">Completed</a></li>
                    </ul>
                    <button class="clear-completed" @click="removeCompleted"
                            v-show="todos.length > remaining">
                        Clear completed
                    </button>
                </footer>
            </div>
        </section>
        <div v-if="error" class="error" @click="handleErrorClick">
            ERROR: {{this.error}}
        </div>
    </div>
</template>

<script>
    import api from '../Api.js';

    let filters = {
        all: function (todos) {
            return todos
        },
        active: function (todos) {
            return todos.filter(function (todo) {
                return !todo.completed
            })
        },
        completed: function (todos) {
            return todos.filter(function (todo) {
                return todo.completed
            })
        }
    };
    const Todos = {
        name: 'Todos',
        props: {
            activeUser: Object
        },
        data: function() {
            return {
                todos: [],
                newTodo: '',
                editedTodo: null,
                visibility: 'all',
                loading: true,
                error: null,
            }
        },
        mounted() {
            api.getAll().then(response => {
                this.$log.debug("Data loaded: ", response.data);
                this.todos = response.data;
            })
                .catch(error => {
                    this.$log.debug(error);
                    this.error = "Failed to load todos"
                })
                .finally(() => this.loading = false);
            // this.todos = [{title: 'Drink coffee', completed: false},
            //     {title: 'Finish spring-vue app', completed: false}];

        },

        computed: {
            filteredTodos: function () {
                return filters[this.visibility](this.todos)
            },
            remaining: function () {
                return filters.active(this.todos).length
            },
            allDone: {
                get: function() {
                    return this.remaining === 0
                },
                set: function (value) {
                    return this.todos.forEach(todo => todo.completed = value)
                }
            },
            userEmail: function () {
                return this.activeUser ? this.activeUser.email : ''
            },
            inputPlaceholder: function () {
                return this.activeUser ? this.activeUser.given_name + ', what need to be done today?' :
                    'What needs to be done today?'
            }
        },

        filters: {
            pluralize: function (n) {
                return n === 1 ? 'item' : 'items'
            }
        },

        methods: {
            addTodo() {
                var value = this.newTodo && this.newTodo.trim();
                if(!value) return;

                this.todos.push({
                    title: value,
                    completed: false
                });

                this.newTodo = ''
            },
            setVisibility(vis) {
                this.visibility = vis
            },
            // eslint-disable-next-line no-unused-vars
            completeTodo(todo) {},
            removeTodo(todo) {
                this.todos.splice(this.todos.indexOf(todo), 1)
            },
            editTodo(todo) {
                this.beforeEditCache = todo.title;
                this.editedTodo = todo
            },
            doneEdit(todo) {
                if(!this.editedTodo) return;

                this.editedTodo = null;
                todo.title = todo.title.trim();

                if(!todo.title) this.removeTodo(todo)
            },
            cancelEdit(todo) {
                this.editedTodo = null;
                todo.title = this.beforeEditCache
            },
            removeCompleted() {
                this.todos = filters.active(this.todos)
            },
            handleErrorClick() {
                this.error = null;
            }
        },

        directives: {
            'todo-focus': function (el, binding) {
                if (binding.value) el.focus()
            }
        }
    };

    export default Todos
</script>

<style>
    [v-cloak] { display: none}
</style>