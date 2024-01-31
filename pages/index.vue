<template>
  <div>
    <div class="flex px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput v-model="q" placeholder="Filtreaza carti" />
    </div>

    <!-- Formular pentru introducerea manuală a datelor -->
    <div class="p-3">
      <label for="author">Autor:</label>
      <UInput v-model="newRecord.author" id="author" />

      <label for="book">Titlu carte:</label>
      <UInput v-model="newRecord.book" id="book" />

      <label for="language">Scris in limba:</label>
      <UInput v-model="newRecord.language" id="language" />

      <label for="published">Publicat in anul:</label>
      <UInput v-model="newRecord.published" type="number" id="published" />

      <label for="sales">Vanzari:</label>
      <UInput v-model="newRecord.sales" type="number" id="sales" />

      <button @click="addNewRecord" class="bg-blue-500 text-white px-4 py-2 rounded mt-3">
        Adauga Carte Noua
      </button>
    </div>

    <UTable :rows="rows" :columns="columns"></UTable>

    <div class="flex justify-end px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import * as Realm from 'realm-web';

const { data: carti, pending } = useLazyAsyncData<any[]>('carti', async () => {
  const app = new Realm.App({ id: 'data-syppe' });
  const credentials = Realm.Credentials.apiKey('0kpD5gNR9pARmYv7V1yoMV6c2jiBkjfF28d3Pca1Rfx2DCxgqocD2U3p1E4jCAIA');
  await app.logIn(credentials);

  if (app.currentUser) {
    const mongo = app.currentUser.mongoClient('Cluster0');
    const collection = mongo.db('Carti').collection('CartiSuperbe');
    const results = await collection.find();
    return results.map((result) => ({
      author: result.author,
      book: result.book,
      language: result.language,
      published: result.published,
      sales: result.sales
    }));
  }

  return [];
}, {
  default: () => [],
  watch: []
});

const columns = [
  {
    key: 'book',
    label: 'Titlu carte',
    sortable: true,
  },
  {
    key: 'author',
    label: 'Autor',
    sortable: true,
  },
  {
    key: 'language',
    label: 'Scris in limba',
    sortable: true,
  },
  {
    key: 'published',
    label: 'Publicat in anul',
    sortable: true,
  },
  {
    key: 'sales',
    label: 'Vanzari',
    sortable: true,
  }
];

const q = ref('');
const newRecord = ref({
  author: '',
  book: '',
  language: '',
  published: 0,
  sales: 0
});

const addNewRecord = async () => {
  const app = new Realm.App({ id: 'data-syppe' });
  const credentials = Realm.Credentials.apiKey('0kpD5gNR9pARmYv7V1yoMV6c2jiBkjfF28d3Pca1Rfx2DCxgqocD2U3p1E4jCAIA');
  await app.logIn(credentials);

  if (app.currentUser) {
    const mongo = app.currentUser.mongoClient('Cluster0');
    const collection = mongo.db('Carti').collection('CartiSuperbe');

    await collection.insertOne(newRecord.value);

    const updatedResults = await collection.find();
    carti.value = updatedResults.map((result) => ({
      author: result.author,
      book: result.book,
      language: result.language,
      published: result.published,
      sales: result.sales
    }));
  }

  newRecord.value = {
    author: '',
    book: '',
    language: '',
    published: 0,
    sales: 0
  };
};

const filteredRows = computed(() => {
  if (!q.value) {
    return carti.value;
  }

  const lowercasedQuery = q.value.toLowerCase();

  return carti.value.filter((row) => {
    return Object.values(row).some((value) => {
      return String(value).toLowerCase().includes(lowercasedQuery);
    });
  });
});

const page = ref(1);
const pageCount = 15;

const rows = computed(() => {
  const startIndex = (page.value - 1) * pageCount;
  const endIndex = startIndex + pageCount;
  return filteredRows.value.slice(startIndex, endIndex);
});
</script>
