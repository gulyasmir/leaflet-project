<template>
  <div class="upload-block">

    <div class="border-gray" @dragover="dragover" @dragleave="dragleave" @drop="drop">
      <input type="file" multiple name="fields[assetsFieldHandle][]" id="assetsFieldHandle"
        class="w-px h-px opacity-0 overflow-hidden absolute" @change="onChange" ref="file"
        accept=".pdf,.jpg,.jpeg,.png,.svg" />

      <label for="assetsFieldHandle" class="block cursor-pointer">
        <div class="upload-files">
          <img src="/images/icon-to-download.jpg" alt="">
          <p> Перетащите файлы или
            <span class="underline">нажмите сюда </span> для выбора файла
          </p>
        </div>
      </label>
      <ul class="mt-4" v-if="this.filelist.length" v-cloak>
        <li class="text-sm p-1" v-for="file in filelist">
          <span>{{ file.name }} </span>
          <button class="upload-button button" type="button" @click="remove(filelist.indexOf(file))" title="Remove file">
            Удалить
          </button>
        </li>
      </ul>

      <a class="submit-button" v-on:click="submitFiles()" v-show="filelist.length > 0">{{ buttonText }}</a>
      <h2> {{ resultText }}</h2>
    </div>

  </div>
</template>

<script>
import * as axios from 'axios';
export default {
  data() {
    return {
      filelist: [],
      dragAndDropCapable: false,
      files: [],
      uploadPercentage: 0,
      buttonText: 'Отправить',
      resultText: ''
    };
  },
  methods: {
    onChange() {
      this.filelist = [...this.$refs.file.files];
    },
    remove(i) {
      this.filelist.splice(i, 1);
    },
    dragover(event) {
      event.preventDefault();
      // Add some visual fluff to show the user can drop its files
      if (!event.currentTarget.classList.contains("bg-green-300")) {
        event.currentTarget.classList.remove("bg-gray-100");
        event.currentTarget.classList.add("bg-green-300");
      }
    },
    dragleave(event) {
      // Clean up
      event.currentTarget.classList.add("bg-gray-100");
      event.currentTarget.classList.remove("bg-green-300");
    },
    drop(event) {
      event.preventDefault();
      this.$refs.file.files = event.dataTransfer.files;
      this.onChange(); // Trigger the onChange event manually
      // Clean up
      event.currentTarget.classList.add("bg-gray-100");
      event.currentTarget.classList.remove("bg-green-300");
    },
    submitFiles() {
      this.buttonText = 'Идет загрузка'
      let formData = new FormData();

      for (var i = 0; i < this.filelist.length; i++) {
        let file = this.filelist[i];

        formData.append("filelist[" + i + "]", file);
      }
      axios
        .post("https://server-leaflet.herokuapp.com/fileupload", formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        })
        .then(function (res) {
         
          // console.log('res', res)
          //console.log("SUCCESS!!");
        })
        .catch(function (err) {
         
          // console.log("FAILURE!!", err);
        });

      setTimeout(() => {
        this.filelist = []
        this.resultText = 'Файлы успешно загружены!'

      }, 2000)
    },
  },
};
</script>

<style>
[v-cloak] {
  display: none;
}

.upload-files img {
  height: 100px;
}

a.submit-button {
  display: block;
  margin: auto;
  text-align: center;
  width: 200px;
  padding: 10px;
  text-transform: uppercase;
  background-color: #46a03a;
  color: white;
  font-weight: bold;
  margin-top: 20px;
  border-radius: 3px;
  cursor: pointer;
}

.submit-button:hover {
  border-radius: 1px;
}

ul {
  padding: 0;
}

ul li {
  list-style: none;
  padding: 5px;
  width: 80%;
  display: flex;
  justify-content: space-around;
}

.upload-block {
  width: 40%;
  margin: 5% 30%;
  text-align: center;
}

.opacity-0 {
  opacity: 0;
}

.underline {
  border-bottom: 1px solid #888;
}

.border-gray {
  border: 1px solid #888;
  background-color: antiquewhite;
  padding: 15px 50px;
}

.upload-button {
  border-radius: 3px;
  border: 1px solid #cf4b4b;
  background: #d45a5a;
  color: #fff;
  padding: 3px 10px;
}

.upload-button:hover {
  border-radius: 1px;
}
</style>
