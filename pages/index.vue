<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import * as Realm from 'realm-web';

const { data: carti, pending } = useLazyAsyncData<any[]>('carti', async () => {
  const app = new Realm.App({ id: 'data-syppe' });
  const credentials = Realm.Credentials.apiKey('0kpD5gNR9pARmYv7V1yoMV6c2jiBkjfF28d3Pca1Rfx2DCxgqocD2U3p1E4jCAIA');
  await app.logIn(credentials);

  if (app.currentUser) {
    const mongo = app.currentUser.mongoClient('Cluster0');
    const collection = mongo.db('Carti').collection('CartiSuperbe');
    const results = await collection.find();
    console.log(results)
    return results.map((result) => ({
        author: result.author,  
        book: result.book,
        language: result.language,
        published: result.published,
        sales: result.sales
    }));
  }

  return []; // Default value when data fetching fails or no data available
}, {
  default: () => [],
  watch: [] // Add the necessary dependencies here
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

const q = ref('')

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
const page = ref(1)
const pageCount = 15

const rows = computed(() => {
  const startIndex = (page.value - 1) * pageCount;
  const endIndex = startIndex + pageCount;
  return filteredRows.value.slice(startIndex, endIndex);
});
</script>

<template>
  <div>
    <div class="flex px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput v-model="q" placeholder="Filtreaza carti" />
    </div>
    <UTable :rows="rows" :columns="columns"></UTable>
    <div class="flex justify-end px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length" />
    </div>
  </div>
</template>
