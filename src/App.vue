<script setup>
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios'


// Form state
const form = reactive({
  id: null,
  title: '',
  content: ''
})

const articles = ref([])
const loading = ref(false)
const error = ref('')

// Load data from API
const load = async () => {
  loading.value = true
  try {
    const res = await axios.get('https://juicy-repeated-venus.glitch.me/articles')
    articles.value = res.data
  } catch (err) {
    error.value = 'Gagal memuat data'
  } finally {
    loading.value = false
  }
}

// Save or update article
const save = async () => {
  if (!form.title.trim() || !form.content.trim()) {
    error.value = 'Judul dan isi harus diisi!'
    return
  }

  try {
    const url = form.id
      ? `https://juicy-repeated-venus.glitch.me/articles/${form.id}`
      : 'https://juicy-repeated-venus.glitch.me/articles'
    const method = form.id ? 'put' : 'post'
    const res = await axios[method](url, {
      title: form.title,
      content: form.content
    })

    if (form.id) {
      articles.value = articles.value.map((a) =>
        a.id === res.data.id ? res.data : a
      )
    } else {
      articles.value.push(res.data)
    }

    resetForm()
  } catch (err) {
    error.value = 'Gagal menyimpan data'
  }
}

// Fill form with article to edit
const edit = (article) => {
  form.id = article.id
  form.title = article.title
  form.content = article.content
}

// Delete article
const remove = async (id) => {
  if (!confirm('Yakin ingin menghapus artikel ini?')) return
  try {
    await axios.delete(`https://juicy-repeated-venus.glitch.me/articles/${id}`)
    articles.value = articles.value.filter((a) => a.id !== id)
  } catch (err) {
    error.value = 'Gagal menghapus data'
  }
}

// Reset form state
const resetForm = () => {
  form.id = null
  form.title = ''
  form.content = ''
}

// Load on mount
onMounted(load)
</script>

<template>
  <div class="container">
    <h1>📝 Manajemen Artikel</h1>

    <div class="form-section">
      <h2>{{ form.id ? 'Edit Artikel' : 'Tambah Artikel' }}</h2>

      <form @submit.prevent="save" class="form-card">
        <input v-model="form.title" placeholder="Judul Artikel" class="input" />
        <textarea v-model="form.content" placeholder="Isi Artikel" class="textarea" />

        <div class="form-actions">
          <button type="submit" class="btn btn-primary">
            {{ form.id ? 'Update' : 'Simpan' }}
          </button>
          <button v-if="form.id" type="button" class="btn btn-secondary" @click="resetForm">
            Batal
          </button>
        </div>

        <p v-if="error" class="error-msg">{{ error }}</p>
      </form>
    </div>

    <div class="article-section">
      <div class="toolbar">
        <h2>📚 Daftar Artikel</h2>
        <button @click="load" class="btn btn-refresh">🔃 Muat Ulang</button>
      </div>

      <div v-if="loading" class="loading">Loading data...</div>

      <div v-else-if="articles.length === 0" class="no-data">
        Tidak ada artikel ditemukan.
      </div>

      <div class="grid">
        <div v-for="article in articles" :key="article.id" class="card">
          <h3>{{ article.title }}</h3>
          <p>{{ article.content }}</p>
          <div class="actions">
            <button class="btn btn-edit" @click="edit(article)">Edit</button>
            <button class="btn btn-delete" @click="remove(article.id)">Hapus</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap');

* {
  box-sizing: border-box;
}
body {
  margin: 0;
  font-family: 'Inter', sans-serif;
  background: #f3f4f6;
}
.container {
  max-width: 900px;
  margin: auto;
  padding: 2rem;
  color: #1f2937;
}
h1 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 2rem;
  color: #bfcae2;
}

.form-section {
  margin-bottom: 2rem;
}
.form-card {
  background: #fff;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}
.input,
.textarea {
  width: 100%;
  margin-bottom: 1rem;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  font-size: 1rem;
}
.textarea {
  height: 120px;
  resize: vertical;
}
.form-actions {
  display: flex;
  gap: 0.75rem;
}
.btn {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s ease;
}
.btn-primary {
  background-color: #3b82f6;
  color: white;
}
.btn-primary:hover {
  background-color: #2563eb;
}
.btn-secondary {
  background-color: #e5e7eb;
  color: #374151;
}
.btn-secondary:hover {
  background-color: #d1d5db;
}
.btn-refresh {
  background: #10b981;
  color: white;
}
.btn-refresh:hover {
  background: #059669;
}
.article-section {
  margin-top: 2rem;
}
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}
.grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
}
.card {
  background: rgb(255, 255, 255);
  border-radius: 10px;
  padding: 1.25rem;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease;
}
.card:hover {
  transform: translateY(-2px);
}
.card h3 {
  margin-top: 0;
  font-size: 1.25rem;
  word-break: break-word; /* ⬅️ tambahkan ini */
  overflow-wrap: anywhere; /* opsional tambahan */
}

.card p {
  margin: 0.75rem 0;
  color: #4b5563;
  line-height: 1.5;
  word-break: break-word; /* ⬅️ tambahkan ini */
  overflow-wrap: anywhere;
}

.actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}
.btn-edit {
  background-color: #facc15;
  color: #1f2937;
}
.btn-edit:hover {
  background-color: #eab308;
}
.btn-delete {
  background-color: #ef4444;
  color: white;
}
.btn-delete:hover {
  background-color: #dc2626;
}
.error-msg {
  margin-top: 0.75rem;
  color: #dc2626;
  font-weight: 500;
}
.loading {
  padding: 1rem;
  text-align: center;
  color: #6b7280;
}
.no-data {
  padding: 1rem;
  text-align: center;
  color: #9ca3af;
}
</style>
