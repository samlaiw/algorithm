<template>
  <div class="container text-black">
    <div class="col-12 text-center">
      <h1>
        利用演算法找出最佳路徑
      </h1>
      <table class="matrix mx-auto">
        <tr v-for="(m, i) in matrix" :key="i">
          <td class="border rounded-circle" v-for="(mm, j) in m" :key="j"
            :class="{blocked: mm == 1, start: mm == 2, end: mm == 3, path: mm == 4 && resultPath.show}"
            @click="block(i, j)">
          </td>
        </tr>
      </table>
    </div>
    <input type="button" class="btn-primary" value="產生路徑" @click="getPath" />
    
    <input type="button" class="btn-primary" data-bs-toggle="offcanvas" data-bs-target="#offcanvasTop"
      aria-controls="offcanvasTop" value="變換條件" />

    <div class="offcanvas offcanvas-top" tabindex="5" id="offcanvasTop" aria-labelledby="offcanvasTopLabel">
      <div class="offcanvas-header">
        <button type="button" class="btn-close text-reset" data-bs-dismiss="offcanvas" aria-label="Close"></button>
      </div>
      <div class="offcanvas-body">
        <div class="row">
          <div class="col-6">
            <h1>格子</h1>
            <div class="row my-3">
              <label class="col-2 col-form-label" for="inputW">X軸格子數</label>
              <div class="col-10">
                <input type="number" min="3" max="20" v-model="input.w">
              </div>
              <label class="col-2 col-form-label" for="inputH">Y軸格子數</label>
              <div class="col-10">
                <input type="number" min="3" max="20" v-model="input.h">
              </div>
            </div>
            <input type="button" class="btn-primary" value="更新格子" @click="generateGrid" />
          </div>
          <div class="col-6">
            <h1>設定</h1>
            <div class="row my-3" v-if="!heuristicDisabled">
              <label class="col-2 col-form-label" for="inputW">演算法</label>
              <div class="col-10">
                  <select id="cars" name="cars" v-model="heuristic">
                    <option v-for="(h, i) in heuristicOptions" :key="i" :value="h.value">{{ h.name }}</option>
                  </select>
              </div>
              <label class="col-2 col-form-label" for="inputH">搜尋法</label>
              <div class="col-10">
                <select id="cars" name="cars" v-model="heuristic">
                    <option v-for="(f, i) in finderOptions" :key="i" :value="f.value">{{ f.name }}</option>
                  </select>
              </div>
            </div>
          </div>

        </div>



      </div>
    </div>
  </div>

</template>

<script setup>
  import {
    ref,
    reactive,
    computed
  } from 'vue'
  import PF from 'pathfinding'

  // 0 = walkable
  // 1 = blocked
  // 2 = start
  // 3 = end
  // 4 = path
  const matrix = reactive([
    [2, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 3],
  ])

  const heuristic = ref('manhattan')
  const finder = ref('AStarFinder')

  const heuristicOptions = reactive([{
      value: 'manhattan',
      name: '曼哈頓距離'
    },
    {
      value: 'chebyshev',
      name: '柴比雪夫距離'
    },
    {
      value: 'euclidean',
      name: '歐幾里得距離'
    },
  ])

  const finderOptions = reactive([
    {
      value: 'BestFirstFinder',
      name: '最良優先搜尋'
    },
    {
      value: 'JumpPointFinder',
      name: '跳點搜索'
    },
    {
      value: 'BiAStarFinder',
      name: '雙向 A*'
    },
    {
      value: 'BiBestFirstFinder',
      name: '雙向最良優先搜尋'
    },
    {
      value: 'BiBreadthFirstFinder',
      name: '雙向廣度優先搜尋'
    },
    {
      value: 'BiDijkstraFinder',
      name: '雙向 dijkstra'
    },
  ])

  const resultPath = reactive({
    show: false,
    length: 0
  })

  const input = reactive({
    w: 5,
    h: 3
  })

  const block = (i, j) => {
    if (resultPath.show) {
      resultPath.show = false
      resultPath.length = 0
    }

    if (matrix[i][j] === 1) {
      matrix[i][j] = 0
    } else if (matrix[i][j] !== 2 && matrix[i][j] !== 3) {
      matrix[i][j] = 1
    }
  }

  const generateGrid = () => {
    matrix.splice(0, matrix.length)
    for (let i = 0; i < input.h; i++) {
      const arr = []
      for (let j = 0; j < input.w; j++) {
        arr.push(0)
      }
      matrix.push(arr)
    }
    matrix[0][0] = 2
    matrix[input.h - 1][input.w - 1] = 3
  }

  const getPath = () => {
    const start = {
      x: 0,
      y: 0
    }
    const end = {
      x: 0,
      y: 0
    }
    const matrixClone = JSON.parse(JSON.stringify(matrix))
    for (const i in matrixClone) {
      for (const j in matrixClone[i]) {
        if (matrixClone[i][j] == 2) {
          start.x = j
          start.y = i
          matrixClone[i][j] = 0
        } else if (matrixClone[i][j] == 3) {
          end.x = j
          end.y = i
          matrixClone[i][j] = 0
        } else if (matrixClone[i][j] == 4) {
          matrixClone[i][j] = 0
          matrix[i][j] = 0
        }
      }
    }
    const pfinder = new PF[finder.value]({
      heuristic: PF.Heuristic[heuristic.value]
    })
    const grid = new PF.Grid(matrixClone)
    // find path and remove start/end
    const path = pfinder.findPath(start.x, start.y, end.x, end.y, grid).slice(1, -1)
    for (const i in path) {
      matrix[path[i][1]][path[i][0]] = 4
    }
    resultPath.show = true
    resultPath.length = path.length
  }

  const heuristicDisabled = computed(() => {
    if (finder.value === 'AStarFinder' || finder.value === 'BestFirstFinder' ||
      finder.value === 'BiAStarFinder' || finder.value === 'BiBestFirstFinder'
    ) {
      return false
    }
    return true
  })
</script>