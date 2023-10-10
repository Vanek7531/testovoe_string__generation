<script setup>
import { computed, onMounted, ref } from "vue";

const stringsArray = ref([]);
const chrs = "abdehkmnpswxzABDEFGHKMNPQRSTWXZ123456789";
const progress = computed(() => {
  return `${(stringsArray.value.length / 100000).toFixed(3)}%`;
});

const stop = ref(false);

const generationRandomString = (limit) => {
  if (stringsArray.value.length > limit || stop.value) return;
  let str = "";
  for (var i = 0; i < 100; i++) {
    var pos = Math.floor(Math.random() * chrs.length);
    str += chrs.substring(pos, pos + 1);
  }
  stringsArray.value.push(str);
  localStorage.setItem("stringsArray", JSON.stringify(stringsArray.value));
  if (stringsArray.value.length < limit) generationRandomString(limit);
  else if (stringsArray.value.length < limit + 50) {
    setTimeout(() => {
      generationRandomString(limit + 50);
    }, 100);
  }
};

const currentSearchVal = ref("");
const searchResults = ref([]);

const searchRes = (serachPagination) => {
  searchResults.value = [];
  stringsArray.value.find((el, idx) => {
    if (
      el.includes(currentSearchVal.value) &&
      searchResults.value.length < serachPagination * 5
    )
      searchResults.value.push(el);
  });
  if (searchResults.value.length === 0)
    return searchResults.value.push("ничего не найденно");
  searchResults.value = searchResults.value.slice(
    serachPagination * 5 - 5,
    serachPagination * 5
  );
  return searchResults.value;
};

onMounted(() => {
  stringsArray.value = localStorage.getItem("stringsArray")
    ? JSON.parse(localStorage.getItem("stringsArray"))
    : [];

  if (stringsArray.value.length > 0)
    generationRandomString(stringsArray.value.length + 50);
  else generationRandomString(50);
});
</script>

<template>
  <div>
    <div style="display: flex; gap: 10px; flex-direction: column">
      <div style="display: flex; gap: 10px">
        <input type="text" v-model="currentSearchVal" />
        <div style="cursor: pointer" @click="searchRes(1)">Поиск!</div>
      </div>
      <pre v-for="(item, idx) in searchResults" :key="item"
        >{{ idx + 1 }}: {{ item }}</pre
      >
      <div style="display: flex; gap: 10px">
        <div v-for="(item, idx) in 5" :key="item">
          <div style="cursor: pointer" @click="searchRes(idx + 1)">
            {{ idx + 1 }}стр
          </div>
        </div>
      </div>
    </div>
    <br />
    <p>Прогресс</p>
    <div>прогресс: {{ progress }}</div>
    <br />
    <div
      @click="generationRandomString(stringsArray.length + 50), (stop = false)"
      style="
        cursor: pointer;
        padding: 10px;
        border: 1px solid rgb(255, 255, 255);
      "
    >
      Сгенирировать
    </div>
    <br />
    <div
      style="
        cursor: pointer;
        padding: 10px;
        border: 1px solid rgb(255, 255, 255);
      "
      @click="stop = true"
    >
      STOP
    </div>
  </div>
</template>

<style lang="scss" scoped></style>
