<template>
  <div class="wrapper">
    <header>
      <!-- кнопка назад второй страницы -->
      <h2 v-show="showH2">Notepad</h2>
      <div class="button-container" v-show="showNotePage">
        <button @click.prevent @click="backToFirstPage(item)">back</button>
        <button @click.prevent @click="saveChanges">Save</button>
      </div>
      <h2 v-show="showNotePage" class="heading-notes-page">{{ item.name }}</h2>
      <form v-show="showForm" @submit.stop>
        <label>
          <input
            type="text"
            placeholder="Заголовок*"
            name="note-name"
            required
            v-model="inpValueName"
          />
        </label>
        <textarea
          rows="3"
          type="text"
          name="text"
          placeholder="Текст записи*"
          required
          v-model="inpValueText"
        >
        </textarea>
        <button type="button" @click="addNote"></button>
      </form>
      <div v-show="show" class="add-note">
        <button @click="showInput"></button>
      </div>
    </header>
    <!-- контейнер с видом второй страницы записи -->
    <div v-show="showNotePage" class="container page-2">
      <form action="">
        <ul>
          <p>Заголовок</p>
          <textarea v-model="item.name"></textarea>
          <p>Описание</p>
          <textarea v-model="item.text"></textarea>
          <p class="note-date">создан: {{ item.createDate }}</p>
          <p class="note-date">ред: {{ item.changesDate }}</p>
        </ul>
      </form>
    </div>
    <!-- контейнер с видом главной страницы записей -->
    <div class="container page-1" v-show="showFirstPage">
      <h4 v-if="notes.length === 0">Empty list</h4>
      <ul class="notes-items">
        <li v-for="note in pagnation()" v-bind:key="note">
          <div class="container item">
            <div class="name">
              <a href="" @click.stop.prevent @click="moveToNotePage(note)"
                >{{ note.name }}
              </a>
            </div>
          </div>
          <div class="information">
            <ul class="note-info">
              <li>{{ note.text }}</li>
              <li class="note-date">создан: {{ note.createDate }}</li>
              <li class="note-date">ред: {{ note.changesDate }}</li>
            </ul>
          </div>
          <div class="delete-note" @click="deleteNote(note)">
            <button>del</button>
          </div>
        </li>
      </ul>
    </div>
    <footer>
      <div v-if="!showNotePage" class="page-buttons">
        <button v-if="page > 1" @click="page = page - 1">prev</button>
        <button v-if="notes.length > 6" @click="page = page + 1">next</button>
      </div>
    </footer>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      notes: [],
      item: {},
      page: 1,
      inpValueName: null,
      inpValueText: null,
      showForm: false,
      showFirstPage: true,
      showNotePage: false,
      showH2: true,
      show: true,
      postFullDate: null,
      // индекс эл. который при изменении будет добавляется на место старого эл. в масиве всех эл.
      index: null,
    };
  },
  created() {
    //читаем данные с localStorage
    const itemData = localStorage.getItem("note-list");
    // если есть уже созданные записи в localStorage добавляем их в список
    if (itemData) {
      this.notes = JSON.parse(itemData);
    }
  },
  methods: {
    getDate() {
      let date = new Date();
      let year = date.getFullYear();
      let month = date.getMonth() + 1;
      let day = date.getDay();
      let hours = date.getHours();
      let minutes = date.getMinutes();
      if (minutes < 10) {
        minutes = "0" + minutes;
      }
      if (month < 10) {
        month = "0" + month;
      }
      if (day < 10) {
        day = "0" + (day + 1);
      }
      this.postFullDate = (hours + ":" + minutes + " " + day + "." + month + "." + year);
    },
    pagnation() {
      // пагинация и отображение 6 эл. на странице
      const start = (this.page - 1) * 6;
      const end = this.page * 6;
      return this.notes.slice(start, end);
    },
    // добавление новой записи
    addNote() {
      if (this.inpValueName && this.inpValueText) {
        // получаем дату
        this.getDate();
        // добавляем в масив элементов объект со значениями
        this.notes.unshift({
          name: this.inpValueName,
          text: this.inpValueText,
          createDate: this.postFullDate,
          changesDate: null,
        });
        //записываем список записей в localStorage
        localStorage.setItem("note-list", JSON.stringify(this.notes));
        this.inpValueName = null;
        this.inpValueText = null;
        this.showForm = false;
        this.show = true;
      }
    },
    showInput() {
      if (!this.showForm) {
        this.showForm = true;
        this.show = false;
      }
    },
    deleteNote(noteToDelete) {
      this.notes = this.notes.filter((note) => note !== noteToDelete);
      // удаляем из localStorage контакт
      localStorage.setItem("note-list", JSON.stringify(this.notes));
    },
    moveToNotePage(value) {
      this.item.name = value.name;
      this.item.text = value.text;
      this.item.createDate = value.createDate;
      this.item.changesDate = value.changesDate;
      // скрываем первую страницу
      this.showFirstPage = false;
      // скрываем заголовок
      this.showH2 = false;
      // скрываем форму
      this.showForm = false;
      this.show = false;
      // показываем вторую страницу
      this.showNotePage = true;
      // получаем индекс объекта к которому переходим
      this.index = this.notes.indexOf(value);
    },
    backToFirstPage() {
      this.showFirstPage = true;
      this.showNotePage = false;
      this.show = true;
      this.showH2 = true;
      this.item = {};
    },
    saveChanges() {
      // валидация на пустые строки
      if (this.item.name.trim() && this.item.text.trim()) {
        // изменяем масив объектов и добавляем в него охмененный объект
        this.notes.splice(this.index, 1, this.item);
        this.getDate();
        this.item.changesDate = this.postFullDate;
        // перезаписываем LocalStorege
        localStorage.setItem("note-list", JSON.stringify(this.notes));
      }
    },
  },
};
</script>
<style src="./app.css"></style>
