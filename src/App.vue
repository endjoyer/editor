<template>
  <div class="app">
    <div class="buttons">
      <button class="btn btn_undo" @click="undo"></button>
      <button class="btn btn_redo" @click="redo"></button>
      <button class="btn btn_header" @click="makeHeader"></button>
      <button class="btn btn_paragraph" @click="makeParagraph"></button>
      <button class="btn btn_modal" @click="openModal"></button>
      <button class="btn btn_html" @click="copyHtml">Скопировать HTML</button>
    </div>
    <div
      contenteditable="true"
      ref="editor"
      @input="updateHistory"
      class="editor"
    ></div>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <input
          v-model="imageUrl"
          @keyup.enter="insertImage"
          placeholder="Введите URL изображения"
        />
        <button class="btn" :disabled="!isValidUrl" @click="insertImage">
          Добавить
        </button>
        <p v-if="!isValidUrl" class="error">URL изображения не валиден</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      history: [],
      historyIndex: -1,
      showModal: false,
      imageUrl: '',
      savedRange: null,
    };
  },
  computed: {
    isValidUrl() {
      const urlPattern = /^(http|https):\/\/[^ "]+$/;
      return urlPattern.test(this.imageUrl);
    },
  },
  methods: {
    updateHistory() {
      this.history.push(this.$refs.editor.innerHTML);
      this.historyIndex++;
    },
    undo() {
      if (this.historyIndex > 0) {
        this.historyIndex--;
        this.$refs.editor.innerHTML = this.history[this.historyIndex];
      }
    },
    redo() {
      if (this.historyIndex < this.history.length - 1) {
        this.historyIndex++;
        this.$refs.editor.innerHTML = this.history[this.historyIndex];
      }
    },
    makeHeader() {
      document.execCommand('formatBlock', false, 'h1');
    },
    makeParagraph() {
      document.execCommand('formatBlock', false, 'p');
    },
    openModal() {
      this.showModal = true;
      this.savedRange = this.saveRange();
    },
    closeModal() {
      this.showModal = false;
    },
    insertImage() {
      if (this.isValidUrl) {
        this.restoreRange(this.savedRange);
        document.execCommand(
          'insertHTML',
          false,
          '<img src="' + this.imageUrl + '">'
        );
        this.imageUrl = '';
        this.closeModal();
      }
    },
    copyHtml() {
      try {
        navigator.clipboard.writeText(this.$refs.editor.innerHTML);
      } catch (err) {
        console.error('Failed to copy HTML: ', err);
      }
    },
    saveRange() {
      const sel = window.getSelection();
      if (sel.rangeCount > 0) {
        return sel.getRangeAt(0);
      }
      return null;
    },
    restoreRange(range) {
      const sel = window.getSelection();
      sel.removeAllRanges();
      sel.addRange(range);
    },
  },
};
</script>

<style scoped>
@import './assets/normalize.css';
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@100;300;400;500;700;900&display=swap');

.app {
  font-family: 'Roboto', Arial, sans-serif;
  display: flex;
  flex-direction: column;
  background-color: #1e1e1e;
  color: #eaeaea;
  height: 100vh;
  width: 100vw;
  padding: 77px 107px 107px 107px;
  box-sizing: border-box;
  overflow: auto;
}

.buttons {
  display: flex;
  gap: 12px;
  align-items: center;
}

.btn {
  width: 42px;
  height: 38px;
  border: none;
  outline: none;
  border-radius: 4px;
  color: #639eff;
  font-family: Roboto;
  font-size: 15px;
  font-style: normal;
  font-weight: 400;
  line-height: 23px;
  background: #1e1e1e;
  cursor: pointer;
}

.btn:hover {
  opacity: 0.8;
}

.btn_undo {
  background: url('./assets/img/undo.svg') 0 0 no-repeat;
}
.btn_redo {
  background: url('./assets/img/redo.svg') 0 0 no-repeat;
}
.btn_header {
  background: url('./assets/img/header.svg') 0 0 no-repeat;
}
.btn_paragraph {
  background: url('./assets/img/paragraph.svg') 0 0 no-repeat;
}
.btn_modal {
  background: url('./assets/img/modal.svg') 0 0 no-repeat;
}
.btn_html {
  width: fit-content;
  height: fit-content;
}
.editor {
  width: 100%;
  flex-grow: 1;
  overflow: visible;
  outline: none;
}

.editor > img {
  max-width: 100%;
  max-height: 100%;
}

.editor > h1 {
  font-size: 31px;
  font-weight: 400;
  line-height: 23px;
}

.editor > p {
  font-size: 15px;
  font-weight: 400;
  line-height: 23px;
  height: 100%;
  margin: 0;
}

.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
}

.modal-content {
  background-color: #282828;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
}

.close {
  color: #eaeaea;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  text-decoration: none;
  cursor: pointer;
  opacity: 0.8;
}

.error {
  color: red;
}
</style>
