<template>
  <v-app>
    <v-container>
      <!-- Sección superior -->
      <v-text-field v-model="bookToAdd.name" label="Nombre"></v-text-field>
      <v-text-field v-model="bookToAdd.author" label="Autor"></v-text-field>
      <v-text-field v-model="bookToAdd.year" label="Año"></v-text-field>
      <v-select v-model="bookToAdd.genres" :items="genres" label="Géneros" multiple></v-select>
      <v-btn @click="addBook">Agregar</v-btn>

      <!-- Sección inferior -->
      <v-data-table :headers="headers" :items="books" class="elevation-1">
        <template v-slot:item.name="{ item }">
          {{ item.name }}
        </template>
        <template v-slot:item.author="{ item }">
          {{ item.author }}
        </template>
        <template v-slot:item.year="{ item }">
          {{ item.year }}
        </template>
        <template v-slot:item.genres="{ item }">
          {{ item.genres.join(', ') }}
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon class="mr-2" @click="openEditDialog(item)"> mdi-pencil </v-icon>
          <v-icon @click="deleteBook(item)"> mdi-delete </v-icon>
        </template>
      </v-data-table>

      <!-- Dialogo para editar el libro seleccionado -->
      <v-dialog v-model="isEditDialogOpen" max-width="500px">
        <v-card>
          <v-card-title>Edit Book</v-card-title>
          <v-card-text>
            <v-text-field v-model="bookToEdit.name" label="Nombre"></v-text-field>
            <v-text-field v-model="bookToEdit.author" label="Autor"></v-text-field>
            <v-text-field v-model="bookToEdit.year" label="Año"></v-text-field>
            <v-select v-model="bookToEdit.genres" :items="genres" label="Géneros" multiple></v-select>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" text @click="saveEdit">Save</v-btn>
            <v-btn color="secondary" text @click="closeEditDialog">Cancel</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </v-app>
</template>

<script>
import { ref, onMounted } from 'vue';
import 'firebase/firestore';

// Import the functions you need from the SDKs you need
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, getDocs, addDoc, deleteDoc, doc, updateDoc } from 'firebase/firestore/lite';

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
    const bookToAdd = ref({ name: '', author: '', year: '', genres: [] });
    const genres = ref([
      'Ficción',
      'No ficción',
      'Ciencia ficción',
      'Fantasía',
      'Misterio',
      'Biografía',
      'Terror',
      'Romance',
    ]);
    const books = ref([]);
    const headers = ref([
      { title: 'Nombre', key: 'name' },
      { title: 'Autor', key: 'author' },
      { title: 'Año', key: 'year', sortable: true },
      { title: 'Género', key: 'genres' },
      { title: 'Acciones', key: 'actions' },
    ]);

    const isEditDialogOpen = ref(false);
    const bookToEdit = ref({ name: '', author: '', year: '', genres: [] });

    const addBook = async () => {
      try {
        const docRef = await addDoc(collection(db, 'books'), bookToAdd.value);
        const newBook = {
          id: docRef.id,
          ...bookToAdd.value,
        };
        books.value.push(newBook);
        bookToAdd.value = { name: '', author: '', year: '', genres: [] };
      } catch (e) {
        console.error('Error adding document: ', e);
      }
    };

    const deleteBook = async (item) => {
      const deletedBook = await deleteDoc(doc(db, 'books', item.id));
      books.value = books.value.filter((b) => b.id != item.id);
    };

    // Open edit dialog with the selected book to edit
    const openEditDialog = (item) => {
      bookToEdit.value = item;
      isEditDialogOpen.value = true;
    };

    // Close edit dialog and reset bookToEdit
    const closeEditDialog = () => {
      bookToEdit.value = { name: '', author: '', year: '', genres: [] };
      isEditDialogOpen.value = false;
    };

    const saveEdit = async () => {
      try {
        // Get a reference to the book document
        const bookRef = doc(db, 'books', bookToEdit.value.id);

        // Update the book in Firestore
        await updateDoc(bookRef, bookToEdit.value);

        // Update local books
        const bookIndex = books.value.findIndex((book) => book.id === bookToEdit.value.id);
        books.value[bookIndex] = { ...bookToEdit.value };

        // Close dialog
        closeEditDialog();
      } catch (e) {
        console.error('Error updating document: ', e);
      }
    };

    onMounted(async () => {
      const booksCol = collection(db, 'books');
      const booksSnapshot = await getDocs(booksCol);
      const booksList = booksSnapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));

      console.log(booksList);

      booksList.forEach((book) => {
        books.value.push(book);
      });
    });

    return {
      bookToAdd,
      genres,
      books,
      headers,
      addBook,
      deleteBook,
      isEditDialogOpen,
      bookToEdit,
      openEditDialog,
      closeEditDialog,
      saveEdit,
    };
  },
};
</script>
