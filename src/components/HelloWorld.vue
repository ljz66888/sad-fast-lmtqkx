<!-- List.vue -->
<template>
  <div class="list-container">
    <div
      v-for="(item, index) in list"
      :key="index"
      class="list-item"
      :style="{ background: item.background }"
    >
      {{ index }}
    </div>

    <div v-if="isLoading" class="loading">
      Loading...
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

const generateRandomColor = () => {
  const r = Math.floor(Math.random() * 256);
  const g = Math.floor(Math.random() * 256);
  const b = Math.floor(Math.random() * 256);
  return `rgb(${r},${g},${b})`;
};

const list = ref([{ background: "rgb(233,32,38)" }]);
const isLoading = ref(false);
const totalLimit = 50;

const getList = (num: number): Promise<Array<{ background: string }>> => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(
        Array.from({ length: num }, () => ({
          background: generateRandomColor(),
        }))
      );
    }, Math.random() * 5000);
  });
};

const handleScroll = () => {
  if (isLoading.value) return;
  if (list.value.length >= totalLimit) return;

  const scrollHeight = document.documentElement.scrollHeight;
  const scrollTop = document.documentElement.scrollTop;
  const clientHeight = document.documentElement.clientHeight;

  if (scrollTop + clientHeight >= scrollHeight / 2) {
    loadMore();
  }
};

const loadMore = async () => {
  try {
    isLoading.value = true;
    const newData = await getList(10);
    list.value = [...list.value, ...newData];
  } finally {
    isLoading.value = false;
  }
};

onMounted(async () => {
  await loadMore();
  window.addEventListener("scroll", handleScroll);
});

onUnmounted(() => {
  window.removeEventListener("scroll", handleScroll);
});
</script>

<style scoped>
.list-item {
  width: 100%;
  height: 500px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 3rem;
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  transition: background 0.3s ease;
  will-change: transform;
}

.loading {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 10px 20px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  border-radius: 5px;
}
</style>
