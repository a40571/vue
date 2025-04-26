<template>
    <div class="container">
        <h1>📝 Todo App</h1>
        <TodoList :onUpdateCompleted="updateCompleted" />
    </div>
</template>

<script setup>
import TodoList from './components/TodoList.vue'

const updateCompleted = (todo) => {
    if (todo.progress === 100 && !todo.completed) {
        todo.completed = true
        alert(`🎉 Chúc mừng! Bạn đã hoàn thành công việc: "${todo.text}"`)
    } else if (todo.progress < 100 && todo.completed) {
        todo.completed = false
    }
}
</script>

<style scoped>
.container {
    max-width: 500px;
    margin: 50px auto;
    padding: 20px;
    border: 1px solid #eee;
    border-radius: 10px;
    background-color: #f9f9f9;
    font-family: sans-serif;
}
</style>
