<template>
  <div class="container">
    <h1>TAMBAH</h1>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Judul" /><br />
      <textarea v-model="form.content" placeholder="Konten"></textarea><br />
      <button type="submit">Simpan</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h2>{{ article.title }}</h2>
        <p>{{ article.content }}</p>
        <button @click="edit(article)">Edit</button>
        <button @click="remove(article.id)" class="delete-btn">Hapus</button>
      </li>
    </ul>
    <button @click="load" class="load-btn">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles: ', error);
      }
    }

    async function save() {
      try {
        let url, method;
        if (form.id === null) {
          const lastArticle = articles.value[articles.value.length - 1];
          const newId = lastArticle ? parseInt(lastArticle.id) + 1 : 1;
          form.id = newId.toString(); 
          url = 'http://localhost:3000/articles/';
          method = 'post';
        } else {
          url = 'http://localhost:3000/articles/' + form.id;
          method = 'put';
        }
        const response = await axios[method](url, form);
        articles.value = method === 'post' ? [...articles.value, response.data] : articles.value.map((article) => 
          article.id === response.data.id ? response.data : article
        );
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.log('Error saving article: ', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article: ', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, remove, edit, load }; 
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #f4f4f9;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
}

form {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

input[type="text"],
textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

button {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #0056b3;
}

ul {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background: #fff;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.article-item h2 {
  margin: 0;
  color: #007bff;
}

.article-item p {
  color: #555;
}

.article-item button {
  margin-right: 5px;
  background-color: #28a745;
}

.article-item button:hover {
  background-color: #218838;
}

.article-item .delete-btn {
  background-color: #dc3545;
}

.article-item .delete-btn:hover {
  background-color: #c82333;
}

.load-btn {
  display: block;
  width: 100%;
  background-color: #17a2b8;
}

.load-btn:hover {
  background-color: #138496;
}
</style>
