<template>
  <div class="home">
    <mind msg="Welcome to Your Vue.js App"/>
 
  </div>
</template>

<script>
// @ is an alias to /src
import mind from '@/components/mind.vue'

export default {
  name: 'Home',
  components: {
    mind
  }
}
</script>
