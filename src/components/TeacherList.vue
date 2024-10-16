<script setup>
import { ref, computed } from 'vue';

const teachers = ref([
  { id: 1, name: '张老师', subject: '数学', avatar: 'https://placekitten.com/100/100', contact: '123-4567', officeHours: '周一至周五 9:00-17:00' },
  { id: 2, name: '李老师', subject: '语文', avatar: 'https://placekitten.com/101/101', contact: '234-5678', officeHours: '周一至周五 10:00-18:00' },
  { id: 3, name: '王老师', subject: '英语', avatar: 'https://placekitten.com/102/102', contact: '345-6789', officeHours: '周二至周六 8:00-16:00' },
  { id: 4, name: '赵老师', subject: '物理', avatar: 'https://placekitten.com/103/103', contact: '456-7890', officeHours: '周一至周五 9:30-17:30' },
  { id: 5, name: '刘老师', subject: '化学', avatar: 'https://placekitten.com/104/104', contact: '567-8901', officeHours: '周三至周日 11:00-19:00' },
]);

const searchQuery = ref('');
const sortBy = ref('name');
const currentPage = ref(1);
const itemsPerPage = 4;

const filteredAndSortedTeachers = computed(() => {
  let result = teachers.value.filter(teacher => 
    teacher.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    teacher.subject.toLowerCase().includes(searchQuery.value.toLowerCase())
  );

  result.sort((a, b) => a[sortBy.value].localeCompare(b[sortBy.value]));

  return result;
});

const paginatedTeachers = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredAndSortedTeachers.value.slice(start, end);
});

const totalPages = computed(() => 
  Math.ceil(filteredAndSortedTeachers.value.length / itemsPerPage)
);

function changePage(page) {
  currentPage.value = page;
}
</script>

<template>
  <div class="teacher-list">
    <h2>教师列表</h2>
    <div class="controls">
      <input v-model="searchQuery" placeholder="搜索教师或科目" class="search-input">
      <select v-model="sortBy" class="sort-select">
        <option value="name">按姓名排序</option>
        <option value="subject">按科目排序</option>
      </select>
    </div>
    <div class="teacher-grid">
      <div v-for="teacher in paginatedTeachers" :key="teacher.id" class="teacher-card">
        <img :src="teacher.avatar" :alt="teacher.name" class="teacher-avatar">
        <div class="teacher-info">
          <h3>{{ teacher.name }}</h3>
          <p>{{ teacher.subject }}</p>
          <p class="contact">联系方式: {{ teacher.contact }}</p>
          <p class="office-hours">办公时间: {{ teacher.officeHours }}</p>
        </div>
      </div>
    </div>
    <div class="pagination">
      <button 
        v-for="page in totalPages" 
        :key="page" 
        @click="changePage(page)"
        :class="{ active: currentPage === page }"
      >
        {{ page }}
      </button>
    </div>
  </div>
</template>

<style scoped>
.teacher-list {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h2 {
  text-align: center;
  margin-bottom: 20px;
}

.controls {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.search-input, .sort-select {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.search-input {
  flex-grow: 1;
  margin-right: 10px;
}

.teacher-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.teacher-card {
  background-color: #f0f0f0;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.teacher-card:hover {
  transform: translateY(-5px);
}

.teacher-avatar {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.teacher-info {
  padding: 15px;
}

.teacher-info h3 {
  margin: 0 0 5px;
  font-size: 1.2em;
}

.teacher-info p {
  margin: 0;
  color: #666;
}

.contact, .office-hours {
  font-size: 0.9em;
  margin-top: 5px;
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.pagination button {
  margin: 0 5px;
  padding: 5px 10px;
  border: 1px solid #ccc;
  background-color: #fff;
  cursor: pointer;
}

.pagination button.active {
  background-color: #007bff;
  color: #fff;
}

@media (max-width: 768px) {
  .teacher-grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  }
  
  .controls {
    flex-direction: column;
  }
  
  .search-input, .sort-select {
    width: 100%;
    margin-bottom: 10px;
  }
}

@media (max-width: 480px) {
  .teacher-grid {
    grid-template-columns: 1fr;
  }
}
</style>