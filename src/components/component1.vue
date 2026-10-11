<template>
  <div class="todo-app">
    <h1>📋 待办事项清单</h1>
    
    <!-- 输入区域 -->
    <div class="input-section">
      <input
        v-model="newTodoText"
        type="text"
        placeholder="输入新的待办事项..."
        @keyup.enter="addTodo"
        class="todo-input"
      />
      <button @click="addTodo" class="add-btn">添加</button>
    </div>

    <!-- 统计信息 -->
    <div class="stats">
      <span>总计: {{ todos.length }} 项</span>
      <span>已完成: {{ completedCount }} 项</span>
      <span>未完成: {{ activeCount }} 项</span>
    </div>

    <!-- 过滤器 -->
    <div class="filters">
      <button
        v-for="filter in filters"
        :key="filter.value"
        @click="currentFilter = filter.value"
        :class="{ active: currentFilter === filter.value }"
        class="filter-btn"
      >
        {{ filter.label }}
      </button>
    </div>

    <!-- 待办列表 -->
    <ul class="todo-list">
      <li
        v-for="todo in filteredTodos"
        :key="todo.id"
        :class="{ completed: todo.completed }"
        class="todo-item"
      >
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo.id)"
          class="todo-checkbox"
        />
        <span class="todo-text">{{ todo.text }}</span>
        <button @click="removeTodo(todo.id)" class="delete-btn">删除</button>
      </li>
    </ul>

    <!-- 空状态提示 -->
    <div v-if="filteredTodos.length === 0" class="empty-state">
      {{ emptyMessage }}
    </div>

    <!-- 清除已完成按钮 -->
    <button
      v-if="completedCount > 0"
      @click="clearCompleted"
      class="clear-btn"
    >
      清除已完成 ({{ completedCount }})
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

// 定义待办项类型
interface Todo {
  id: number
  text: string
  completed: boolean
}

// 定义过滤器类型
type FilterType = 'all' | 'active' | 'completed'

// 响应式数据
const newTodoText = ref('')
const todos = ref<Todo[]>([])
const currentFilter = ref<FilterType>('all')
let nextId = 1

// 过滤器选项
const filters = [
  { label: '全部', value: 'all' as FilterType },
  { label: '未完成', value: 'active' as FilterType },
  { label: '已完成', value: 'completed' as FilterType }
]

// 计算属性：已完成数量
const completedCount = computed(() => 
  todos.value.filter(todo => todo.completed).length
)

// 计算属性：未完成数量
const activeCount = computed(() => 
  todos.value.filter(todo => !todo.completed).length
)

// 计算属性：根据当前过滤显示待办列表
const filteredTodos = computed(() => {
  switch (currentFilter.value) {
    case 'active':
      return todos.value.filter(todo => !todo.completed)
    case 'completed':
      return todos.value.filter(todo => todo.completed)
    default:
      return todos.value
  }
})

// 计算属性：空状态消息
const emptyMessage = computed(() => {
  switch (currentFilter.value) {
    case 'active':
      return '🎉 没有未完成的任务！'
    case 'completed':
      return '📭 还没有已完成的任务'
    default:
      return '📝 暂无待办事项，添加一个吧！'
  }
})

// 添加新待办
const addTodo = (): void => {
  const text = newTodoText.value.trim()
  if (!text) return

  todos.value.push({
    id: nextId++,
    text,
    completed: false
  })
  newTodoText.value = ''
}

// 切换完成状态
const toggleTodo = (id: number): void => {
  const todo = todos.value.find(todo => todo.id === id)
  if (todo) {
    todo.completed = !todo.completed
  }
}

// 删除待办
const removeTodo = (id: number): void => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}

// 清除所有已完成
const clearCompleted = (): void => {
  todos.value = todos.value.filter(todo => !todo.completed)
}
</script>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: 2rem auto;
  padding: 1.5rem;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', system-ui, sans-serif;
}

h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 1.5rem;
  font-size: 1.8rem;
}

.input-section {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.todo-input {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s;
}

.todo-input:focus {
  outline: none;
  border-color: #42b883;
}

.add-btn {
  padding: 0.75rem 1.5rem;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.add-btn:hover {
  background: #369c6e;
}

.stats {
  display: flex;
  justify-content: space-around;
  padding: 0.75rem;
  background: #f8f9fa;
  border-radius: 8px;
  margin-bottom: 1rem;
  font-size: 0.9rem;
  color: #555;
}

.filters {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
  justify-content: center;
}

.filter-btn {
  padding: 0.4rem 1rem;
  border: 2px solid #e0e0e0;
  background: white;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.filter-btn.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.filter-btn:hover:not(.active) {
  border-color: #42b883;
  color: #42b883;
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 0 0 1rem;
}

.todo-item {
  display: flex;
  align-items: center;
  padding: 0.75rem;
  border-bottom: 1px solid #f0f0f0;
  transition: background 0.2s;
}

.todo-item:hover {
  background: #f8f9fa;
}

.todo-item.completed .todo-text {
  text-decoration: line-through;
  color: #aaa;
}

.todo-checkbox {
  width: 1.2rem;
  height: 1.2rem;
  margin-right: 0.75rem;
  cursor: pointer;
  accent-color: #42b883;
}

.todo-text {
  flex: 1;
  font-size: 1rem;
  color: #333;
}

.delete-btn {
  padding: 0.3rem 0.8rem;
  background: #ff4757;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.8rem;
  opacity: 0;
  transition: opacity 0.2s, background 0.2s;
}

.todo-item:hover .delete-btn {
  opacity: 1;
}

.delete-btn:hover {
  background: #ee3b4a;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #999;
  font-size: 1rem;
}

.clear-btn {
  display: block;
  width: 100%;
  padding: 0.75rem;
  background: #f8f9fa;
  color: #666;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.clear-btn:hover {
  background: #ff4757;
  color: white;
  border-color: #ff4757;
}
</style>