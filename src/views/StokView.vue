<template>
  <div class="stok">
    <h1>📦 Manajemen Stok Barang</h1>
    
    <!-- Tombol Tambah -->
    <button @click="openAddDialog" class="btn-add">
      + Tambah Barang
    </button>

    <!-- Tabel List Barang -->
    <div class="table-container">
      <table class="data-table">
        <thead>
          <tr>
            <th>No</th>
            <th>Nama Barang</th>
            <th>Deskripsi</th>
            <th>Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="items.length === 0">
            <td colspan="4" class="empty-data">Belum ada data. Silakan tambah barang!</td>
          </tr>
          <tr v-for="(item, index) in paginatedItems" :key="item.id">
            <td>{{ (currentPage - 1) * itemsPerPage + index + 1 }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.description }}</td>
            <td class="actions">
              <button @click="openEditDialog(item)" class="btn-edit">✏️ Edit</button>
              <button @click="confirmDelete(item)" class="btn-delete">🗑️ Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination Controls -->
    <div class="pagination" v-if="totalPages > 1">
      <button 
        @click="changePage(currentPage - 1)" 
        :disabled="currentPage === 1"
        class="btn-page"
      >
        &laquo; Prev
      </button>
      
      <div class="page-numbers">
        <span 
          v-for="page in totalPages" 
          :key="page"
          @click="changePage(page)"
          :class="['page-number', { active: currentPage === page }]"
        >
          {{ page }}
        </span>
      </div>

      <button 
        @click="changePage(currentPage + 1)" 
        :disabled="currentPage === totalPages"
        class="btn-page"
      >
        Next &raquo;
      </button>
    </div>

    <!-- Modal Pop Up (Custom) -->
    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <div class="modal-header">
          <h2>{{ isEditMode ? '✏️ Edit Barang' : '➕ Tambah Barang' }}</h2>
          <button @click="closeModal" class="close-btn">&times;</button>
        </div>
        
        <div class="modal-body">
          <div class="form-group">
            <label>Nama Barang</label>
            <input 
              type="text" 
              v-model="formData.name" 
              placeholder="Masukkan nama barang"
              class="form-input"
            />
          </div>
          
          <div class="form-group">
            <label>Deskripsi</label>
            <textarea 
              v-model="formData.description" 
              placeholder="Masukkan deskripsi barang"
              class="form-textarea"
              rows="4"
            ></textarea>
          </div>
        </div>
        
        <div class="modal-footer">
          <button @click="closeModal" class="btn-cancel">Batal</button>
          <button @click="submitForm" class="btn-submit">
            {{ isEditMode ? 'Update' : 'Simpan' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import Swal from 'sweetalert2'

// Data items
const items = ref([])
const showModal = ref(false)
const isEditMode = ref(false)
const editId = ref(null)

// Pagination data
const currentPage = ref(1)
const itemsPerPage = ref(5) // Tampilkan 5 data per halaman

const totalPages = computed(() => {
  return Math.ceil(items.value.length / itemsPerPage.value)
})

const paginatedItems = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value
  const end = start + itemsPerPage.value
  return items.value.slice(start, end)
})

const changePage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page
  }
}

watch(items, () => {
  // Jika hapus item dari halaman terakhir, sesuaikan halaman saat ini
  if (currentPage.value > totalPages.value && totalPages.value > 0) {
    currentPage.value = totalPages.value
  }
}, { deep: true })

// Form data
const formData = ref({
  name: '',
  description: ''
})

// Load data dari localStorage saat pertama kali
onMounted(() => {
  const savedData = localStorage.getItem('stokBarang')
  if (savedData) {
    items.value = JSON.parse(savedData)
  } else {
    // Data contoh
    items.value = [
      { id: 1, name: 'Laptop', description: 'Laptop gaming 16GB RAM' },
      { id: 2, name: 'Mouse', description: 'Mouse wireless logitech' },
      { id: 3, name: 'Keyboard', description: 'Keyboard mechanical RGB' }
    ]
    saveToLocalStorage()
  }
})

// Simpan ke localStorage
const saveToLocalStorage = () => {
  localStorage.setItem('stokBarang', JSON.stringify(items.value))
}

// Generate ID baru
const generateId = () => {
  return Date.now()
}

// Buka modal tambah
const openAddDialog = () => {
  isEditMode.value = false
  editId.value = null
  formData.value = {
    name: '',
    description: ''
  }
  showModal.value = true
}

// Buka modal edit
const openEditDialog = (item) => {
  isEditMode.value = true
  editId.value = item.id
  formData.value = {
    name: item.name,
    description: item.description
  }
  showModal.value = true
}

// Tutup modal
const closeModal = () => {
  showModal.value = false
  formData.value = {
    name: '',
    description: ''
  }
}

// Submit form (Tambah/Edit)
const submitForm = () => {
  // Validasi
  if (!formData.value.name.trim()) {
    Swal.fire({
      icon: 'warning',
      title: 'Oops...',
      text: 'Nama barang wajib diisi!',
      confirmButtonColor: '#42b883'
    })
    return
  }
  
  if (!formData.value.description.trim()) {
    Swal.fire({
      icon: 'warning',
      title: 'Oops...',
      text: 'Deskripsi barang wajib diisi!',
      confirmButtonColor: '#42b883'
    })
    return
  }

  if (isEditMode.value) {
    // Edit data
    const index = items.value.findIndex(item => item.id === editId.value)
    if (index !== -1) {
      items.value[index] = {
        ...items.value[index],
        name: formData.value.name,
        description: formData.value.description
      }
      Swal.fire({
        icon: 'success',
        title: 'Berhasil!',
        text: 'Data barang berhasil diupdate',
        timer: 1500,
        showConfirmButton: false
      })
    }
  } else {
    // Tambah data baru
    const newItem = {
      id: generateId(),
      name: formData.value.name,
      description: formData.value.description
    }
    items.value.push(newItem)
    Swal.fire({
      icon: 'success',
      title: 'Berhasil!',
      text: 'Data barang berhasil ditambahkan',
      timer: 1500,
      showConfirmButton: false
    })
  }
  
  saveToLocalStorage()
  closeModal()
}

// Konfirmasi delete dengan SweetAlert2
const confirmDelete = (item) => {
  Swal.fire({
    title: `Hapus "${item.name}"?`,
    text: "Data yang dihapus tidak dapat dikembalikan!",
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    cancelButtonColor: '#42b883',
    confirmButtonText: 'Ya, hapus!',
    cancelButtonText: 'Batal',
    reverseButtons: true
  }).then((result) => {
    if (result.isConfirmed) {
      // Hapus data
      const index = items.value.findIndex(i => i.id === item.id)
      if (index !== -1) {
        items.value.splice(index, 1)
        saveToLocalStorage()
        Swal.fire({
          title: 'Terhapus!',
          text: 'Data barang berhasil dihapus',
          icon: 'success',
          timer: 1500,
          showConfirmButton: false
        })
      }
    }
  })
}
</script>

<style scoped>
.stok {
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

.stok h1 {
  color: #42b883;
  text-align: center;
  margin-bottom: 2rem;
}

/* Tombol Tambah */
.btn-add {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  margin-bottom: 2rem;
  transition: all 0.3s ease;
}

.btn-add:hover {
  background-color: #359268;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.3);
}

/* Tabel */
.table-container {
  overflow-x: auto;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
}

.data-table thead {
  background: linear-gradient(135deg, #42b883 0%, #35495e 100%);
  color: white;
}

.data-table th,
.data-table td {
  padding: 12px 16px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.data-table th {
  font-weight: 600;
}

.data-table tbody tr:hover {
  background-color: #f5f5f5;
}

.empty-data {
  text-align: center;
  padding: 40px !important;
  color: #999;
}

/* Tombol Aksi */
.actions {
  display: flex;
  gap: 8px;
}

.btn-edit,
.btn-delete {
  padding: 6px 12px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s ease;
}

.btn-edit {
  background-color: #3498db;
  color: white;
}

.btn-edit:hover {
  background-color: #2980b9;
}

.btn-delete {
  background-color: #e74c3c;
  color: white;
}

.btn-delete:hover {
  background-color: #c0392b;
}

/* Modal Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Modal Content */
.modal-content {
  background: white;
  border-radius: 16px;
  width: 90%;
  max-width: 500px;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    transform: translateY(-50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid #eee;
}

.modal-header h2 {
  margin: 0;
  color: #35495e;
}

.close-btn {
  background: none;
  border: none;
  font-size: 28px;
  cursor: pointer;
  color: #999;
  transition: color 0.3s;
}

.close-btn:hover {
  color: #e74c3c;
}

.modal-body {
  padding: 24px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #35495e;
}

.form-input,
.form-textarea {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
  border-color: #42b883;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 16px 24px;
  border-top: 1px solid #eee;
}

.btn-cancel,
.btn-submit {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.btn-cancel {
  background-color: #95a5a6;
  color: white;
}

.btn-cancel:hover {
  background-color: #7f8c8d;
}

.btn-submit {
  background-color: #42b883;
  color: white;
}

.btn-submit:hover {
  background-color: #359268;
}

/* Responsive */
@media (max-width: 768px) {
  .stok {
    padding: 1rem;
  }
  
  .data-table th,
  .data-table td {
    padding: 8px 12px;
    font-size: 0.9rem;
  }
  
  .actions {
    flex-direction: column;
  }
}

/* Pagination */
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  margin-top: 2rem;
}

.page-numbers {
  display: flex;
  gap: 0.5rem;
}

.page-number {
  padding: 8px 12px;
  border-radius: 6px;
  background-color: #f5f5f5;
  color: #35495e;
  cursor: pointer;
  transition: all 0.3s;
  font-weight: 500;
  min-width: 35px;
  text-align: center;
}

.page-number:hover {
  background-color: #e0e0e0;
}

.page-number.active {
  background-color: #42b883;
  color: white;
  box-shadow: 0 2px 8px rgba(66, 184, 131, 0.3);
}

.btn-page {
  padding: 8px 16px;
  border: none;
  background-color: #fff;
  color: #42b883;
  border: 1px solid #42b883;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
}

.btn-page:hover:not(:disabled) {
  background-color: #42b883;
  color: white;
}

.btn-page:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  border-color: #ccc;
  color: #999;
}
</style>