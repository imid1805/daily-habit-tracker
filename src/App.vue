<template>
  <div id="app">
    <div class="phone-container">
      <div class="status-bar">
        <span>12:00</span>
        <div class="status-icons">
          <span>▼</span>
        </div>
      </div>

      <header>
        <h1>Habit Tracker</h1>
        <p class="subtitle">Reach your goals</p>
      </header>

      <div class="date-section">
        <h2>Today</h2>
        <div class="calendar">
          <div class="days-row">
            <div v-for="day in ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']" :key="day" class="day-label">
              {{ day }}
            </div>
          </div>
          <div class="dates-row">
            <div v-for="date in dates" :key="date" 
                 :class="{ 'current-date': date === currentDate }" class="date">
              {{ date }}
            </div>
          </div>
        </div>

        <div class="tabs">
          <div 
            class="tab" 
            :class="{ active: activeTab === 'habits' }"
            @click="activeTab = 'habits'"
          >
            Habits
          </div>
          <div 
            class="tab" 
            :class="{ active: activeTab === 'todos' }"
            @click="activeTab = 'todos'"
          >
            To do {{ todoCompletionText }}
          </div>
        </div>

        <div class="progress-bar" v-if="activeTab === 'habits'">
          <div class="progress" :style="{ width: completionPercentage + '%' }"></div>
          <span class="progress-text">{{ completionPercentage }}% achieved</span>
        </div>

        <div v-if="activeTab === 'habits'">
          <div v-if="habits.length === 0" class="empty-state">
            No habits yet. Add your first habit!
          </div>
          <ul v-else class="habits-list">
            <li v-for="habit in habits" :key="habit.id" class="habit-item">
              <div class="habit-content">
                <i :class="habit.icon" :style="{ color: habit.color }"></i>
                <span>{{ habit.text }}</span>
              </div>
              <div 
                class="checkbox" 
                :class="{ completed: habit.completed }"
                :style="{ borderColor: habit.color }"
                @click="toggleHabit(habit)"
              >
                <i class="mdi mdi-check" v-if="habit.completed"></i>
              </div>
            </li>
          </ul>

          <div class="add-habit-form" v-if="isAddingHabit">
            <input 
              v-model="newHabitText" 
              @keyup.enter="submitNewHabit"
              placeholder="Enter new habit"
              ref="habitInput"
            >
            <div class="form-buttons">
              <button @click="cancelAddHabit" class="cancel-button">Cancel</button>
              <button @click="submitNewHabit" class="submit-button">Add</button>
            </div>
          </div>

          <button v-else class="add-habit" @click="startAddHabit">
            + Add new habit
          </button>
        </div>

        <div v-else>
          <div v-if="todos.length === 0" class="empty-state">
            No tasks yet. Add your first task!
          </div>
          <ul v-else class="habits-list">
            <li v-for="todo in todos" :key="todo.id" class="habit-item">
              <div class="habit-content">
                <i class="mdi mdi-checkbox-blank-outline" :class="{ 'mdi-checkbox-marked': todo.completed }"></i>
                <span>{{ todo.text }}</span>
              </div>
              <div 
                class="checkbox"
                :class="{ completed: todo.completed }"
                @click="toggleTodo(todo)"
              >
                <i class="mdi mdi-check" v-if="todo.completed"></i>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      activeTab: 'habits',
      isAddingHabit: false,
      newHabitText: '',
      habits: [
        { id: 1, text: 'Clean my home', icon: 'mdi mdi-broom', color: '#4CAF50', completed: false },
        { id: 2, text: 'Drink 8 glasses of water', icon: 'mdi mdi-water', color: '#2196F3', completed: false },
        { id: 3, text: 'Meditate', icon: 'mdi mdi-meditation', color: '#FF9800', completed: false },
        { id: 4, text: '30 minutes of movement', icon: 'mdi mdi-run', color: '#FFC107', completed: false },
        { id: 5, text: 'Screen-free meals', icon: 'mdi mdi-cellphone-off', color: '#E91E63', completed: false },
        { id: 6, text: 'Read before bed', icon: 'mdi mdi-book-open-variant', color: '#4CAF50', completed: false }
      ],
      todos: [
        { id: 1, text: 'Task 1', completed: false },
        { id: 2, text: 'Task 2', completed: true },
        { id: 3, text: 'Task 3', completed: false },
        { id: 4, text: 'Task 4', completed: true }
      ],
      dates: [20, 21, 22, 23, 24, 25, 26],
      currentDate: 20
    }
  },
  methods: {
    toggleHabit(habit) {
      habit.completed = !habit.completed;
      this.saveToLocalStorage();
    },
    toggleTodo(todo) {
      todo.completed = !todo.completed;
      this.saveToLocalStorage();
    },
    startAddHabit() {
      this.isAddingHabit = true;
      this.$nextTick(() => {
        this.$refs.habitInput?.focus();
      });
    },
    cancelAddHabit() {
      this.isAddingHabit = false;
      this.newHabitText = '';
    },
    submitNewHabit() {
      if (this.newHabitText.trim()) {
        const habit = {
          id: Date.now(),
          text: this.newHabitText,
          icon: this.getRandomIcon(),
          color: this.getRandomColor(),
          completed: false
        };
        this.habits.push(habit);
        this.newHabitText = '';
        this.isAddingHabit = false;
        this.saveToLocalStorage();
      }
    },
    getRandomIcon() {
      const icons = ['mdi-star', 'mdi-heart', 'mdi-circle', 'mdi-square', 'mdi-triangle'];
      return 'mdi ' + icons[Math.floor(Math.random() * icons.length)];
    },
    getRandomColor() {
      const colors = ['#4CAF50', '#2196F3', '#FF9800', '#FFC107', '#E91E63', '#9C27B0'];
      return colors[Math.floor(Math.random() * colors.length)];
    },
    saveToLocalStorage() {
      localStorage.setItem('habits', JSON.stringify(this.habits));
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    loadFromLocalStorage() {
      const habits = localStorage.getItem('habits');
      const todos = localStorage.getItem('todos');
      if (habits) this.habits = JSON.parse(habits);
      if (todos) this.todos = JSON.parse(todos);
    }
  },
  computed: {
    completionPercentage() {
      const total = this.habits.length;
      if (total === 0) return 0;
      const completed = this.habits.filter(h => h.completed).length;
      return Math.round((completed / total) * 100);
    },
    todoCompletionText() {
      const completed = this.todos.filter(t => t.completed).length;
      return `${completed}/${this.todos.length}`;
    }
  },
  mounted() {
    this.loadFromLocalStorage();
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Pacifico&display=swap');
@import '@mdi/font/css/materialdesignicons.css';

:root {
  --primary-color: #9C27B0;
  --background-color: #F3E5F5;
  --card-background: #FFFFFF;
  --text-color: #333333;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: var(--background-color);
  font-family: 'Segoe UI', sans-serif;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.phone-container {
  max-width: 400px;
  width: 100%;
  background: var(--card-background);
  border-radius: 30px;
  padding: 20px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.status-bar {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  font-size: 14px;
  color: var(--text-color);
}

header {
  text-align: center;
  margin-bottom: 30px;
}

h1 {
  font-family: 'Pacifico', cursive;
  font-size: 2.5rem;
  color: var(--primary-color);
  margin-bottom: 5px;
}

.subtitle {
  font-size: 1.1rem;
  color: #666;
}

.date-section {
  background: var(--card-background);
  border-radius: 20px;
  padding: 20px;
}

h2 {
  font-size: 1.5rem;
  margin-bottom: 15px;
  font-family: 'Pacifico', cursive;
}

.calendar {
  margin-bottom: 20px;
}

.days-row, .dates-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.day-label, .date {
  width: 40px;
  text-align: center;
  font-size: 0.9rem;
}

.current-date {
  background: var(--primary-color);
  color: white;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
}

.tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  border-bottom: 2px solid #eee;
  padding-bottom: 10px;
}

.tab {
  padding: 8px 16px;
  border: none;
  background: none;
  font-size: 1rem;
  color: #666;
  cursor: pointer;
}

.tab.active {
  color: var(--primary-color);
  font-weight: bold;
  position: relative;
}

.tab.active::after {
  content: '';
  position: absolute;
  bottom: -12px;
  left: 0;
  width: 100%;
  height: 2px;
  background: var(--primary-color);
}

.progress-bar {
  height: 8px;
  background: #E0E0E0;
  border-radius: 4px;
  margin-bottom: 20px;
  position: relative;
}

.progress {
  height: 100%;
  background: #4CAF50;
  border-radius: 4px;
  transition: width 0.3s ease;
}

.progress-text {
  position: absolute;
  right: 0;
  top: -20px;
  font-size: 0.8rem;
  color: #4CAF50;
}

.habits-list {
  list-style: none;
  margin-bottom: 20px;
}

.habit-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #eee;
}

.habit-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.habit-content i {
  font-size: 1.5rem;
}

.checkbox {
  width: 24px;
  height: 24px;
  border: 2px solid;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.checkbox.completed {
  background-color: #4CAF50;
  border-color: #4CAF50 !important;
}

.checkbox.completed i {
  color: white;
}

.empty-state {
  text-align: center;
  color: #666;
  padding: 20px;
}

.add-habit-form {
  margin-top: 20px;
}

.add-habit-form input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  margin-bottom: 10px;
}

.form-buttons {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.cancel-button, .submit-button {
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
}

.cancel-button {
  background: #f5f5f5;
  color: #666;
}

.submit-button {
  background: var(--primary-color);
  color: white;
}

.add-habit {
  width: 100%;
  padding: 12px;
  border: none;
  background: #E8F5E9;
  color: #4CAF50;
  border-radius: 10px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s ease;
}

.add-habit:hover {
  background: #C8E6C9;
}

@media (max-width: 480px) {
  .phone-container {
    padding: 15px;
  }

  h1 {
    font-size: 2rem;
  }

  .day-label, .date {
    width: 30px;
    font-size: 0.8rem;
  }
}
</style>
