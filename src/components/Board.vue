<script setup>
import Cell from './Cell.vue'
import Steps from './Steps.vue'
import { ref, onMounted } from 'vue'

function valsOf(str) {
    let r = [];
    for (let i in str) {
        if (str[i]=='0') r.push([1,2,3,4,5,6,7,8,9])
        else r.push(Number.parseInt(str[i]))
    }
    return r;
}

const obj = ref({
  vals: valsOf("000105000140000670080002400063070010900000003010090520007200080026000035000409000"),
  steps: []
})

onMounted(() => {
  console.log(`The initial count is ${obj.value.vals[0]}.`)
})

function rowIndexes(row) {
    let idxs = []
    for (let c=0 ; c<9 ; c++) {
        idxs.push(row * 9 + c);
    }
    return idxs;
}

function colIndexes(col) {
    let idxs = []
    for (let r=0 ; r<9 ; r++) {
        idxs.push(r * 9 + col);
    }
    return idxs;
}

function sqrIndexes(row,col) {
    const y = Math.floor(row / 3);
    const x = Math.floor(col / 3);
    let idxs = []

    for (let r=0 ; r<3 ; r++) {
        for (let c=0 ; c<3 ; c++) {
            idxs.push( y*27 + r*9 + x*3 + c);
        }
    }

    return idxs;
}

function cellIndex(r,c) {
    return r * 9 + c;
}

function checkSolved() {
    let updated = 0
    let updates = []

    for (let i=0 ; i<81 ; i++) {

        if (Array.isArray(obj.value.vals[i])) {

            let num = null
            for (let j=0 ; j<9 ; j++) {
                if (obj.value.vals[i][j] != '') {
                    if (num == null) {
                        num = obj.value.vals[i][j]
                    } else {
                        num = null
                        break
                    }
                }
            }
            if (num != null) {
                obj.value.vals[i] = num
                updated++
                updates.push(i)
            }
        }
    }
    return { name: "Check Solved", updated, updates }
}

function updatePossibilities() {
    let updated = 0
    for (let r=0 ; r<9 ; r++) {
        for (let c=0 ; c<9 ; c++) {
            let idx = cellIndex(r,c)
            if (!Array.isArray(obj.value.vals[idx])) {

                let rowIdxs = rowIndexes(r)
                for (let i=0 ; i<9 ; i++) {
                    if (Array.isArray(obj.value.vals[rowIdxs[i]])) {
                        if (obj.value.vals[rowIdxs[i]][ obj.value.vals[idx] - 1] != '') {
                            obj.value.vals[rowIdxs[i]][ obj.value.vals[idx] - 1] = ''
                            updated++
                        }
                    }
                }

                let colIdxs = colIndexes(c)
                for (let i=0 ; i<9 ; i++) {
                    if (Array.isArray(obj.value.vals[colIdxs[i]])) {
                        if (obj.value.vals[colIdxs[i]][ obj.value.vals[idx] - 1] != '') {
                            obj.value.vals[colIdxs[i]][ obj.value.vals[idx] - 1] = ''
                            updated++
                        }
                    }
                }

                let sqrIdxs = sqrIndexes(r,c)
                for (let i=0 ; i<9 ; i++) {
                    if (Array.isArray(obj.value.vals[sqrIdxs[i]])) {
                        if (obj.value.vals[sqrIdxs[i]][ obj.value.vals[idx] - 1] != '') {
                            obj.value.vals[sqrIdxs[i]][ obj.value.vals[idx] - 1] = ''
                            updated++
                        }
                    }
                }
            }
        }
    }
    return { name: "Update Possibilities", updated, updates: [] }
}

let step = 0;
const steps = [ checkSolved, updatePossibilities ]

function nextStep() {
    obj.value.steps.push(steps[step % steps.length]())
    step++
    console.log(step)
}

</script>

<template>
  <table class="board">
    <tbody>
        <tr v-for="r in 3" :idx="r">
            <td v-for="c in 3" :idx="c">
                <table>
                    <tbody>
                        <tr v-for="rr in 3" :idx="rr">
                            <td v-for="cc in 3" :idx="cc">
                                <Cell v-bind:x="obj.vals[ ((r-1) * 3 + (rr-1)) * 9 + (c-1)*3 + (cc-1)]" v-bind:u="obj.steps.length == 0 ? false : obj.steps[obj.steps.length-1].updates.includes(((r-1) * 3 + (rr-1)) * 9 + (c-1)*3 + (cc-1))"/>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </td>
        </tr>
    </tbody>
  </table>
  <div>
    <button @click="nextStep">Next Step</button>
    <Steps v-bind:steps="obj.steps" />
  </div>
</template>

<style scoped>

table {
    border-collapse: collapse;
    border-spacing: 0;
}

td {
    padding: 0px;
}
table.board {
    border: 2px solid black;
}

</style>