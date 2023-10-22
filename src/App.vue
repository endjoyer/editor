<template>
  <div class="panel">
    <div class="buttons">
      <button class="btn btn_undo" @click="undo"></button>
      <button class="btn btn_redo" @click="redo"></button>
      <button class="btn btn_header" @click="makeHeader"></button>
      <button class="btn btn_paragraph" @click="makeParagraph"></button>
      <button class="btn btn_modal" @click="openModal"></button>
      <button class="btn btn_text" @click="copyHtml">Скопировать HTML</button>
    </div>
    <div class="buttons">
      <button class="btn btn_text" @click="saveToCache">Сохранить</button>
      <button class="btn btn_text" @click="clearCache">Очистить</button>
    </div>
  </div>
  <div
    contenteditable="true"
    ref="editor"
    @input="updateHistory"
    @keydown.tab.prevent="insertTab"
    class="editor"
  ></div>
  <div v-if="showModal" class="modal">
    <div class="modal-content">
      <span class="close" @click="closeModal">&times;</span>
      <p class="modal__title">Введите URL изображения</p>
      <label class="modal__label">
        <input
          class="modal__input"
          v-model="imageUrl"
          @keyup.enter="insertImage"
        />
        <span v-if="imageUrl && !isValidUrl" class="error">
          URL изображения не валиден
        </span>
      </label>
      <button class="submit" :disabled="!isValidUrl" @click="insertImage">
        Добавить
      </button>
    </div>
  </div>
</template>

<script>
import interact from 'interactjs';
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
      if (this.$refs.editor.lastChild) {
        if (this.$refs.editor.lastChild.nodeName !== 'DIV') {
          this.history.push(this.$refs.editor.innerHTML);
          this.historyIndex++;
        }
      }
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
        if (this.savedRange) {
          this.restoreRange(this.savedRange);
        }
        document.execCommand(
          'insertHTML',
          false,
          '<img class="resizable" src="' + this.imageUrl + '">'
        );
        this.imageUrl = '';
        this.closeModal();
        this.initResizable();
      }
    },
    initResizable() {
      this.$nextTick(() => {
        interact('.resizable')
          .resizable({
            edges: { left: false, right: true, bottom: true, top: false },
          })
          .on('resizemove', function (event) {
            var target = event.target;

            target.style.width = event.rect.width + 'px';
            target.style.height = event.rect.height + 'px';

            if (event.deltaRect.left !== 0) {
              target.style.left = event.rect.left + 'px';
            }
            if (event.deltaRect.top !== 0) {
              target.style.top = event.rect.top + 'px';
            }
          });
      });
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
    insertTab() {
      document.execCommand('insertHTML', false, '&nbsp;&nbsp;&nbsp;&nbsp;');
    },
    saveToCache() {
      localStorage.setItem('editorContent', this.$refs.editor.innerHTML);
    },
    clearCache() {
      localStorage.removeItem('editorContent');
      this.$refs.editor.innerHTML = '';
    },
  },
  mounted() {
    if (localStorage.getItem('editorContent')) {
      this.$refs.editor.innerHTML = localStorage.getItem('editorContent');
    }
  },
};
</script>

<style>
@import './assets/normalize.css';
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400&display=swap');

#app {
  background: #1e1e1e;
  font-family: 'Roboto', Arial, sans-serif;
  font-style: normal;
  font-weight: 400;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  background-color: #1e1e1e;
  color: #eaeaea;
  min-height: 100vh;

  padding: 125px 100px 100px 100px;
  box-sizing: border-box;
  overflow: auto;
  margin: 0 auto;
}

/* .app {
  font-family: 'Roboto', Arial, sans-serif;
  font-style: normal;
  font-weight: 400;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  background-color: #1e1e1e;
  color: #eaeaea;
  height: 100vh;
  max-width: 1080px;
  padding: 125px 100px 100px 100px;
  box-sizing: border-box;
  overflow: auto;
  margin: 0 auto;
} */

.panel {
  position: fixed;
  top: 55px;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #1c1c1c;
  border-radius: 10px;
}

.buttons {
  display: flex;
  gap: 12px;
  align-items: center;
  padding: 10px;
}

.btn {
  width: 42px;
  height: 38px;
  border: none;
  outline: none;
  border-radius: 4px;
  color: #639eff;
  font-size: 15px;
  line-height: 23px;
  background: #1e1e1e;
  cursor: pointer;
  transition: all 0.3s ease 0s;
}

.btn:hover {
  opacity: 0.8;
}

.btn:active {
  opacity: 1;
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
.btn_text {
  width: max-content;
  height: fit-content;
  font-size: 15px;
  line-height: 23px;
}
.editor {
  box-sizing: border-box;
  width: 100%;
  flex-grow: 1;
  overflow: visible;
  outline: none;
  font-size: 15px;
  line-height: 23px;
  background: rgba(184, 167, 83, 0.4) 0 0 no-repeat;
  padding: 30px;
  max-width: 1080px;
  margin: 0 auto;
  border-radius: 10px;
}

.editor img {
  max-width: 100%;
  max-height: 100%;
}

.editor h1 {
  font-size: 31px;
  line-height: 35px;
  margin: 15px 0;
}

.editor p {
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
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-sizing: border-box;
  border-radius: 20px;
  background-color: #282828;
  width: 400px;
  min-height: 200px;
  margin: auto;
  padding: 20px;
  position: relative;
}

.modal__title {
  font-size: 19px;
  line-height: 26px;
  margin: 10px 0 15px 0;
}

.modal__label {
  display: block;
  min-height: 60px;
}

.modal__input {
  color: #eaeaea;
  background-color: #282828;
  display: block;
  width: 100%;
  font-size: 16px;
  line-height: 26px;
  border: none;
  outline: none;
  border-bottom: 1px solid rgba(213, 213, 213, 0.133);
}
.modal__input:focus {
  border-bottom: 1px solid rgba(217, 217, 217, 0.227);
}

.error {
  margin: 15px 0 0 0;
  color: rgb(191, 20, 20);
  font-size: 13px;
  line-height: 18px;
}

.submit {
  margin: 0;
  padding: 10px 30px;
  font-size: 18px;
  line-height: 26px;
  border: none;
  outline: none;
  border-radius: 7px;
  color: #639eff;
  background: #1e1e1e;
  cursor: pointer;
  transition: all 0.25s ease 0s;
}

.submit:hover {
  opacity: 0.8;
}

.close {
  position: absolute;
  right: -25px;
  top: -25px;
  color: #eaeaea;
  float: right;
  font-size: 35px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  text-decoration: none;
  cursor: pointer;
  opacity: 0.8;
}

.close:active {
  opacity: 1;
}

@media (max-width: 600px) {
  .app {
    padding: 125px 20px 20px 20px;
  }

  .btn_text {
    width: min-content;
    font-size: 13px;
    line-height: 17px;
  }
}
</style>
