<template>
  <div id="app">
    <m-header></m-header>
    <tab></tab>
    <keep-alive>
      <router-view/>
    </keep-alive>
  </div>
</template>

<script>
import MHeader from 'components/m-header/MHeader';
import Tab from 'components/tab/tab';
export default {
  components: {MHeader, Tab}
}
</script>

<style lang="stylus">

</style>
