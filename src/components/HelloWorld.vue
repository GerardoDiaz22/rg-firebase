<template>
  <v-app>
    <v-container>
      <!-- Sección superior -->
      <v-text-field v-model="book.name" label="Nombre"></v-text-field>
      <v-text-field v-model="book.author" label="Autor"></v-text-field>
      <v-text-field v-model="book.year" label="Año"></v-text-field>
      <v-select v-model="book.genres" :items="genres" label="Géneros" multiple></v-select>
      <v-btn @click="addBook">Agregar</v-btn>

      <!-- Sección inferior -->
      <v-data-table :headers="headers" :items="books" class="elevation-1">
        <template v-slot:item.name="{ item }">
          {{ item.data.name }}
        </template>
        <template v-slot:item.author="{ item }">
          {{ item.data.author }}
        </template>
        <template v-slot:item.year="{ item }">
          {{ item.data.year }}
        </template>
        <template v-slot:item.genres="{ item }">
          {{ item.data.genres.join(', ') }}
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon size="small" @click="deleteBook(item)">
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-container>
  </v-app>
</template>

<script>
import { ref, onMounted } from 'vue'
import 'firebase/firestore'

// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs, addDoc, deleteDoc, doc  } from 'firebase/firestore/lite';

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: import.meta.env.VITE_API_KEY,
  authDomain: import.meta.env.VITE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_APP_ID,
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

export default {
  setup() {
    const book = ref({ name: '', author: '', year: '', genres: [] })
    const genres = ref(['Ficción', 'No ficción', 'Ciencia ficción', 'Fantasía', 'Misterio', 'Biografía', 'Terror', 'Romance'])
    const books = ref([])
    const headers = ref([
      { title: 'Nombre', key: 'name' },
      { title: 'Autor', key: 'author' },
      { title: 'Año', key: 'year', sortable: true },
      { title: 'Género', key: 'genres' },
      { title: 'Acciones', key: 'actions' }
    ])

    const addBook = async () => {
      try {
        const docRef = await addDoc(collection(db, "books"), book.value);
        book.value.id = docRef.id;
        books.value.push(book.value);
        book.value = { name: '', author: '', year: '', genres: [] };
      } catch (e) {
        console.error("Error adding document: ", e);
      }
    }

    const deleteBook = async (item) => {
      const deletedBook = await deleteDoc(doc(db, "books", item.id));
      books.value  = books.value.filter(b => b.id != item.id);
    }

    onMounted(async () => {
      const booksCol = collection(db, 'books');
      const booksSnapshot = await getDocs(booksCol);
      const booksList = booksSnapshot.docs.map(doc => ({
        'id': doc.id, 
        'data': doc.data()
      }));

      console.log(booksList);

      booksList.forEach(book => {
        books.value.push(book)
      })
    })

    return { book, genres, books, headers, addBook, deleteBook }
  }
}
</script>
