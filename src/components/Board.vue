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
//  vals: valsOf("000105000140000670080002400063070010900000003010090520007200080026000035000409000"),
//    vals: valsOf("070408029002000004854020007008374200020000000003261700000093612200000403130642070"), // nake triple row
    vals: valsOf("000000000904607000076804100309701080008000300050308702007502610000403208000000000"), // hidden pair
//    vals: valsOf('000030086000020040090078520371856294900142375400397618200703859039205467700904132'), // naked quad square
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

function isCompleted() {
    for (let i=0 ; i<81 ; i++) {
        if (Array.isArray(obj.value.vals[i])) {
            return false;
        }
    }
    return true;
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

function cellContainsNote(cell,v) {
    return Array.isArray(cell) && cell[v-1] == v
}

function hiddenSinglesRows() {
    let updated = 0
    let updates = []

    for (let r=0 ; r<9 ; r++) {
        let rowIdxs = rowIndexes(r)
        for (let i=1 ; i<=9 ; i++) {
            let count = 0
            let atC = -1
            for (let c=0 ; c<rowIdxs.length ; c++) {
                let cell = obj.value.vals[rowIdxs[c]]
                if (cellContainsNote(cell,i)) {
                    count++
                    atC = c
                }
            }
            if (count==1) {
                obj.value.vals[rowIdxs[atC]] = i
                updated++
                updates.push(rowIdxs[atC])
            }
        }
    }

    return { name: "Hidden Singles - Rows", updated: updated!=0, updates }
}

function hiddenSinglesCols() {
    let updated = 0
    let updates = []

    for (let c=0 ; c<9 ; c++) {
        let colIdxs = colIndexes(c)
        for (let i=1 ; i<=9 ; i++) {
            let count = 0
            let atR = -1
            for (let r=0 ; r<colIdxs.length ; r++) {
                let cell = obj.value.vals[colIdxs[r]]
                if (cellContainsNote(cell,i)) {
                    count++
                    atR = r
                }
            }
            if (count==1) {
                obj.value.vals[colIdxs[atR]] = i
                updated++
                updates.push(colIdxs[atR])
            }
        }
    }

    return { name: "Hidden Singles - Columns", updated: updated!=0, updates }
}

function hiddenSinglesSqrs() {
    let updated = 0
    let updates = []

    for (let c=0 ; c<9 ; c+=3) {
        for (let r=0 ; r<9 ; r+=3) {
            let sqrIdxs = sqrIndexes(r,c)
            for (let i=1 ; i<=9 ; i++) {
                let count = 0
                let atX = -1
                for (let x=0 ; x<sqrIdxs.length ; x++) {
                    let cell = obj.value.vals[sqrIdxs[x]]
                    if (cellContainsNote(cell,i)) {
                        count++
                        atX = x
                    }
                }
                if (count==1) {
                    obj.value.vals[sqrIdxs[atX]] = i
                    updated++
                    updates.push(sqrIdxs[atX])
                }
            }
        }
    }

    return { name: "Hidden Singles - Squares", updated: updated!=0, updates }
}

function cellContainsTwoNotes(cell) {
    if (Array.isArray(cell)) {
        let r = []
        for (let i=0 ; i<cell.length ; i++) {
            if (cell[i]!='') {
                r.push(cell[i])
            }
        }
        if (r.length==2) {
            return r
        }
        return false
    } else {
        return false
    }
}

function checkPairs(ps) {
    
    let res = []
    while (ps.length>1) {
        let p0 = ps.shift()
        for (let i=0 ; i<ps.length ; i++) {
            if (p0.p[0]==ps[i].p[0] && p0.p[1]==ps[i].p[1]) {
                res.push([p0,ps[i]])
            }
        }
    }
    if (res.length>0) return res
    return false
}

function nakedPairsRows() {
    let updated = 0
    let updates = []

    for (let r=0 ; r<9 ; r++) {
        let rowIdxs = rowIndexes(r)
        let pairs = []
        for (let c=0 ; c<rowIdxs.length ; c++) {
            let cell = obj.value.vals[rowIdxs[c]]
            let ns = cellContainsTwoNotes(cell)
            if (ns && ns.length==2) {
                pairs.push({at:rowIdxs[c],p:ns})
            }
        }
        let cp = checkPairs(pairs)
        while (cp.length>0) {
            let p = cp.shift()
            for (let c=0 ; c<rowIdxs.length ; c++) {
                if (rowIdxs[c]==p[0].at || rowIdxs[c]==p[1].at) continue
                let cell = obj.value.vals[rowIdxs[c]]
                if (!Array.isArray(cell)) continue
                for (let x=0 ; x<9 ; x++) {
                    if (cell[x]==p[0].p[0] || cell[x]==p[0].p[1]) {
                        cell[x]=''
                        updated++
                        updates.push(rowIdxs[c])
                    }
                }
            }
        }
    }

    return { name: "Naked Pairs - Rows", updated: updated!=0, updates }
}

function nakedPairsCols() {
    let updated = 0
    let updates = []

    for (let c=0 ; c<9 ; c++) {
        let colIdxs = colIndexes(c)
        let pairs = []
        for (let r=0 ; r<colIdxs.length ; r++) {
            let cell = obj.value.vals[colIdxs[r]]
            let ns = cellContainsTwoNotes(cell)
            if (ns && ns.length==2) {
                pairs.push({at:colIdxs[r],p:ns})
            }
        }
        let cp = checkPairs(pairs)
        while (cp.length>0) {
            let p = cp.shift()
            for (let r=0 ; r<colIdxs.length ; r++) {
                if (colIdxs[r]==p[0].at || colIdxs[r]==p[1].at) continue
                let cell = obj.value.vals[colIdxs[r]]
                if (!Array.isArray(cell)) continue
                for (let x=0 ; x<9 ; x++) {
                    if (cell[x]==p[0].p[0] || cell[x]==p[0].p[1]) {
                        cell[x]=''
                        updated++
                        updates.push(colIdxs[r])
                    }
                }
            }
        }
    }

    return { name: "Naked Pairs - Columns", updated: updated!=0, updates }
}

function nakedPairsSqrs() {
    let updated = 0
    let updates = []

    for (let r=0 ; r<9 ; r+=3) {
        for (let c=0 ; c<9 ; c+=3) {
            let sqrIdxs = sqrIndexes(r,c)

            let pairs = []
            for (let s=0 ; s<sqrIdxs.length ; s++) {
                let cell = obj.value.vals[sqrIdxs[s]]
                console.log(`${r},${c},${sqrIdxs[s]},${cell}`)
                let ns = cellContainsTwoNotes(cell)
                if (ns && ns.length==2) {
                    pairs.push({at:sqrIdxs[s],p:ns})
                    console.log(`two notes at rc=${r},${c}, notes=${ns}`)
                }
            }
            let cp = checkPairs(pairs)
            while (cp.length>0) {
                console.log('sqr pairs',cp);
                let p = cp.shift()
                for (let s=0 ; s<sqrIdxs.length ; s++) {
                    if (sqrIdxs[s]==p[0].at || sqrIdxs[s]==p[1].at) continue
                    let cell = obj.value.vals[sqrIdxs[s]]
                    if (!Array.isArray(cell)) continue
                    for (let x=0 ; x<9 ; x++) {
                        if (cell[x]==p[0].p[0] || cell[x]==p[0].p[1]) {
                            console.log(cell)
                            cell[x]=''
                            updated++
                            updates.push(sqrIdxs[s])
                        }
                    }
                }
            }
        }
    }
    return { name: "Naked Pairs - Squares", updated: updated!=0, updates }
}

function numberIndexes(idxs) {

    let r = {};
    for (let i=1 ; i<=9 ; i++) {
        r[i] = [];
    }
    for (let i=1 ; i<=9 ; i++) {
        for (let x=0 ; x<idxs.length ; x++) {
            if (cellContainsNote(obj.value.vals[idxs[x]],i)) {
                r[i].push(idxs[x])
            }
        }
    }
    console.log(r)
    return r;
}
function combinations(items, numSubItems) {
    var result = [];
    var indexes = new Array(numSubItems);
    for (var i = 0 ; i < numSubItems; i++) {
        indexes[i] = i;
    }
    while (indexes[0] < (items.length - numSubItems + 1)) {
        var v = [];
        for (var i = 0 ; i < numSubItems; i++) {
            v.push(items[indexes[i]]);
        }
        result.push(v);
        indexes[numSubItems - 1]++;
        var l = numSubItems - 1; // reference always is the last position at beginning
        while ( (indexes[numSubItems - 1] >= items.length) && (indexes[0] < items.length - numSubItems + 1)) {
            l--; // the last position is reached
            indexes[l]++;
            for (var i = l +1 ; i < numSubItems; i++) {
                indexes[i] = indexes[l] + (i - l);
            }
        }        
    }
    return result;
}

function cellsGuesses(idxs) {

    let gs = new Map();
    for (let i=0 ; i<idxs.length ; i++) {
        let c = 0;
        if (Array.isArray(obj.value.vals[idxs[i]])) {
            for (let j=0 ; j<obj.value.vals[idxs[i]].length ; j++) {
                if (obj.value.vals[[idxs[i]]][j]!='') {
                    c++;
                }
            }
        }
        gs.set(idxs[i],c)
    }
    return gs;
}

function guessNumbers(idxs) {

    let gs = new Map();
    for (let i=0 ; i<idxs.length ; i++) {
        if (Array.isArray(obj.value.vals[idxs[i]])) {
            for (let j=0 ; j<obj.value.vals[idxs[i]].length ; j++) {
                if (obj.value.vals[[idxs[i]]][j]!='') {
                    gs.set(obj.value.vals[[idxs[i]]][j],true)
                }
            }
        }
    }
    return gs
}

function keysInRange(map,min,max) {
    const newObj = new Map();
    for (const [key, value] of map.entries()) {
        if (value>=min && value<=max) {
            newObj.set(key,value)
        }
    }
    return newObj;
}

function arrayToMap(arr) {
    let map = new Map()
    for (let i=0 ; i<arr.length ; i++) {
        map.set(arr[i],true)
    }
    return map
}

function removeNotes(idx,noteMap) {
    let cell = obj.value.vals[idx]
    let r = false
    if (Array.isArray(cell)) {
        for (let i=0 ; i<cell.length ; i++) {
            if (cell[i],cell[i]!='' && noteMap.has(cell[i])) {
                cell[i] = ''
                r = true
            }
        }
    }
    return r
}

function naked(idxs,minNotes,maxNotes,numCells) {
    let updates = []

    let guesses = keysInRange(cellsGuesses(idxs),minNotes,maxNotes)
    let combos = combinations([...guesses.keys()],numCells)
    for (var i=0 ; i<combos.length ; i++) {
        let gns = guessNumbers(combos[i])
        if ([...gns.keys()].length==numCells) {

            let comboMap = arrayToMap(combos[i])

            for (let rr=0 ; rr<9 ; rr++) {
                if (Array.isArray(obj.value.vals[idxs[rr]])  && !comboMap.has(idxs[rr])) {
                    if (removeNotes(idxs[rr],gns)) {
                        updates.push(idxs[rr])
                    }
                }
            }
        }
    }
    return updates;
}

function nakedTriples(idxs) {
    return naked(idxs,2,3,3)
    // let updates = []

    // let guesses = keysInRange(cellsGuesses(idxs),2,3)
    // let combos = combinations([...guesses.keys()],3)
    // for (var i=0 ; i<combos.length ; i++) {
    //     let gns = guessNumbers(combos[i])
    //     if ([...gns.keys()].length==3) {

    //         // found 3 cells with 3 numbers, naked triple!
    //         // remove these 3 numbers from all other cells
    //         let comboMap = arrayToMap(combos[i])

    //         for (let rr=0 ; rr<9 ; rr++) {
    //             if (Array.isArray(obj.value.vals[idxs[rr]])  && !comboMap.has(idxs[rr])) {
    //                 if (removeNotes(idxs[rr],gns)) {
    //                     updates.push(idxs[rr])
    //                 }
    //             }
    //         }
    //     }
    // }
    // return updates;
}

function nakedQuads(idxs) {
    return naked(idxs,2,4,4)
    // let updates = []

    // let guesses = keysInRange(cellsGuesses(idxs),2,4)
    // let combos = combinations([...guesses.keys()],4)
    // console.log(combos)

    // for (var i=0 ; i<combos.length ; i++) {
    //     let gns = guessNumbers(combos[i])
    //     if ([...gns.keys()].length==4) {

    //         // found 4 cells with 4 numbers, naked quad!
    //         // remove these 4 numbers from all other cells
    //         let comboMap = arrayToMap(combos[i])

    //         for (let rr=0 ; rr<9 ; rr++) {
    //             if (Array.isArray(obj.value.vals[idxs[rr]])  && !comboMap.has(idxs[rr])) {
    //                 if (removeNotes(idxs[rr],gns)) {
    //                     updates.push(idxs[rr])
    //                 }
    //             }
    //         }
    //     }
    // }
    // return updates;
}

function nakedTriplesRows() {
    let updates = []

    for (let r=0 ; r<9 ; r++) {
        let rowIdxs = rowIndexes(r)
        updates = updates.concat(nakedTriples(rowIdxs))
    }

    return { name: "Naked Triples - Rows", updates }
}

function nakedTriplesCols() {
    let updates = []

    for (let c=0 ; c<9 ; c++) {
        let colIdxs = colIndexes(c)
        updates = updates.concat(nakedTriples(colIdxs))
    }

    return { name: "Naked Triples - Columns", updates }
}

function nakedTriplesSqrs() {
    let updates = []

    for (let r=0 ; r<9 ; r+=3) {
        for (let c=0 ; c<9 ; c+=3) {
            let sqrIdxs = sqrIndexes(r,c)
            updates = updates.concat(nakedTriples(sqrIdxs))
        }
    }
    return { name: "Naked Triples - Squares", updates }
}


function nakedQuadsRows() {
    let updates = []

    for (let r=0 ; r<9 ; r++) {
        let rowIdxs = rowIndexes(r)
        updates = updates.concat(nakedQuads(rowIdxs))
    }

    return { name: "Naked Quads - Rows", updates }
}

function nakedQuadsCols() {
    let updates = []

    for (let c=0 ; c<9 ; c++) {
        let colIdxs = colIndexes(c)
        updates = updates.concat(nakedQuads(colIdxs))
    }

    return { name: "Naked Quads - Columns", updates }
}

function nakedQuadsSqrs() {
    let updates = []

    for (let r=0 ; r<9 ; r+=3) {
        for (let c=0 ; c<9 ; c+=3) {
            let sqrIdxs = sqrIndexes(r,c)
            updates = updates.concat(nakedQuads(sqrIdxs))
        }
    }
    return { name: "Naked Quads - Squares", updates }
}


let step = 0;
let completed = ref(false)
const steps = [ 
    // check solved
    checkSolved,
    // update notes
    updatePossibilities,
    // hidden singles
    hiddenSinglesRows, hiddenSinglesCols, hiddenSinglesSqrs,
    // naked pairs
    nakedPairsRows, nakedPairsCols, nakedPairsSqrs,
    // naked triples
    nakedTriplesRows, nakedTriplesCols, nakedTriplesSqrs,
    // hidden pairs
    
    // hidden triples

    // naked quads
    nakedQuadsRows, nakedQuadsCols, nakedQuadsSqrs

    // hidden quads

    // pointing pairs

    // box/line reduction

    // x-wing

    // simple coloring

    // y-wing

    // sword fish

    // xyz wing

    // BUG

    // X-Cycles

    // XY-Chain

    // 3D Medusa

]

function nextStep() {
    if (completed.value) return
    let res = steps[step % steps.length]()
    obj.value.steps.push(res)
    if (res.updates.length>0) {
        step = 0
        completed.value = isCompleted()
    } else {
        step++
    }
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
  <div class="steps">
    <button :class="{ completed: completed }" @click="nextStep">Next Step</button>
    <Steps v-bind:steps="obj.steps" />
  </div>
</template>

<style scoped>

table {
    border-collapse: collapse;
    border-spacing: 0;
}

button.completed {
    background-color: aquamarine;
}
td {
    padding: 0px;
    width: 60px;
}
table.board {
    border: 2px solid black;
    width: 540px;
    float: left;
    table-layout: fixed;
}

div.steps {
    width: 50%;
    float: right;
}
</style>