<script setup>
import { computed, onMounted, ref, watch } from "vue";

const stringsArray = ref([]);
const chrs = "abdehkmnpswxzABDEFGHKMNPQRSTWXZ123456789";
const progress = computed(() => {
  return `${(stringsArray.value.length / 100000).toFixed(2)}%`;
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
  if (stringsArray.value.length < limit) generationRandomString(limit);
  else if (stringsArray.value.length < limit + 500) {
    setTimeout(() => {
      generationRandomString(limit + 500);
    }, 100);
  }
};

const fileTxt = ref();
const text = ref();

const readFile = () => {
  let file = fileTxt.value.files[0];
  console.log("fileTxt", file);
  let reader = new FileReader();
  reader.readAsText(file);
  reader.onload = function () {
    console.log(reader.result);
    stringsArray.value = JSON.parse(reader.result);
    console.log("stringsArray.value", stringsArray.value);
  };
};

const downLoad = () => {
  let csvData =
    "data:application/txt;charset=utf-8," +
    encodeURIComponent(JSON.stringify(stringsArray.value));
  var link = document.createElement("a");
  link.setAttribute("download", "strings");
  link.href = csvData;
  link.download = "text.txt";
  document.body.appendChild(link);
  link.click();
  link.remove();
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

window.onbeforeunload = function (event) {
	return confirm();

};


watch(currentSearchVal, (newVal) => {
  console.log("newVal", newVal);
});
</script>

<template>
  <div>
		<h2>Если вы обновите страницу и не сохраните сгенирированные строки, генеировать придется заново(единственный способ котрый я придумла как хранить 10кк строк)</h2>
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
      @click="generationRandomString(stringsArray.length + 500), (stop = false)"
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
    <br />
    <div style="cursor: pointer" @click="downLoad()">
      Сохранить текстовый файл со строками
    </div>
    <br />
    <div @click="readFile()">Взять строки из файла и начать генерацию!</div>
    <br />

    <!-- <div><pre>{{ stringsArray }}</pre> - массив строк</div> -->
    Добавте скаченный ранее файл если есть, если нет просто начните генерацию
    выше
    <input type="file" name="file" id="file" ref="fileTxt" />
  </div>
</template>

<style lang="scss" scoped></style>
