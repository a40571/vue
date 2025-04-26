
<template>
    <div>
        <input
            ref="todoInput"
            v-model="newTodo"
            @keyup.enter="addTodo"
            placeholder="Nhập công việc..."
        />
        <ul>
            <li v-for="(todo, index) in todos" :key="index" class="todo-item">
                <!-- Chỉnh sửa hoặc xem công việc -->
                <div v-if="editIndex === index" class="todo-content full">
          <textarea
              v-model="editText"
              @keyup.enter="saveEdit(index)"
              @blur="saveEdit(index)"
              @keyup.esc="cancelEdit"
              class="edit-textarea"
              rows="2"
          ></textarea>
                </div>

                <div v-else class="todo-content">
                    <span :class="{ completed: todo.completed }">{{ todo.text }}</span>
                </div>
                <div class="checkbox-container">
                    <input
                        type="checkbox"
                        v-model="todo.showNotification"
                        id="checkbox-{{ index }}"
                    />
                    <label for="checkbox-{{ index }}">Nhận thông báo khi hoàn thành</label>
                </div>
                <!-- Tiến độ công việc -->
                <div class="progress-container">
                    <label :for="'progress-' + index">Tiến độ:</label>
                    <input
                        type="range"
                        v-model="todo.progress"
                        min="0"
                        max="100"
                        step="1"
                        :id="'progress-' + index"
                        class="progress-slider"
                        @change="handleProgressChange(todo)"
                    />
                    <span>{{ todo.progress }}%</span>
                </div>

                <!-- Nút hành động -->
                <div class="actions">
                    <button v-if="editIndex !== index" @click="startEdit(index)">✏️</button>
                    <button @click="removeTodo(index)">❌</button>
                </div>
            </li>
        </ul>

        <!-- Hiển thị phần trăm công việc hoàn thành -->
        <div v-if="todos.length > 0">
            <p>Hoàn thành tổng thể: {{ completedPercentage }}%</p>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue'

const newTodo = ref('')
const todos = ref([])
const editIndex = ref(null)
const editText = ref('')
const todoInput = ref(null) // Chỉ khai báo một lần duy nhất!

onMounted(() => {
    const saved = localStorage.getItem('todos')
    if (saved) {
        todos.value = JSON.parse(saved).map((todo) => ({
            ...todo,
            progress: todo.progress ?? 0,
            completed: todo.progress == 100
        }))
    }
})

watch(
    todos,
    (newTodos) => {
        localStorage.setItem('todos', JSON.stringify(newTodos))
    },
    { deep: true }
)

const addTodo = () => {
    if (newTodo.value.trim()) {
        todos.value.push({
            text: newTodo.value.trim(),
            completed: false,
            progress: 0,
            showNotification: false
        })
        newTodo.value = ''
    }
}

const removeTodo = (index) => {
    if (confirm('Bạn có chắc muốn xoá công việc này không?')) {
        todos.value.splice(index, 1)
    }
}

const startEdit = (index) => {
    editIndex.value = index
    editText.value = todos.value[index].text
}

const saveEdit = (index) => {
    const newText = editText.value.trim()
    if (newText) {
        todos.value[index].text = newText
    }
    editIndex.value = null
    editText.value = ''
}

const cancelEdit = () => {
    editIndex.value = null
    editText.value = ''
}

const completedPercentage = computed(() => {
    if (todos.value.length === 0) return 0
    const totalProgress = todos.value.reduce((sum, todo) => sum + Number(todo.progress || 0), 0)
    return Math.round(totalProgress / todos.value.length)
})

const handleProgressChange = (todo) => {
    if (todo.progress === '100' && !todo.completed && todo.showNotification) {
        todo.completed = true
        alert(`🎉 Chúc mừng! Bạn đã hoàn thành công việc: "${todo.text}"`)
    } else if (todo.progress < 100 && todo.completed) {
        todo.completed = false
    }

    const allCompleted = todos.value.length > 0 && todos.value.every(t => t.completed)
    if (allCompleted) {
        if (confirm('✅ Bạn đã hoàn thành tất cả công việc! Bạn có muốn thêm công việc mới không?')) {
            nextTick(() => {
                todoInput.value?.focus()
            })
        }
    }
}
</script>


<style scoped>
input[type='text'] {
    padding: 8px;
    margin-bottom: 12px;
    width: 100%;
    border: 1px solid #ccc;
    border-radius: 5px;
}

ul {
    list-style: none;
    padding: 0;
}

.todo-item {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin: 10px 0;
    padding: 8px;
    border: 1px solid #eee;
    border-radius: 5px;
    background-color: #fff;
}

.todo-content {
    flex: 1;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    line-height: 1.4;
    max-width: 100%;
}

.todo-content.full {
    white-space: normal;
    overflow: visible;
    padding: 0;
}

textarea.edit-textarea {
    width: 100%;
    font-size: 1rem;
    padding: 6px;
    resize: none;
    border-radius: 4px;
    border: 1px solid #ccc;
}

.actions {
    display: flex;
    gap: 8px;
}

button {
    background-color: #ff5252;
    color: white;
    border: none;
    padding: 4px 8px;
    cursor: pointer;
    border-radius: 4px;
    font-size: 14px;
    transition: background-color 0.3s ease;
}

button:hover {
    background-color: #e04848;
}

.completed {
    text-decoration: line-through;
    color: gray;
    word-break: break-word;
}

.progress-container {
    display: flex;
    align-items: center;
    gap: 8px;
}

.progress-slider {
    width: 150px;
    height: 6px;
    border-radius: 5px;
    background-color: #ddd;
    -webkit-appearance: none;
    appearance: none;
}

.progress-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background-color: #4caf50;
    cursor: pointer;
}

.progress-slider::-moz-range-thumb {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background-color: #4caf50;
    cursor: pointer;
}
</style>
