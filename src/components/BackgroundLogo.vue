<template>
  <div class="background-logo"></div>
</template>

<script>
export default {
  name: "BackgroundLogo",
};
</script>

<style scoped>
.background-logo {
  background-image: url('https://images.evetech.net/Corporation/98057377_512.png');
  background-repeat: no-repeat;
  background-position: center;
  background-size: 30%;
  opacity: 0.1;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: -1;
}
</style>