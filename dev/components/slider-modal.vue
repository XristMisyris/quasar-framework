<template>
  <quasar-slider arrows dots class="text-white full-height">
    <div slot="slide" class="bg-primary centered">
      <div>Slide 1</div>
      <button class="dark glossy" @click="close()">Close Me</button>
    </div>
    <div slot="slide" class="bg-secondary centered">
      <div>Slide 2</div>
      <button class="dark glossy" @click="close()">Close Me</button>
    </div>
    <div slot="slide" class="bg-tertiary centered">
      <div>Slide 3</div>
      <button class="dark glossy" @click="close()">Close Me</button>
    </div>
  </quasar-slider>
</template>

<script>
export default {}
</script>
