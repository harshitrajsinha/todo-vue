<template>
  <div class="todo-list-container">
    <div class="todo-list__heading">Lists</div>
    <!-- Pass a default icon in case icon is not present -->
    <DisplayItemList
      v-model:list-model="todoListData.value"
      listStyles="todo-list-style"
      @getListId="handleListId"
      ><template #list-icon="{ icon }">
        <span class="todo-list-icon">{{ icon }}</span>
      </template>
      <template #list-count="{ count }">
        <div class="todo-lists__list-count">{{ count }}</div>
      </template>
    </DisplayItemList>
    <div :class="['todo-list__input', { active: toShowTodoInput }]">
      <button @click="toggleTodoEmojiPicker">{{ buttonTxt }}</button
      ><EmojiPicker
        v-if="showTodoEmojiPicker"
        :display-recent="true"
        :native="false"
        @select="onSelectGroupEmoji"
      />
      <input
        type="text"
        maxlength="25"
        v-model="newTodo['title']"
        @input="writeNewTodo"
        @blur="getNewTodo"
      />
    </div>
    <CreateListBtn
      btnText="Create new list"
      class="create-list-btn-style"
      @toShowModal="toggleTodoListInput"
    />
  </div>
</template>

<script setup>
import EmojiPicker from "vue3-emoji-picker";
import "vue3-emoji-picker/css";
import DisplayItemList from "./DisplayItemList.vue";
import CreateListBtn from "./CreateListBtn.vue";
import { ref } from "vue";

const defaultButtonTxt = "😁";
let buttonTxt = ref(defaultButtonTxt);
let timerId = ref(null);
let newTitle = ref(true);
const defaultTodoObj = { icon: buttonTxt.value, title: "", "task-list": [] };
let newTodo = ref(defaultTodoObj);
let showTodoEmojiPicker = ref(false);
let todoListData = defineModel("todo-list-display");
let todoData = defineModel("todo-data");
let lastItem = defineModel("last-todo-item");
let toShowTodoInput = ref(false);
let emit = defineEmits(["todoItem"]);

const handleListId = (id) => {
  emit("todoItem", id);
};

const toggleTodoListInput = () => {
  toShowTodoInput.value = !toShowTodoInput.value;
  if (toShowTodoInput.value) {
    newTodo["title"] = "";
    newTodo.value = { icon: defaultButtonTxt, title: "", "task-list": [] };
    buttonTxt.value = defaultButtonTxt;
    newTitle.value = true;
  }
};

const onSelectGroupEmoji = (emoji) => {
  buttonTxt.value = emoji["i"];
  newTodo.value["icon"] = emoji["i"];
  showTodoEmojiPicker.value = !showTodoEmojiPicker.value;
  //getNewTodo();
};
const toggleTodoEmojiPicker = () => {
  showTodoEmojiPicker.value = !showTodoEmojiPicker.value;
  if (timerId.value !== null) {
    clearTimeout(timerId.value);
    timerId.value = null;
  }
};

const writeNewTodo = () => {
  if (newTodo.value["title"]) {
    if (newTodo.value["title"].length === 1 && newTitle.value) {
      newTitle.value = false;
      todoData.value.push(newTodo.value);
    } else if (newTodo.value["title"].length === 1 && !newTitle.value) {
      lastItem.value["title"] = newTodo.value["title"];
    } else if (newTodo.value["title"].length > 1) {
      lastItem.value["title"] = newTodo.value["title"];
    }
  }
};

const getNewTodo = (event) => {
  if (newTodo.value["title"]) {
    if (
      newTodo.value["icon"] === defaultButtonTxt &&
      !showTodoEmojiPicker.value
    ) {
      timerId.value = setTimeout(() => {
        // If emoji picker is clicked then this will cancel
        lastItem.value = { ...newTodo.value };

        toggleTodoListInput();
        timerId.value = null;
      }, 2000);
    } else {
      lastItem.value = { ...newTodo.value };

      toggleTodoListInput();
    }
  }
};
</script>

<style scoped>
.todo-list-container {
  position: relative;
  background-color: white;
  padding: 2rem;
  padding-left: 1rem;
  margin-top: 1rem;
  margin-bottom: 1rem;
  margin-left: 1rem;
  border-radius: 0.5rem;
}

.todo-list__heading {
  font-size: 1.8rem;
  padding-left: 0.5rem;
  margin-bottom: 1.5rem;
}

::v-deep(li.todo-list-style) {
  padding: 0.5rem;
  margin-bottom: 0.8rem;
  gap: 1rem;
  font-size: 1rem;
  font-weight: 700;
}

::v-deep(li.todo-list-style:hover) {
  background-color: #efecec;
}

.todo-lists__list-count {
  border-radius: 50%;
  padding: 0rem 0.6rem;
  background-color: #e1e1e1;
  margin-left: auto;
}

.todo-list__input {
  display: flex;
  gap: 0.2rem;
  margin: 1rem 0;
  position: relative;
  bottom: -100vh;
  max-height: 0;
  transition: max-height 0.8s ease-in-out;
}

.todo-list__input.active {
  position: relative;
  bottom: 0;
  max-height: 200px;
}

.todo-list__input button {
  border-radius: 0.5rem;
  border: 1px solid black;
  width: 2rem;
  font-size: 1.2rem;
  cursor: pointer;
}

.todo-list__input input {
  border-radius: 0.5rem;
  padding: 0.5rem;
  background-color: #efecec;
  width: -webkit-fill-available;
  border: 1px solid black;
  outline: none;
}

::v-deep(button.create-list-btn-style) {
  width: -webkit-fill-available;
  color: black;
  background: #efecec;
  text-align: left;
  border: 1px solid rgb(96, 96, 96);
  font-weight: 500;
}

.todo-list-icon {
  transform: scale(1.3);
}

.v3-emoji-picker {
  position: absolute;
  top: 2.5rem;
  z-index: 2;
}
</style>
<!-- if (
          event.relatedTarget &&
          !event.relatedTarget.tagName === "BUTTON" &&
          !event.relatedTarget.classList.contains("create-list-btn")
        ) -->
