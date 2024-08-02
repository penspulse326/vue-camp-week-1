<script setup lang="ts">
import { ref, onMounted, nextTick, watch } from 'vue';

const drinkList = ref<Drink[] | null>(null);
const editingInput = ref<HTMLInputElement[]>([]);
const editedItem = ref<EditedItem | null>(null);

function handleEditItem(name: string, index: number) {
  const newData: EditedItem = {
    name,
    index
  };
  editedItem.value = newData;
}

function handleSaveItem() {
  if (!editedItem.value || !drinkList.value) return;

  const { name, index } = editedItem.value;

  drinkList.value[index].name = name;
  editedItem.value = {
    name: '',
    index: -1
  };
}

async function getDrinkStock() {
  try {
    const response = await fetch('./src/data.json');
    const data: Drink[] = await response.json();
    drinkList.value = data;
  } catch (error) {
    console.log(error);
  }
}

onMounted(() => {
  setTimeout(() => {
    getDrinkStock();
  }, 1000);
});

watch(editedItem, () => {
  nextTick(() => {
    editingInput.value[0]?.focus();
  });
});

interface Drink {
  name: string;
  description: string;
  price: number;
  stock: number;
}

interface EditedItem {
  name: string;
  index: number;
}
</script>

<template>
  <main>
    <div v-if="!drinkList">讀取菜單中...</div>
    <table v-else>
      <thead>
        <tr>
          <th scope="col">品項</th>
          <th scope="col">描述</th>
          <th scope="col">價格</th>
          <th scope="col">庫存</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in drinkList" :key="item.name">
          <td @dblclick="handleEditItem(item.name, index)">
            <span v-if="index !== editedItem?.index">{{ item.name }}</span>
            <input
              v-else
              type="text"
              :key="item.name"
              ref="editingInput"
              autofocus
              v-model="editedItem.name"
              @blur="handleSaveItem"
            />
          </td>
          <td>
            <small>{{ item.description }}</small>
          </td>
          <td>{{ item.price }}</td>
          <td>
            <button type="button" @click="item.stock--" class="btn">-</button>
            <span class="stock">{{ item.stock }}</span
            ><button type="button" @click="item.stock--" class="btn">+</button>
          </td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<style scoped>
table {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 8px;

  text-align: center;
}

thead {
  border-bottom: 1px solid #ccc;
}

tbody {
  border-top: 1px solid black;
}

th,
td {
  padding: 4px 8px;
  min-width: 100px;
}

input {
  max-width: 90px;
}

.btn {
  padding: 4px 8px;
  width: 24px;
  height: 24px;
  border: 1px solid #ccc;
  border-radius: 50%;
  outline: none;

  cursor: pointer;
}

.stock {
  margin: 0 8px;
}
</style>
