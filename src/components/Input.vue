<template>
  <div class="input">
    <div style="margin: 1em;">
      <h3 style="color: white">INPUT</h3>
      <input id="audio-file" type="file" accept=".mp3" />
    </div>
    <audio style="opacity: 0" id="audio-player" controls>
      <source src="" id="src" />
    </audio>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  mounted() {
    const input = document.getElementById("audio-file");
    input && input.addEventListener("change", this.uploadAudio, false);
  },
  methods: {
    uploadAudio(event) {
      const files = event.target.files;
      const player = document.getElementById("audio-player");
      const fileUrl = URL.createObjectURL(files[0]);
      player.src = fileUrl;
      player.play();
      this.$emit("audio-upload");
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.input {
  position: absolute;
  width: 10vw;
  height: 10vh;
  display: flex;
  align-content: center;
}
</style>
