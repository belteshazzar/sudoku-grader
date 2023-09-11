<script setup>

defineProps(['x','u'])
//import { ref, onMounted } from 'vue'


// onMounted(() => {
//   console.log(`The initial count is ${obj.value.vals[0]}.`)
// })

</script>

<template>
  <table v-if=" ! Array.isArray(x) ">
    <tr>
      <td class="solution" :class="{ updated: u }">{{  x  }}</td>
    </tr>
  </table>
  <table v-if=" Array.isArray(x) ">
    <tbody>
        <tr v-for="r in 3" :idx="r">
            <td class="note" v-for="c in 3" :idx="c">
                {{ x[ (r-1) * 3 + (c-1)] }}
            </td>
        </tr>
    </tbody>
  </table>
</template>

<style scoped>

table {
  border-collapse: collapse;
  border-spacing: 0;
  border: 1px solid black;
  width: 60px;
  height: 60px;
}

td.solution {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-size: 20pt;
    text-align: center;
    vertical-align: middle;
    line-height: 0%;
}

td.updated {
  background-color: aquamarine;
}
td.note {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-size: 12pt;
    text-align: center;
    vertical-align: middle;
    line-height: 0%;
}

</style>