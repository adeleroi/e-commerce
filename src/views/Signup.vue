<template>
  <div class="signup-container">
    <credentials :login="true"/>
  </div>
</template>

<script>
import credentials from "../components/credentials"
export default {
    components:{
        credentials,
    }

}
</script>

<style>

</style>