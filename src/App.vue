<template>
  <div class="app-container">
    <div class="background-image"></div>
    <div class="container">
      <form @submit.prevent="save" class="skincare-form">
        <div class="form-group">
          <input type="text" v-model="form.nama" placeholder="Nama Produk" />
        </div>
        <div class="form-group">
          <textarea v-model="form.jenis" placeholder="Jenis Produk"></textarea>
        </div>
        <button type="submit">Save</button>
      </form>

      <button @click="load" class="load-button">Load</button>

      <div class="table-wrapper">
        <table class="skincare-table">
          <thead>
            <tr>
              <th>Nama Produk</th>
              <th>Jenis Produk</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="skincare in skincares" :key="skincare.id">
              <td>{{ skincare.nama }}</td>
              <td>{{ skincare.jenis }}</td>
              <td>
                <button @click="edit(skincare)" class="edit-button">Edit</button>
                <button @click="del(skincare.id)" class="delete-button">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      nama: '',
      jenis: '',
    });

    const skincares = ref([]);

    async function load() {
      console.log('Tombol Load ditekan');
      try {
        const response = await axios.get('http://localhost:3000/skincares');
        console.log('Data dimuat:', response.data);
        skincares.value = response.data;
      } catch (error) {
        console.error('Kesalahan saat memuat data skincare:', error.message);
      }
    }

    async function save() {
      try {
        let method, url;

        if (form.id) {
          // Update existing item
          method = 'put';
          url = `http://localhost:3000/skincares/${form.id}`;
        } else {
          // Create new item
          method = 'post';
          url = 'http://localhost:3000/skincares';
        }

        const response = await axios[method](url, {
          nama: form.nama,
          jenis: form.jenis,
        });

        if (method === 'put') {
          skincares.value = skincares.value.map(skincare =>
            skincare.id === response.data.id ? response.data : skincare
          );
        } else {
          skincares.value.push(response.data);
        }

        form.id = null;
        form.nama = '';
        form.jenis = '';
      } catch (error) {
        console.error('Kesalahan saat menyimpan skincare:', error.message);
      }
    }

    async function del(id) {
      console.log(`ID yang diterima untuk penghapusan: ${id}`);
      if (!id) {
        console.error('Kesalahan: ID adalah null atau tidak terdefinisi');
        return;
      }

      if (confirm('Apakah Anda yakin ingin menghapus item ini?')) {
        try {
          console.log(`Menghapus item skincare dengan id ${id}`);
          await axios.delete(`http://localhost:3000/skincares/${id}`);
          skincares.value = skincares.value.filter(skincare => skincare.id !== id);
          console.log(`Item skincare dengan id ${id} telah dihapus`);
        } catch (error) {
          console.error('Kesalahan saat menghapus skincare:', error.message);
        }
      }
    }

    function edit(skincare) {
      form.id = skincare.id;
      form.nama = skincare.nama;
      form.jenis = skincare.jenis;
    }

    onMounted(load);

    return { form, skincares, save, edit, del, load };
  },
};
</script>

<style>
:root {
  --background-color: #EAF1B1; /* Pale Lime */
  --primary-color: rgba(255, 255, 255, 0.7); /* White with transparency */
  --secondary-color: #728C5A;  /* Sage */
  --text-color: #102F15;       /* Dark Forest */
  --button-color: rgba(114, 140, 90, 0.7); /* Sage with transparency */
  --button-hover-color: rgba(16, 47, 21, 0.7); /* Dark Forest with transparency */
  --edit-button-color: rgba(114, 140, 90, 0.7); /* Sage with transparency */
  --edit-button-hover-color: rgba(16, 47, 21, 0.7); /* Dark Forest with transparency */
  --delete-button-color: rgba(16, 47, 21, 0.7); /* Dark Forest with transparency */
  --delete-button-hover-color: rgba(114, 140, 90, 0.7); /* Sage with transparency */
  --table-header-bg: #728C5A;  /* Sage */
  --table-border: #EAF1B1;     /* Pale Lime */
  --table-row-even-bg: #FFFFFF; /* White */
  --table-row-odd-bg: #F7F5F2; /* Light Almond */
  --table-hover-bg: #F0ECE3;   /* Light Coffee */
}

* {
  font-family:'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  color: var(--text-color);
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  position: relative;
}

.background-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url('../src/assets/background.jpg'); 
  background-size: cover;
  background-position: center;
  z-index: -1;
  opacity: 0.5;
}

.app-container {
  width: 100%;
  max-width: 800px;
  padding: 20px;
}

.container {
  background-color: transparent;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(15px); 
  justify-content: center;
  margin-left: 320px;
  width: 500px;
}

.container:hover {
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

.skincare-form {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.skincare-form .form-group {
  display: flex;
  flex-direction: column;
}

.skincare-form input,
.skincare-form textarea {
  width: 100%;
  margin-bottom: 10px;
  padding: 12px;
  font-size: 16px;
  background-color: #EAF1B1;
  border: none;
  border-radius: 6px;
  outline: none;
  transition: background-color 0.3s, border 0.3s;
}

.skincare-form input:focus,
.skincare-form textarea:focus {
  background-color: #F7F5F2;
}

.skincare-form input::placeholder,
.skincare-form textarea::placeholder {
  color: #888;
}

.skincare-form button {
  padding: 12px 20px;
  font-size: 16px;
  background-color: var(--button-color);
  color: #ffffff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s, box-shadow 0.3s;
  backdrop-filter: blur(10px);
}

.skincare-form button:hover {
  background-color: var(--button-hover-color);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.load-button {
  margin-bottom: 20px;
  padding: 12px 20px;
  font-size: 16px;
  background-color: var(--button-color);
  color: #ffffff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s, box-shadow 0.3s;
  backdrop-filter: blur(10px);
}

.load-button:hover {
  background-color: var(--button-hover-color);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.table-wrapper {
  overflow-x: auto;
}

.skincare-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  border-radius: 10px;
}

.skincare-table thead th {
  border-bottom: 2px solid var(--table-border);
  padding: 12px;
  text-align: left;
  background-color: var(--table-header-bg);
  color: #ffffff;
  text-transform: uppercase;
  font-weight: normal;
}

.skincare-table tbody td {
  border-bottom: 1px solid var(--table-border);
  padding: 12px;
}

.skincare-table tbody tr:nth-child(even) {
  background-color: var(--table-row-even-bg);
}

.skincare-table tbody tr:nth-child(odd) {
  background-color: var(--table-row-odd-bg);
}

.skincare-table tbody tr:hover {
  background-color: var(--table-hover-bg);
}

.edit-button,
.delete-button {
  margin-right: 5px;
  padding: 8px 12px;
  font-size: 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s, box-shadow 0.3s;
  backdrop-filter: blur(10px);
}

.edit-button {
  background-color: var(--edit-button-color);
  color: #ffffff;
}

.edit-button:hover {
  background-color: var(--edit-button-hover-color);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.delete-button {
  background-color: var(--delete-button-color);
  color: #ffffff;
}

.delete-button:hover {
  background-color: var(--delete-button-hover-color);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

</style>
