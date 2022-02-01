<template>
  <v-app>
    <meet />
  </v-app>
</template>

<script>
import Meet from "./components/Meet.vue";

export default {
  name: "App",
  components: {
    meet: Meet,
  },
};
</script>

