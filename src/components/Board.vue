<script setup>
import { matchedRouteKey } from "vue-router";
import Cell from "./Cell.vue";
import Steps from "./Steps.vue";
import { ref, onMounted } from "vue";

function valsOf(str) {
  let r = [];
  for (let i in str) {
    if (str[i] == "0") r.push([1, 2, 3, 4, 5, 6, 7, 8, 9]);
    else r.push(Number.parseInt(str[i]));
  }
  return r;
}

const obj = ref({
  //  vals: valsOf("000105000140000670080002400063070010900000003010090520007200080026000035000409000"),
  //    vals: valsOf("070408029002000004854020007008374200020000000003261700000093612200000403130642070"), // nake triple row
  // vals: valsOf("000000000904607000076804100309701080008000300050308702007502610000403208000000000"), // hidden pair
  // vals: valsOf("000001030231090000065003100678924300103050006000136700009360570006019843300000000"), // hidden triple
  //    vals: valsOf('000030086000020040090078520371856294900142375400397618200703859039205467700904132'), // naked quad square
  //    vals: valsOf('901500046425090081860010020502000000019000460600000002196040253200060817000001694'), // hidden quad
  //    vals: valsOf('017903600000080000900000507072010430000402070064370250701000065000030000005601720'), // pointing pairs
  //vals: valsOf('016007803090800000870001060048000300650009082039000650060900020080002936924600510'), // box/line reduction + y-wing
  // vals: valsOf('100000569492056108056109240009640801064010000218035604040500016905061402621000005'), //x-wing rows
  //    vals: valsOf('000000094760910050090002081070050010000709000080031067240100070010090045900000100'), //x-wing cols
  //vals: valsOf('100400006046091080005020000000500109090000050402009000000010900080930560500008004'), // simple coloring
  vals: valsOf(
    "002900000010057094060000030200006007006000809300400005005000000000700000701028006",
  ),
  steps: [],
});

onMounted(() => {
  console.log(`The initial count is ${obj.value.vals[0]}.`);
});

function rowIndexes(row) {
  let idxs = [];
  for (let c = 0; c < 9; c++) {
    idxs.push(row * 9 + c);
  }
  return idxs;
}

function colIndexes(col) {
  let idxs = [];
  for (let r = 0; r < 9; r++) {
    idxs.push(r * 9 + col);
  }
  return idxs;
}

function sqrIndexes(row, col) {
  const y = Math.floor(row / 3);
  const x = Math.floor(col / 3);
  let idxs = [];

  for (let r = 0; r < 3; r++) {
    for (let c = 0; c < 3; c++) {
      idxs.push(y * 27 + r * 9 + x * 3 + c);
    }
  }

  return idxs;
}

function cellIndex(r, c) {
  return r * 9 + c;
}

function isCompleted() {
  for (let i = 0; i < 81; i++) {
    if (Array.isArray(obj.value.vals[i])) {
      return false;
    }
  }
  return true;
}

function checkSolved() {
  let updated = 0;
  let updates = [];

  for (let i = 0; i < 81; i++) {
    if (Array.isArray(obj.value.vals[i])) {
      let num = null;
      for (let j = 0; j < 9; j++) {
        if (obj.value.vals[i][j] != "") {
          if (num == null) {
            num = obj.value.vals[i][j];
          } else {
            num = null;
            break;
          }
        }
      }
      if (num != null) {
        obj.value.vals[i] = num;
        updated++;
        updates.push(i);
      }
    }
  }
  return { name: "Check Solved", updated, updates };
}

function updatePossibilities() {
  // let updated = 0
  let updates = new Map();
  for (let r = 0; r < 9; r++) {
    for (let c = 0; c < 9; c++) {
      let idx = cellIndex(r, c);
      if (!Array.isArray(obj.value.vals[idx])) {
        let rowIdxs = rowIndexes(r);
        for (let i = 0; i < 9; i++) {
          if (Array.isArray(obj.value.vals[rowIdxs[i]])) {
            if (obj.value.vals[rowIdxs[i]][obj.value.vals[idx] - 1] != "") {
              obj.value.vals[rowIdxs[i]][obj.value.vals[idx] - 1] = "";
              updates.set(rowIdxs[i], true);
              // updated++
            }
          }
        }

        let colIdxs = colIndexes(c);
        for (let i = 0; i < 9; i++) {
          if (Array.isArray(obj.value.vals[colIdxs[i]])) {
            if (obj.value.vals[colIdxs[i]][obj.value.vals[idx] - 1] != "") {
              obj.value.vals[colIdxs[i]][obj.value.vals[idx] - 1] = "";
              updates.set(colIdxs[i], true);
              // updated++
            }
          }
        }

        let sqrIdxs = sqrIndexes(r, c);
        for (let i = 0; i < 9; i++) {
          if (Array.isArray(obj.value.vals[sqrIdxs[i]])) {
            if (obj.value.vals[sqrIdxs[i]][obj.value.vals[idx] - 1] != "") {
              obj.value.vals[sqrIdxs[i]][obj.value.vals[idx] - 1] = "";
              updates.set(sqrIdxs[i], true);
              // updated++
            }
          }
        }
      }
    }
  }
  return {
    name: "Update Possibilities",
    updated: updates.size,
    updates: [...updates.keys()],
  };
}

function cellContainsNote(cell, v) {
  return Array.isArray(cell) && cell[v - 1] == v;
}

function hiddenSinglesRows() {
  let updated = 0;
  let updates = [];

  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    for (let i = 1; i <= 9; i++) {
      let count = 0;
      let atC = -1;
      for (let c = 0; c < rowIdxs.length; c++) {
        let cell = obj.value.vals[rowIdxs[c]];
        if (cellContainsNote(cell, i)) {
          count++;
          atC = c;
        }
      }
      if (count == 1) {
        obj.value.vals[rowIdxs[atC]] = i;
        updated++;
        updates.push(rowIdxs[atC]);
      }
    }
  }

  return { name: "Hidden Singles - Rows", updated: updated != 0, updates };
}

function hiddenSinglesCols() {
  let updated = 0;
  let updates = [];

  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    for (let i = 1; i <= 9; i++) {
      let count = 0;
      let atR = -1;
      for (let r = 0; r < colIdxs.length; r++) {
        let cell = obj.value.vals[colIdxs[r]];
        if (cellContainsNote(cell, i)) {
          count++;
          atR = r;
        }
      }
      if (count == 1) {
        obj.value.vals[colIdxs[atR]] = i;
        updated++;
        updates.push(colIdxs[atR]);
      }
    }
  }

  return { name: "Hidden Singles - Columns", updated: updated != 0, updates };
}

function hiddenSinglesSqrs() {
  let updated = 0;
  let updates = [];

  for (let c = 0; c < 9; c += 3) {
    for (let r = 0; r < 9; r += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      for (let i = 1; i <= 9; i++) {
        let count = 0;
        let atX = -1;
        for (let x = 0; x < sqrIdxs.length; x++) {
          let cell = obj.value.vals[sqrIdxs[x]];
          if (cellContainsNote(cell, i)) {
            count++;
            atX = x;
          }
        }
        if (count == 1) {
          obj.value.vals[sqrIdxs[atX]] = i;
          updated++;
          updates.push(sqrIdxs[atX]);
        }
      }
    }
  }

  return { name: "Hidden Singles - Squares", updated: updated != 0, updates };
}

function cellContainsTwoNotes(cell) {
  if (Array.isArray(cell)) {
    let r = [];
    for (let i = 0; i < cell.length; i++) {
      if (cell[i] != "") {
        r.push(cell[i]);
      }
    }
    if (r.length == 2) {
      return r;
    }
    return false;
  } else {
    return false;
  }
}

function checkPairs(ps) {
  let res = [];
  while (ps.length > 1) {
    let p0 = ps.shift();
    for (let i = 0; i < ps.length; i++) {
      if (p0.p[0] == ps[i].p[0] && p0.p[1] == ps[i].p[1]) {
        res.push([p0, ps[i]]);
      }
    }
  }
  if (res.length > 0) return res;
  return false;
}

function nakedPairsRows() {
  let updated = 0;
  let updates = [];

  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let pairs = [];
    for (let c = 0; c < rowIdxs.length; c++) {
      let cell = obj.value.vals[rowIdxs[c]];
      let ns = cellContainsTwoNotes(cell);
      if (ns && ns.length == 2) {
        pairs.push({ at: rowIdxs[c], p: ns });
      }
    }
    let cp = checkPairs(pairs);
    while (cp.length > 0) {
      let p = cp.shift();
      for (let c = 0; c < rowIdxs.length; c++) {
        if (rowIdxs[c] == p[0].at || rowIdxs[c] == p[1].at) continue;
        let cell = obj.value.vals[rowIdxs[c]];
        if (!Array.isArray(cell)) continue;
        for (let x = 0; x < 9; x++) {
          if (cell[x] == p[0].p[0] || cell[x] == p[0].p[1]) {
            cell[x] = "";
            updated++;
            updates.push(rowIdxs[c]);
          }
        }
      }
    }
  }

  return { name: "Naked Pairs - Rows", updated: updated != 0, updates };
}

function nakedPairsCols() {
  let updated = 0;
  let updates = [];

  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let pairs = [];
    for (let r = 0; r < colIdxs.length; r++) {
      let cell = obj.value.vals[colIdxs[r]];
      let ns = cellContainsTwoNotes(cell);
      if (ns && ns.length == 2) {
        pairs.push({ at: colIdxs[r], p: ns });
      }
    }
    let cp = checkPairs(pairs);
    while (cp.length > 0) {
      let p = cp.shift();
      for (let r = 0; r < colIdxs.length; r++) {
        if (colIdxs[r] == p[0].at || colIdxs[r] == p[1].at) continue;
        let cell = obj.value.vals[colIdxs[r]];
        if (!Array.isArray(cell)) continue;
        for (let x = 0; x < 9; x++) {
          if (cell[x] == p[0].p[0] || cell[x] == p[0].p[1]) {
            cell[x] = "";
            updated++;
            updates.push(colIdxs[r]);
          }
        }
      }
    }
  }

  return { name: "Naked Pairs - Columns", updated: updated != 0, updates };
}

function nakedPairsSqrs() {
  let updated = 0;
  let updates = [];

  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);

      let pairs = [];
      for (let s = 0; s < sqrIdxs.length; s++) {
        let cell = obj.value.vals[sqrIdxs[s]];
        // console.log(`${r},${c},${sqrIdxs[s]},${cell}`)
        let ns = cellContainsTwoNotes(cell);
        if (ns && ns.length == 2) {
          pairs.push({ at: sqrIdxs[s], p: ns });
          // console.log(`two notes at rc=${r},${c}, notes=${ns}`)
        }
      }
      let cp = checkPairs(pairs);
      while (cp.length > 0) {
        // console.log('sqr pairs',cp);
        let p = cp.shift();
        for (let s = 0; s < sqrIdxs.length; s++) {
          if (sqrIdxs[s] == p[0].at || sqrIdxs[s] == p[1].at) continue;
          let cell = obj.value.vals[sqrIdxs[s]];
          if (!Array.isArray(cell)) continue;
          for (let x = 0; x < 9; x++) {
            if (cell[x] == p[0].p[0] || cell[x] == p[0].p[1]) {
              // console.log(cell)
              cell[x] = "";
              updated++;
              updates.push(sqrIdxs[s]);
            }
          }
        }
      }
    }
  }
  return { name: "Naked Pairs - Squares", updated: updated != 0, updates };
}

function numberIndexes(idxs) {
  let r = {};
  for (let i = 1; i <= 9; i++) {
    r[i] = [];
  }
  for (let i = 1; i <= 9; i++) {
    for (let x = 0; x < idxs.length; x++) {
      if (cellContainsNote(obj.value.vals[idxs[x]], i)) {
        r[i].push(idxs[x]);
      }
    }
  }
  // console.log(r)
  return r;
}
function combinations(items, numSubItems) {
  var result = [];
  var indexes = new Array(numSubItems);
  for (var i = 0; i < numSubItems; i++) {
    indexes[i] = i;
  }
  while (indexes[0] < items.length - numSubItems + 1) {
    var v = [];
    for (var i = 0; i < numSubItems; i++) {
      v.push(items[indexes[i]]);
    }
    result.push(v);
    indexes[numSubItems - 1]++;
    var l = numSubItems - 1; // reference always is the last position at beginning
    while (
      indexes[numSubItems - 1] >= items.length &&
      indexes[0] < items.length - numSubItems + 1
    ) {
      l--; // the last position is reached
      indexes[l]++;
      for (var i = l + 1; i < numSubItems; i++) {
        indexes[i] = indexes[l] + (i - l);
      }
    }
  }
  return result;
}

function cellsGuesses(idxs) {
  let gs = new Map();
  for (let i = 0; i < idxs.length; i++) {
    let c = 0;
    if (Array.isArray(obj.value.vals[idxs[i]])) {
      for (let j = 0; j < obj.value.vals[idxs[i]].length; j++) {
        if (obj.value.vals[[idxs[i]]][j] != "") {
          c++;
        }
      }
    }
    gs.set(idxs[i], c);
  }
  return gs;
}

function guessNumbers(idxs) {
  let gs = new Map();
  for (let i = 0; i < idxs.length; i++) {
    if (Array.isArray(obj.value.vals[idxs[i]])) {
      for (let j = 0; j < obj.value.vals[idxs[i]].length; j++) {
        if (obj.value.vals[[idxs[i]]][j] != "") {
          gs.set(obj.value.vals[[idxs[i]]][j], true);
        }
      }
    }
  }
  return gs;
}

function keysInRange(map, min, max) {
  const newObj = new Map();
  for (const [key, value] of map.entries()) {
    if (value >= min && value <= max) {
      newObj.set(key, value);
    }
  }
  return newObj;
}

function arrayToMap(arr) {
  let map = new Map();
  for (let i = 0; i < arr.length; i++) {
    map.set(arr[i], true);
  }
  return map;
}

function removeNotes(idx, noteMap) {
  let cell = obj.value.vals[idx];
  let r = false;
  if (Array.isArray(cell)) {
    for (let i = 0; i < cell.length; i++) {
      if ((cell[i], cell[i] != "" && noteMap.has(cell[i]))) {
        cell[i] = "";
        r = true;
      }
    }
  }
  return r;
}

function naked(idxs, minNotes, maxNotes, numCells) {
  let updates = [];

  let guesses = keysInRange(cellsGuesses(idxs), minNotes, maxNotes);
  let combos = combinations([...guesses.keys()], numCells);
  for (var i = 0; i < combos.length; i++) {
    let gns = guessNumbers(combos[i]);
    if ([...gns.keys()].length == numCells) {
      let comboMap = arrayToMap(combos[i]);

      for (let rr = 0; rr < 9; rr++) {
        if (
          Array.isArray(obj.value.vals[idxs[rr]]) &&
          !comboMap.has(idxs[rr])
        ) {
          if (removeNotes(idxs[rr], gns)) {
            updates.push(idxs[rr]);
          }
        }
      }
    }
  }
  return updates;
}

function hidden(idxs, numCells) {
  let nids = numberIndexes(idxs);

  let filtered = {};
  for (const [key, value] of Object.entries(nids)) {
    if (value.length > 0 && value.length <= numCells) {
      filtered[key] = value;
    }
  }

  let possibleNumbers = [];
  for (const key of Object.keys(filtered)) {
    possibleNumbers.push(Number.parseInt(key));
  }

  let possibleNumberCombinations = combinations(possibleNumbers, numCells);

  for (const combo of possibleNumberCombinations) {
    let cellIndexes = new Map();
    for (const num of combo) {
      let cellIndexesForNum = filtered[num];
      for (let cellIndex of cellIndexesForNum) {
        cellIndexes.set(cellIndex, cellIndex);
      }
    }
    let comboCellIndexes = [...cellIndexes.keys()];
    if (comboCellIndexes.length == numCells) {
      let updated = false;
      for (let i of comboCellIndexes) {
        updated = removeNotesExcept(i, combo) || updated;
      }
      if (updated) return comboCellIndexes;
    }
  }

  return [];
}

function asRows(idxs) {
  let rows = [];

  for (let i = 0; i < idxs.length; ) {
    let row = [idxs[i]];
    i++;
    while (i < idxs.length && idxs[i] == row[row.length - 1] + 1) {
      row.push(idxs[i]);
      i++;
    }
    rows.push(row);
  }
  return rows;
}

function asCols(idxs) {
  let cols = [];

  let m = new Map();
  for (let i of idxs) {
    m.set(i, i);
  }

  let inCol = new Map();

  for (let i = 0; i < idxs.length; i++) {
    if (inCol.has(idxs[i])) continue;
    let col = [idxs[i]];
    inCol.set(idxs[i], true);
    let next = idxs[i] + 9;
    while (m.has(next) && !inCol.has(next)) {
      col.push(next);
      inCol.set(next, true);
      next = next + 9;
    }
    cols.push(col);
  }

  return cols;
}

function row(idx) {
  return Math.floor(idx / 9);
}

function col(idx) {
  return idx % 9;
}

function sqr(idx) {
  let r = row(idx);
  let c = col(idx);
  let y = Math.floor(r / 3);
  let x = Math.floor(c / 3);
  let s = y * 3 + x;
  // console.log(`r=${r}, c=${c}, s=${s}`)
  return s;
}

function boxLineReduction(idxs) {
  // console.log('boxLineReduction',idxs)

  let nids = numberIndexes(idxs);

  let filtered = {};
  for (const [key, value] of Object.entries(nids)) {
    if (value.length > 1 && value.length <= 3) {
      let v = { sqrs: new Map(), idxs: value };
      for (let i = 0; i < value.length; i++) {
        let s = sqr(value[i]);
        v.sqrs.set(s, s);
      }
      filtered[key] = v;
    }
  }
  // console.log(filtered)

  for (const [key, value] of Object.entries(filtered)) {
    let sqrs = [...value.sqrs.keys()];
    if (sqrs.length == 1) {
      // console.log(`${key} only appears in sqr ${sqrs[0]}`)
      let s = sqrs[0];
      let r = Math.floor(s / 3);
      let c = s - 3 * r;
      // console.log(`s=${s},r=${r*3},c=${c*3}`)
      let sqrIdxs = sqrIndexes(r * 3, c * 3);
      let otherSqrIdxs = sqrIdxs.filter((i) => !idxs.includes(i));
      let updates = [];
      for (let i of otherSqrIdxs) {
        if (Array.isArray(obj.value.vals[i])) {
          if (obj.value.vals[i][key - 1] == `${key}`) {
            updates.push(i);
            obj.value.vals[i][key - 1] = "";
          }
        }
      }
      if (updates.length > 0) {
        return updates;
      }
    }
  }
  return [];
}

function pointingPairs(idxs) {
  // console.log('pointingPairs',idxs)

  let nids = numberIndexes(idxs);

  // console.log(nids)
  let filtered = {};
  for (const [key, value] of Object.entries(nids)) {
    if (value.length > 1 && value.length <= 3) {
      let rcs = { rows: new Map(), cols: new Map(), coords: [] };
      for (let i = 0; i < value.length; i++) {
        let r = row(value[i]);
        let c = col(value[i]);
        rcs.rows.set(r, r);
        rcs.cols.set(c, c);
        rcs.coords.push({ r, c });
      }
      // rcs.rows = [ ... rcs.rows.keys()]
      // rcs.cols = [ ... rcs.cols.keys()]
      filtered[key] = rcs;
    }
  }
  // console.log(filtered)

  for (const [key, value] of Object.entries(filtered)) {
    if (value.rows.size == 1) {
      let r = value.rows.keys().next().value;
      // console.log(`${key} in row ${r} at cols ${[ ... value.cols.keys()]}`)
      let ris = rowIndexes(r);
      ris = ris.filter((v) => !value.cols.has(col(v)));
      // console.log(ris)
      let updates = [];
      for (let ir of ris) {
        if (removeNote(ir, key)) {
          // console.log(`removed note ${key} from ${ir}`)
          updates.push(ir);
        }
      }
      if (updates.length > 0) {
        return updates;
      }
    }
    if (value.cols.size == 1) {
      let c = value.cols.keys().next().value;
      // console.log(`${key} in col ${c} at rows ${[ ... value.rows.keys()]}`)
      let cis = colIndexes(c);
      cis = cis.filter((v) => !value.rows.has(row(v)));
      // console.log(cis)
      let updates = [];
      for (let ic of cis) {
        if (removeNote(ic, key)) {
          // console.log(`removed note ${key} from ${ic}`)
          updates.push(ic);
        }
      }
      if (updates.length > 0) {
        return updates;
      }
    }
  }

  return [];
}

function removeNote(idx, note) {
  if (!Array.isArray(obj.value.vals[idx])) return false;
  const cell = obj.value.vals[idx];
  if (cell[note - 1] == "") return false;
  cell[note - 1] = "";
  return true;
}

function removeNotesExcept(idx, notes) {
  // console.log('------------------------')
  let updated = false;
  let m = new Map();
  notes.forEach((n) => m.set(n, n));
  let cell = obj.value.vals[idx];

  for (let i = 0; i < cell.length; i++) {
    if (cell[i] != "" && !m.has(cell[i])) {
      cell[i] = "";
      updated = true;
    }
  }
  // console.log('removeNotesExcept',idx,obj.value.vals[idx],notes,m,updated)
  return updated;
}

function hiddenPairs(idxs) {
  return hidden(idxs, 2);
}
function hiddenTriples(idxs) {
  return hidden(idxs, 3);
}

function hiddenQuads(idxs) {
  return hidden(idxs, 4);
}

function nakedTriples(idxs) {
  return naked(idxs, 2, 3, 3);
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
  return naked(idxs, 2, 4, 4);
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
  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let updates = nakedTriples(rowIdxs);
    if (updates.length > 0) return { name: "Naked Triples - Rows", updates };
  }

  return { name: "Naked Triples - Rows", updates: [] };
}

function nakedTriplesCols() {
  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let updates = nakedTriples(colIdxs);
    if (updates.length > 0) return { name: "Naked Triples - Columns", updates };
  }

  return { name: "Naked Triples - Columns", updates: [] };
}

function nakedTriplesSqrs() {
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      let updates = nakedTriples(sqrIdxs);
      if (updates.length > 0)
        return { name: "Naked Triples - Squares", updates };
    }
  }
  return { name: "Naked Triples - Squares", updates: [] };
}

function nakedQuadsRows() {
  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let updates = nakedQuads(rowIdxs);
    if (updates.length > 0) return { name: "Naked Quads - Rows", updates };
  }

  return { name: "Naked Quads - Rows", updates: [] };
}

function nakedQuadsCols() {
  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let updates = nakedQuads(colIdxs);
    if (updates.length > 0) return { name: "Naked Quads - Columns", updates };
  }

  return { name: "Naked Quads - Columns", updates: [] };
}

function nakedQuadsSqrs() {
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      let updates = nakedQuads(sqrIdxs);
      if (updates.length > 0) return { name: "Naked Quads - Squares", updates };
    }
  }
  return { name: "Naked Quads - Squares", updates: [] };
}

function hiddenPairsRows() {
  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let updates = hiddenPairs(rowIdxs);
    if (updates.length > 0) return { name: "Hidden Pairs - Rows", updates };
  }

  return { name: "Hidden Pairs - Rows", updates: [] };
}

function hiddenPairsCols() {
  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let updates = hiddenPairs(colIdxs);
    if (updates.length > 0) return { name: "Hidden Pairs - Columns", updates };
  }

  return { name: "Hidden Pairs - Columns", updates: [] };
}

function hiddenPairsSqrs() {
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      let updates = hiddenPairs(sqrIdxs);
      if (updates.length > 0)
        return { name: "Hidden Pairs - Squares", updates };
    }
  }
  return { name: "Hidden Pairs - Squares", updates: [] };
}

function hiddenTriplesRows() {
  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let updates = hiddenTriples(rowIdxs);
    if (updates.length > 0) return { name: "Hidden Triples - Rows", updates };
  }

  return { name: "Hidden Triples - Rows", updates: [] };
}

function hiddenTriplesCols() {
  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let updates = hiddenTriples(colIdxs);
    if (updates.length > 0)
      return { name: "Hidden Triples - Columns", updates };
  }

  return { name: "Hidden Triples - Columns", updates: [] };
}

function hiddenTriplesSqrs() {
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      let updates = hiddenTriples(sqrIdxs);
      if (updates.length > 0)
        return { name: "Hidden Triples - Squares", updates };
    }
  }
  return { name: "Hidden Triples - Squares", updates: [] };
}

function hiddenQuadsRows() {
  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let updates = hiddenQuads(rowIdxs);
    if (updates.length > 0) return { name: "Hidden Quads - Rows", updates };
  }

  return { name: "Hidden Quads - Rows", updates: [] };
}

function hiddenQuadsCols() {
  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let updates = hiddenQuads(colIdxs);
    if (updates.length > 0) return { name: "Hidden Quads - Columns", updates };
  }

  return { name: "Hidden Quads - Columns", updates: [] };
}

function hiddenQuadsSqrs() {
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      let updates = hiddenQuads(sqrIdxs);
      if (updates.length > 0)
        return { name: "Hidden Quads - Squares", updates };
    }
  }
  return { name: "Hidden Quads - Squares", updates: [] };
}

function pointingPairsSqrs() {
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let sqrIdxs = sqrIndexes(r, c);
      let updates = pointingPairs(sqrIdxs);
      if (updates.length > 0) return { name: "Pointing Pairs", updates };
    }
  }
  return { name: "Pointing Pairs", updates: [] };
}

function appearTwice(idxs) {
  let nids = numberIndexes(idxs);
  let filtered = {};

  for (const [key, value] of Object.entries(nids)) {
    if (value.length == 2) {
      filtered[key] = value;
    }
  }
  return filtered;
}

function xwingRows() {
  // console.log('xwing rows')

  let notesByRow = new Map();

  //  by row
  for (let i = 0; i < 9; i++) {
    let nids = numberIndexes(rowIndexes(i));
    let fids = {};
    for (const [key, value] of Object.entries(nids)) {
      if (value.length == 2) {
        fids[key] = value;
      }
    }
    notesByRow.set(i, fids);
  }

  // console.log(notesByRow)

  for (let n = 1; n <= 9; n++) {
    let rowsWithNoteN = [];
    for (let r = 0; r < 9; r++) {
      let mr = notesByRow.get(r);
      if (mr[n]) {
        let notes = [];
        for (let i = 0; i < mr[n].length; i++) {
          notes.push({ r, idx: mr[n][i], c: col(mr[n][i]) });
        }
        rowsWithNoteN.push({ row: r, notes });
      }
    }

    let rowPairs = combinations(rowsWithNoteN, 2);

    //        console.log('row pairs',rowPairs)

    for (let rowPairWithNoteN of rowPairs) {
      // console.log(`rows with note ${n} twice:`,rowPairWithNoteN)

      // check same columsn

      if (
        rowPairWithNoteN[0].notes[0].c == rowPairWithNoteN[1].notes[0].c &&
        rowPairWithNoteN[0].notes[1].c == rowPairWithNoteN[1].notes[1].c
      ) {
        // console.log(`rows with note ${n} twice in same columns`)
        // remove note from columns

        let removeNote = n;
        let col0 = rowPairWithNoteN[0].notes[0].c;
        let col1 = rowPairWithNoteN[1].notes[1].c;
        let col0Idxs = colIndexes(col0);
        let col1Idxs = colIndexes(col1);
        let skipRow0 = rowPairWithNoteN[0].row;
        let skipRow1 = rowPairWithNoteN[1].row;
        // console.log('remove note:',removeNote,'col0',col0,'col1',col1,'skip row0',skipRow0,'skip row1',skipRow1)
        let removedNotesFrom = [];

        for (let r = 0; r < 9; r++) {
          if (r == skipRow0) continue;
          if (r == skipRow1) continue;

          let col0Cell = obj.value.vals[col0Idxs[r]];
          let col1Cell = obj.value.vals[col1Idxs[r]];

          // console.log('row',r,'cell0',col0Cell,'cell1',col1Cell)

          if (Array.isArray(col0Cell)) {
            // console.log(col0Cell[removeNote-1],removeNote)
            if (col0Cell[removeNote - 1] == removeNote) {
              // console.log(`remove note ${removeNote} from col:${col0},row:${r}`)
              col0Cell[removeNote - 1] = "";
              removedNotesFrom.push(col0Idxs[r]);
            }
          }
          if (Array.isArray(col1Cell)) {
            // console.log(col1Cell[removeNote-1],removeNote)
            if (col1Cell[removeNote - 1] == removeNote) {
              // console.log(`remove note ${removeNote} from col:${col1},row:${r}`)
              col1Cell[removeNote - 1] = "";
              removedNotesFrom.push(col1Idxs[r]);
            }
          }
        }

        // console.log('removedNotesFrom',removedNotesFrom)

        if (removedNotesFrom.length > 0) {
          return { name: "X-Wing - Rows", updates: removedNotesFrom };
        }
      }
    }
  }

  return { name: "X-Wing - Rows", updates: [] };
}

function xwingCols() {
  // console.log('xwing cols')

  let notesByCol = new Map();

  //  by col
  for (let i = 0; i < 9; i++) {
    let nids = numberIndexes(colIndexes(i));
    let fids = {};
    for (const [key, value] of Object.entries(nids)) {
      if (value.length == 2) {
        fids[key] = value;
      }
    }
    notesByCol.set(i, fids);
  }

  //    console.log(notesByCol)

  for (let n = 1; n <= 9; n++) {
    let colsWithNoteN = [];
    for (let c = 0; c < 9; c++) {
      let mc = notesByCol.get(c);
      if (mc[n]) {
        let notes = [];
        for (let i = 0; i < mc[n].length; i++) {
          notes.push({ c, idx: mc[n][i], r: row(mc[n][i]) });
        }
        colsWithNoteN.push({ col: c, notes });
      }
    }

    //        console.log(`note: ${n}`,colsWithNoteN)

    let colPairs = combinations(colsWithNoteN, 2);

    //        console.log('col pairs',colPairs)

    for (let colPairWithNoteN of colPairs) {
      // if (colsWithNoteN.length == 2) {
      //            console.log(`cols with note ${n} twice:`,colPairWithNoteN)

      // check same row

      if (
        colPairWithNoteN[0].notes[0].r == colPairWithNoteN[1].notes[0].r &&
        colPairWithNoteN[0].notes[1].r == colPairWithNoteN[1].notes[1].r
      ) {
        //                console.log(`cols with note ${n} twice in same rows`)
        // remove note from rows

        let removeNote = n;
        let row0 = colPairWithNoteN[0].notes[0].r;
        let row1 = colPairWithNoteN[1].notes[1].r;
        let row0Idxs = rowIndexes(row0);
        let row1Idxs = rowIndexes(row1);
        let skipCol0 = colPairWithNoteN[0].col;
        let skipCol1 = colPairWithNoteN[1].col;
        //                console.log('remove note:',removeNote,'row0',row0,'row1',row1,'skip col0',skipCol0,'skip col1',skipCol1)
        let removedNotesFrom = [];

        for (let c = 0; c < 9; c++) {
          if (c == skipCol0) continue;
          if (c == skipCol1) continue;

          let row0Cell = obj.value.vals[row0Idxs[c]];
          let row1Cell = obj.value.vals[row1Idxs[c]];

          // console.log('row',r,'cell0',col0Cell,'cell1',col1Cell)

          if (Array.isArray(row0Cell)) {
            // console.log(col0Cell[removeNote-1],removeNote)
            if (row0Cell[removeNote - 1] == removeNote) {
              // console.log(`remove note ${removeNote} from col:${col0},row:${r}`)
              row0Cell[removeNote - 1] = "";
              removedNotesFrom.push(row0Idxs[c]);
            }
          }
          if (Array.isArray(row1Cell)) {
            // console.log(col1Cell[removeNote-1],removeNote)
            if (row1Cell[removeNote - 1] == removeNote) {
              // console.log(`remove note ${removeNote} from col:${col1},row:${r}`)
              row1Cell[removeNote - 1] = "";
              removedNotesFrom.push(row1Idxs[c]);
            }
          }
        }

        // console.log('removedNotesFrom',removedNotesFrom)

        if (removedNotesFrom.length > 0) {
          return { name: "X-Wing - Cols", updates: removedNotesFrom };
        }
      }
    }
  }

  return { name: "X-Wing - Cols", updates: [] };
}

function boxLineReductionRows() {
  for (let r = 0; r < 9; r++) {
    let rowIdxs = rowIndexes(r);
    let updates = boxLineReduction(rowIdxs);
    if (updates.length > 0)
      return { name: "Box/Line Reduction - Rows", updates };
  }

  return { name: "Box/Line Reduction - Rows", updates: [] };
}

function boxLineReductionCols() {
  for (let c = 0; c < 9; c++) {
    let colIdxs = colIndexes(c);
    let updates = boxLineReduction(colIdxs);
    if (updates.length > 0)
      return { name: "Box/Line Reduction - Columns", updates };
  }

  return { name: "Box/Line Reduction - Columns", updates: [] };
}

function chaining() {
  let nodes = [];

  //  by col
  for (let i = 0; i < 9; i++) {
    let nids = numberIndexes(colIndexes(i));
    let fids = {};
    for (const [key, value] of Object.entries(nids)) {
      if (value.length == 2) {
        fids[key] = value.map((v) => {
          return {
            type: "COL",
            col: i,
            idx: v,
            note: Number.parseInt(key),
            row: row(v),
          };
        });
        fids[key][0].pair = fids[key][1].idx;
        fids[key][1].pair = fids[key][0].idx;
        nodes.push(fids[key][0]);
        nodes.push(fids[key][1]);
      }
    }
  }

  //  by row
  for (let i = 0; i < 9; i++) {
    let nids = numberIndexes(rowIndexes(i));
    let fids = {};
    for (const [key, value] of Object.entries(nids)) {
      if (value.length == 2) {
        fids[key] = value.map((v) => {
          return {
            type: "ROW",
            col: col(v),
            idx: v,
            note: Number.parseInt(key),
            row: i,
          };
        });
        fids[key][0].pair = fids[key][1].idx;
        fids[key][1].pair = fids[key][0].idx;
        nodes.push(fids[key][0]);
        nodes.push(fids[key][1]);
      }
    }
  }

  //  by square
  for (let r = 0; r < 9; r += 3) {
    for (let c = 0; c < 9; c += 3) {
      let nids = numberIndexes(sqrIndexes(r, c));
      let s = (r / 3) * 3 + c / 3;
      let fids = {};
      for (const [key, value] of Object.entries(nids)) {
        if (value.length == 2) {
          fids[key] = value.map((v) => {
            return {
              type: "SQR",
              sqr: s,
              col: col(v),
              idx: v,
              note: Number.parseInt(key),
              row: row(v),
            };
          });
          fids[key][0].pair = fids[key][1].idx;
          fids[key][1].pair = fids[key][0].idx;
          nodes.push(fids[key][0]);
          nodes.push(fids[key][1]);
        }
      }
    }
  }

  for (let n = 1; n <= 9; n++) {
    let ncols = [];
    let nrows = [];
    let nsqrs = [];
    let found = false;
    for (let i = 0; i < nodes.length; i++) {
      if (nodes[i].note == n) {
        found = true;
        if (nodes[i].type == "COL") ncols.push(nodes[i]);
        else if (nodes[i].type == "ROW") nrows.push(nodes[i]);
        else if (nodes[i].type == "SQR") nsqrs.push(nodes[i]);
      }
    }
    if (found) {
      console.log(ncols, nrows, nsqrs);
    }
  }

  return { name: "Simple Colouring/Chaining", updates: [] };
}
// scoring: https://github.com/pocketjoso/sudokuJS/blob/master/sudokuJS.js
// boards: https://github.com/einaregilsson/sudoku.js/blob/master/norvig/top95.txt
// https://github.com/ADi7YA26/sudoku-react-app/blob/main/src/helper/solver.js
// https://github.com/robatron/sudoku.js/blob/master/README.md
let step = 0;
let completed = ref(false);
const steps = [
  // check solved
  checkSolved,
  // update notes
  updatePossibilities,
  // hidden singles
  hiddenSinglesRows,
  hiddenSinglesCols,
  hiddenSinglesSqrs,
  // naked pairs
  nakedPairsRows,
  nakedPairsCols,
  nakedPairsSqrs,
  // naked triples
  nakedTriplesRows,
  nakedTriplesCols,
  nakedTriplesSqrs,
  // hidden pairs
  hiddenPairsRows,
  hiddenPairsCols,
  hiddenPairsSqrs,
  // hidden triples
  hiddenTriplesRows,
  hiddenTriplesCols,
  hiddenTriplesSqrs,
  // naked quads
  nakedQuadsRows,
  nakedQuadsCols,
  nakedQuadsSqrs,
  // hidden quads
  hiddenQuadsRows,
  hiddenQuadsCols,
  hiddenQuadsSqrs,
  // pointing pairs
  pointingPairsSqrs,
  // box/line reduction
  boxLineReductionRows,
  boxLineReductionCols,

  // TOUGH ------

  // x-wing
  xwingRows,
  xwingCols,
  // simple coloring
  chaining,
  // y-wing

  // sword fish

  // xyz wing

  // BUG

  // X-Cycles

  // XY-Chain

  // 3D Medusa
];

function nextStep() {
  if (completed.value) return false;
  if (step >= steps.length) return false;
  let res = steps[step]();
  obj.value.steps.push(res);
  if (res.updates.length > 0) {
    step = 0;
    completed.value = isCompleted();
  } else {
    step++;
  }
  console.log(step);
  return true;
}

function solve() {
  while (nextStep()) {}
  console.log(completed.value ? "SOLVED" : "NOT SOLVED");
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
                  <Cell
                    v-bind:x="
                      obj.vals[
                        ((r - 1) * 3 + (rr - 1)) * 9 + (c - 1) * 3 + (cc - 1)
                      ]
                    "
                    v-bind:u="
                      obj.steps.length == 0
                        ? false
                        : obj.steps[obj.steps.length - 1].updates.includes(
                            ((r - 1) * 3 + (rr - 1)) * 9 +
                              (c - 1) * 3 +
                              (cc - 1),
                          )
                    "
                  />
                </td>
              </tr>
            </tbody>
          </table>
        </td>
      </tr>
    </tbody>
  </table>
  <div class="steps">
    <button :class="{ completed: completed }" @click="nextStep">
      Next Step
    </button>
    <button :class="{ completed: completed }" @click="solve">Solve</button>
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
