<template>
  <div class="category-page">
    <div class="header">
      <input v-model="keyword" placeholder="Keyword" class="search" />
      <button @click="goToNew" class="btn-new">+ New</button>
    </div>

    <div class="table-container">
      <table class="table">
        <thead>
          <tr>
            <th>Thumbnail</th>
            <th>Title</th>
            <th>Slug</th>
            <th>Description</th>
            <th>Status</th>
            <th>Updated at</th>
            <th>Operations</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="cat in paginatedCategories" :key="cat.id">
            <td><!-- Thumbnail nếu có --></td>
            <td>{{ cat.name?.origin || cat.slug || cat.id }}</td>
            <td>{{ cat.slug }}</td>
            <td>{{ cat.description }}</td>
            <td>{{ cat.status || '-' }}</td>
            <td>{{ cat.updated_at ? new Date(cat.updated_at).toLocaleString() : '-' }}</td>
            <td>
              <router-link :to="`/websites/categories/${cat.id}`" class="btn">Edit</router-link>
            </td>
          </tr>
          <tr v-if="filteredCategories.length === 0">
            <td colspan="7" style="text-align:center;">No Data</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="pagination" v-if="totalPages > 1">
      <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
      <span>Page {{ currentPage }} / {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

definePageMeta({
  layout: 'websites'
})

const categories = ref([])
const keyword = ref('')
const router = useRouter()

// Pagination
const currentPage = ref(1)
const itemsPerPage = 10

// Fetch categories
onMounted(async () => {
  try {
    const res = await axios.get('/api/v1/websites/categories/')
    console.log('API response:', res.data)
    categories.value = Array.isArray(res.data) ? res.data : (res.data.results || [])
  } catch (e) {
    console.error('Error fetching categories:', e)
    categories.value = []
  }
})

// Filtered categories
const filteredCategories = computed(() => {
  if (!keyword.value) return categories.value
  return categories.value.filter(cat =>
    (cat.name?.origin || '').toLowerCase().includes(keyword.value.toLowerCase()) ||
    (cat.slug || '').toLowerCase().includes(keyword.value.toLowerCase()) ||
    (cat.description || '').toLowerCase().includes(keyword.value.toLowerCase())
  )
})

// Pagination logic
const totalPages = computed(() => Math.ceil(filteredCategories.value.length / itemsPerPage))

const paginatedCategories = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredCategories.value.slice(start, start + itemsPerPage)
})

function prevPage() {
  if (currentPage.value > 1) currentPage.value--
}

function nextPage() {
  if (currentPage.value < totalPages.value) currentPage.value++
}

// Reset page if keyword changes
watch(keyword, () => {
  currentPage.value = 1
})

function goToNew() {
  router.push('/websites/categories/new')
}
</script>

<style scoped>
.category-page {
  max-width: 1200px;
  margin: 40px auto;
  background: #fff;
  padding: 32px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.header {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin-bottom: 16px;
}

.search {
  padding: 8px;
  margin-right: 12px;
  border-radius: 4px;
  border: 1px solid #ddd;
}

.btn-new {
  padding: 8px 20px;
  background: #b71c1c;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.table-container {
  max-height: 500px;
  overflow-y: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 12px;
}

.table th, .table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

.table th {
  background: #b71c1c;
  color: #fff;
}

.btn {
  padding: 4px 12px;
  background: #b71c1c;
  color: #fff;
  border: none;
  border-radius: 4px;
  text-decoration: none;
}

/* Pagination */
.pagination {
  margin-top: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 12px;
}

.pagination button {
  padding: 6px 12px;
  border-radius: 4px;
  border: 1px solid #b71c1c;
  background: #fff;
  color: #b71c1c;
  cursor: pointer;
}

.pagination button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
