<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import axios from 'axios'

// Define reactive variables
const todos = ref([])
const name = ref('')
const input_content = ref('')
const input_category = ref(null)

// Computed property to sort todos by createdAt timestamp
const todos_asc = computed(() => todos.value.sort((a, b) => a.createdAt - b.createdAt))

// Watchers to save name and todos to localStorage
watch(name, (newVal) => {
    localStorage.setItem('name', newVal)
})
watch(todos, (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal))
}, { deep: true })

// Fetch todos from the server when the component is mounted
onMounted(() => {
    name.value = localStorage.getItem('name') || ''
    fetchTodos()
})

// Function to fetch todos from the server
const fetchTodos = async () => {
    try {
        const response = await axios.get('http://localhost:3001/todos')
        todos.value = response.data
    } catch (error) {
        console.error('Error fetching todos:', error)
    }
}

// Function to add a new todo
const addTodo = async () => {
    if (input_content.value.trim() === '' || input_category.value === null) {
        return
    }

    const newTodo = {
        content: input_content.value,
        category: input_category.value,
        done: false,
        editable: false,
        createdAt: new Date().getTime()
    }

    try {
        await axios.post('http://localhost:3001/todos', newTodo)
        await fetchTodos() // Refresh todos after adding a new one

        input_content.value = '' // Clear input fields after adding a todo
        input_category.value = null
    } catch (error) {
        console.error('Error adding todo:', error)
    }
}

// Function to remove a todo
const removeTodo = async (todo) => {
    try {
        await axios.delete(`http://localhost:3001/todos/${todo.id}`)
        await fetchTodos() // Refresh todos after deleting
    } catch (error) {
        console.error('Error deleting todo:', error)
    }
}
</script>

<template>
    <main class="app">
        <section class="greeting">
            <h2 class="title">
                Hello User <input type="text" id="name" placeholder="Welcome to our productive space!" v-model="name">
            </h2>
        </section>

        <section class="create-todo">
            <h3>Kickstart Your To-Do List</h3>

            <form id="new-todo-form" @submit.prevent="addTodo">
                <h4>What's on your todo list today?</h4>
                <input 
                    type="text" 
                    name="content" 
                    id="content" 
                    placeholder="For instance, you could add tasks like Write a professional email or Plan a personal project."

                    v-model="input_content" />
                
                <h4>Pick a category</h4>
                <div class="options">
                    <label>
                        <input 
                            type="radio" 
                            name="category" 
                            id="category1" 
                            value="business"
                            v-model="input_category" />
                        <span class="bubble business"></span>
                        <div>Professional</div>
                    </label>

                    <label>
                        <input 
                            type="radio" 
                            name="category" 
                            id="category2" 
                            value="personal"
                            v-model="input_category" />
                        <span class="bubble personal"></span>
                        <div>Personal</div>
                    </label>
                </div>

                <input type="submit" value="Add todo" />
            </form>
        </section>

        <section class="todo-list">
            <h3>TODO LIST</h3>
            <div class="list" id="todo-list">
                <div v-for="todo in todos_asc" :key="todo.id" :class="`todo-item ${todo.done && 'done'}`">
                    <label>
                        <input type="checkbox" v-model="todo.done" />
                        <span :class="`bubble ${todo.category == 'business' ? 'business' : 'personal'}`"></span>
                    </label>

                    <div class="todo-content">
                        <input type="text" v-model="todo.content" />
                        <p>Created At: {{ new Date(todo.createdAt).toLocaleString() }}</p> 
                    </div>

                    <div class="actions">
                        <button class="delete" @click="removeTodo(todo)">Delete</button>
                    </div>
                </div>
            </div>
        </section>
    </main>
</template>


