<template>
<img ref="image" />
  <br />
  <input type="file" @change="onFileUpload" />
  <p>{{ message }}</p>
  <canvas ref="canvas" />
</template>

<script>
import { createFFmpeg, fetchFile } from "@ffmpeg/ffmpeg";
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "App",
  setup() {
    // app state
    const ffmpeg = createFFmpeg({
      log: true,
    });
    const message = ref("Click Start to Transcode");
    let canvas = ref(null);
    const image = ref(null)

    async function onFileUpload(e) {
      const file = e.target.files[0];
      message.value = "Loading ffmeg-core.js";
      await ffmpeg.load();
      message.value = "Start transcoding";
      ffmpeg.FS("writeFile", file.name, await fetchFile(file));
      await ffmpeg.run(
        "-i",
        file.name,
        '-filter:v', 'select=eq(n\\,0)', '-frames:v', '1', 'out.png'
      );
      message.value = "Complete transcoding";
      const data = ffmpeg.FS("readFile", "out.png");
      console.log(data.buffer)

      const url = URL.createObjectURL(
        new Blob([data.buffer], { type: "image/png" })
      );
      const handler = () => {
        image.value.removeEventListener('load', handler)
        const context = canvas.value.getContext('2d')
        canvas.value.width = image.value.width
        canvas.value.height = image.value.height
        context.drawImage(image.value, 0, 0)
        console.log(context.getImageData(0, 0, image.value.width, image.value.height))
      }
      image.value.addEventListener('load', handler)
      image.value.src = url
    }
    return {
      canvas,
      image,
      message,
      onFileUpload,
    };
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
