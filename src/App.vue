<template>
  <div class="deltaForceData">

    <div v-if="backToTopButton" @click="gotop" class=" top">
      TOP↑
    </div>
    <div class="ner">
      <div class="search">
        <h3 class="tt">小学生DeltaForce</h3>
        <div class="line">
          <!-- 鼠标回车  -->
          <input @keyup.enter="onsearch" v-model="searchValue" type="text" placeholder="搜索物品名称">
          <div class="sbtn" @click="onsearch">搜索</div>
        </div>
      </div>
      <div class="searchContent">
        <ul class="nav shadow-sm">
          <li v-for="(item, index) in treeList" :key="index">
            <p>{{ item.name }}</p>
            <ul class="child">
              <li v-for="child in item.children" :key="child.type" :class="{ act: child.type === classification }"
                @click="changeType(child)">{{
                  child.name }}</li>
            </ul>
          </li>
        </ul>
        <div class=" w-full">
          <div class="filters">

            <el-popover trigger="click">
              <template #reference>
                <el-badge :value="filters.grades.length" :show-zero="false">
                  <el-button size="large" type="default"><i class="bi bi-funnel mr-1"></i> 物品等级</el-button>
                </el-badge>
              </template>
              <el-checkbox-group @change="changeGrades" v-model="filters.grades" class="itemColor">
                <el-checkbox :label="item.grade" v-for="(item, index) in colorFiltersList" :key="index">
                  <div class="checkitem" :class="item.class">
                    <i :class="item.icon" class=" mr-2"></i>
                    <span>{{ item.name }}</span>
                  </div>
                </el-checkbox>
              </el-checkbox-group>

            </el-popover>

          </div>
          <ul class="objectList">
            <li data-aos="fade-up" data-aos-duration="300" :data-aos-delay="(index % 6) * 100"
              v-for="(item, index) in filteredItems" :key="item.id" :class="`shadow-sm L${item.grade}`">
              <div class="image">
                <img v-lazy="item.pic" width="50" height="50">
              </div>
              <div class="info">
                <h3>{{ item.objectName }}</h3>
                <div class="taglist">
                  <span :data-in="item.style" v-for="(tag, index) in getRowTag(item)" class="tag">{{ tag.name }}</span>
                </div>
                <p> {{ item.desc }}</p>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div v-if="vshowloading" class="loadDtaa">
      <div class="sb">
        <p>{{ loadingJsName }}</p>
      </div>
    </div>
  </div>
</template>
<script setup>
import { reactive, ref } from 'vue'
const filteredItems = ref([])
import Fuse from 'fuse.js'

const backToTopButton = ref(false)
const vshowloading = ref(false)
const loadingJsName = ref('')
const searchValue = ref('')
// const acname = ref('消耗品')
const classification = ref('consume')
const filters = reactive({
  grades: []
})

const colorFiltersList = ref([
  { name: '红色', class: 'text-red-700', grade: 6, icon: 'bi bi-6-square' },
  { name: '金色', class: 'text-yellow-600', grade: 5, icon: 'bi bi-5-square' },
  { name: '紫色', class: 'text-purple-600', grade: 4, icon: 'bi bi-4-square' },
  { name: '蓝色', class: 'text-blue-700', grade: 3, icon: 'bi bi-3-square' },
  { name: '绿色', class: 'text-green-600', grade: 2, icon: 'bi bi-2-square' },
  { name: '灰色', class: 'text-gray-700', grade: 1, icon: 'bi bi-1-square' }

])
let treeList = [
  {
    "name": "道具",
    "children": [
      { "name": "消耗品", "path": "./json/props/consume.json", "type": "consume" },
      { "name": "收集品", "path": "./json/props/collection.json", "type": "collection" },
      { "name": "钥匙", "path": "./json/props/key.json", "type": "key" },
      { "name": "曼德尔转", "path": "./json/props/mandel.json", "type": "mandel" }

    ]
  },
  {
    "name": "装备",
    "children": [
      { "name": "护甲", "path": "./json/protect/armor.json", "type": "armor" },
      { "name": "背包", "path": "./json/protect/bag.json", "type": "bag" },
      { "name": "胸挂", "path": "./json/protect/chest.json", "type": "chest" },
      { "name": "头盔", "path": "./json/protect/helmet.json", "type": "helmet" }
    ]
  },
  {
    "name": "配件",
    "children": [
      { "name": "后握把", "path": "./json/acc/accBackGrip.json", "type": "accBackGrip" },
      { "name": "枪管", "path": "./json/acc/accBarrel.json", "type": "accBarrel" },
      { "name": "前握把", "path": "./json/acc/accForeGrip.json", "type": "accForeGrip" },
      { "name": "功能性配件", "path": "./json/acc/accFunctional.json", "type": "accFunctional" },
      { "name": "护木", "path": "./json/acc/accHandGuard.json", "type": "accHandGuard" },
      { "name": "弹匣", "path": "./json/acc/accMagazine.json", "type": "accMagazine" },
      { "name": "枪口", "path": "./json/acc/accMuzzle.json", "type": "accMuzzle" },
      { "name": "瞄具", "path": "./json/acc/accScope.json", "type": "accScope" },
      { "name": "枪托", "path": "./json/acc/accStock.json", "type": "accStock" }
    ]
  },
  {
    "name": "枪械",
    "children": [
      { "name": "全部", "path": "./json/gun/gunRifle.json", "type": "gunRifle" }
    ]
  },

]
var fuse = {}
let allData = {}
let allList = []
let loadJsCount = {
  total: 0,
  loaded: 0,
}
window.addEventListener('scroll', () => {
  if (window.scrollY > 700) { // 调整 100 为您需要的像素值
    backToTopButton.value = true
  } else {
    backToTopButton.value = false
  }
});


function gotop() {
  window.scrollTo(0, 0)
}
async function loadScript() {
  loadingJsName.value = `🔨正在准备数据..`
  vshowloading.value = true
  for (let i = 0; i < treeList.length; i++) {
    loadJsCount.total = loadJsCount.total + treeList[i].children.length
    for (let j = 0; j < treeList[i].children.length; j++) {
      let item = treeList[i].children[j]
      getJsonData(item)
    }
  }

}

function changeGrades(arr) {
  console.log(arr)
  if (arr.length === 0) {
    filteredItems.value = allData[row.type]
    return
  }
  filteredItems.value = allData[classification.value].filter(item => {
    return arr.includes(item.grade)
  })
}

function loadCompleted() {
  loadingJsName.value = `准备加载完成..`
  vshowloading.value = false
  const fuseOptions = {
    keys: [
      "objectName",
      "desc.firstName"
    ]
  };

  fuse = new Fuse(allList, fuseOptions);
  let first = treeList[0].children[0]
  changeType(first)
}

function onsearch() {
  if (!searchValue.value) {
    return
  }
  let result = fuse.search(searchValue.value)
  filteredItems.value = result.map(item => item.item)
}
function resetFilter() {
  filters.grades = []
}
function changeType(row) {
  resetFilter()
  classification.value = row.type
  loadingJsName.value = `🔨正在准备数据..`
  vshowloading.value = true
  setTimeout(() => {
    vshowloading.value = false
  }, 1000)
  filteredItems.value = allData[row.type]
  gotop()
}
let objectMap = {
  availableCount: {
    label: '可用次数',
    unit: '次',

  },
  activeTime: {
    label: '启用时间',
    unit: '秒'
  },
  capacity: {
    label: '容量',
    unit: '格'
  },
  type: {
    label: '类型',
  },
  propsSource: {
    label: '',
  },
  replyEffect: {},
  repairPoints: {
    label: '修复',
    unit: '点'
  },
  durability: {
    label: '耐久度',
    unit: '%'
  },
  repairEfficiency: {
    label: '修复效率',
  },
  activeTime: {
    label: '启用时间',
    unit: '秒',
    style: `    background-color: #ffcc81;
    color: #70500d;`
  }

}
function getRowTag(row) {
  let obj = {}
  if (row.propsDetail) {
    obj = row.propsDetail
  }
  if (row.protectDetail) {
    obj = row.protectDetail
  }
  let list = []

  if (row.secondClassCN) {
    list.push({
      name: row.secondClassCN,
    })
  }
  if (row.thirdClassCN) {
    list.push({
      name: row.thirdClassCN,
    })
  }
  Object.keys(obj).map(item => {
    if (objectMap[item]) {
      let name = (objectMap[item].label || '') + obj[item] + (objectMap[item].unit || '')
      name = name.replace(/秒秒/, '秒')
      list.push({
        name: name,
        style: objectMap[item].style || ''
      })
    }
  })
  return list
}

async function getJsonData(row) {
  let res = await fetch(row.path).then(response => {
    return response.json();
  })
  let list = res.jData.data.data.list
  allData[row.type] = list
  loadingJsName.value = `...`
  allList = allList.concat(list)
  loadJsCount.loaded += 1
  if (loadJsCount.loaded === loadJsCount.total) {
    loadCompleted()
  }
}
loadScript()
</script>
<style lang="scss">
.loadDtaa {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background-color: #000000c2;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99;

  .sb {
    z-index: 100;
    width: 400px;
    height: 200px;
    background-color: #FFFFFF;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 10px;
  }
}

.top {
  position: fixed;
  right: 20px;
  bottom: 20px;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  box-shadow: 1px 2px 6px #0000007d;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  background-color: #FFFFFF;
  transition: 0.3s;
  z-index: 3;

  &:hover {
    background-color: #2364c0;
    color: #FFFFFF;
  }
}

.deltaForceData {
  height: 100vh;
  background-image: linear-gradient(to right, #9bb3c3, #9eb0ca);

  .ner {
    background-color: #f4f4f4;

    .search {
      height: 200px;
      background-color: #000000;
      position: relative;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;

      .tt {
        color: #FFFFFF;
        font-weight: bold;
        position: absolute;
        top: 20px;
        font-size: 24px;
        color: #45f79b;
        font-weight: bold;
      }

      .line {
        position: absolute;
        width: 600px;
        display: flex;
        border-radius: 10px;
        background-color: #FFFFFF;
        height: 50px;
        overflow: hidden;
        padding-left: 20px;

        input {

          outline: none;
          width: 400px;
          border: none;
        }

        .sbtn {
          width: 120px;
          background-color: #3688fc;
          position: absolute;
          color: #FFFFFF;
          right: 5px;
          top: 5px;
          height: 40px;
          display: flex;
          align-items: center;
          justify-content: center;
          border-radius: 10px;
          transition: 0.3s ease-in-out;
          cursor: pointer;

          &:hover {
            background-color: #2364c0;
          }
        }
      }
    }

    .searchContent {
      display: flex;
      padding: 10px;

      .nav {
        position: sticky;
        top: 15px;
        bottom: 15px;
        width: 300px;
        flex-shrink: 0;
        background-color: #fff;
        border-radius: 16px;
        margin-right: 10px;
        padding: 20px;
        height: calc(100vh - 20px);
        overflow: auto;

        &>li {
          font-size: 14px;
          text-transform: uppercase;
          cursor: pointer;

          p {
            background-color: #FFFFFF;
            font-size: 12px;
            font-family: 'OPPO Sans Regular';
            border-radius: 5px;
            color: #a2a2a2;
            height: 30px;
            line-height: 30px;
            padding: 0px 20px;
          }

          .child {

            li {
              font-size: 14px;
              line-height: 18px;
              padding: 5px 20px;
              transition: 0.1s;
              border-radius: 5px;
              margin-bottom: 5px;

              &.act {
                background-color: #2e67fb;
                color: #fff;
              }

              &:hover {
                background-color: #d5d5d570;
                color: #0085ef;
              }
            }
          }
        }
      }

      .objectList {
        width: 100%;
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        /* 自适应列数 */
        grid-auto-rows: min-content;
        /* 行高自适应 */
        gap: 15px;

        /* 项目之间的间隙 */
        li {
          border-radius: 15px;
          overflow: hidden;
          padding: 20px;
          position: relative;
          max-width: 400px;
          transition: 0.5s;

          &:hover {
            box-shadow: 1px 3px 10px #0000001a;
          }

          &::before {
            content: '';
            position: absolute;
            left: 0;
            right: 0;
            top: 240px;
            bottom: 0;
            background-color: #fff;
            z-index: 1;
          }

          .taglist {
            display: flex;
            flex-wrap: wrap;
            gap: 5px;
            margin-bottom: 10px;
          }

          .tag {
            font-size: 12px;
            padding: 5px 10px;
            border-radius: 3px;
            font-family: '新宋体';
            background-color: #346bf826;
            color: #0f3caf;
          }

          .image {
            padding: 10px;
            box-sizing: content-box;
            display: flex;
            justify-content: center;
            align-items: center;
            margin-bottom: 20px;

            img {
              flex-shrink: 0;
              width: 200px;
              height: 200px;
              object-fit: contain;
              margin: 0 auto;
              filter: drop-shadow(13px 8px 14px rgba(0, 0, 0, 0.2));
            }
          }

          &.L6 {
            background-image: linear-gradient(to bottom, #97434c 0%, #fff 70%);
          }

          &.L5 {
            background-image: linear-gradient(to bottom, #8a5e3e 0%, #fff 70%);
          }

          &.L4 {
            background-image: linear-gradient(to bottom, #6c57a0 0%, #fff 70%);
          }

          &.L3 {
            background-image: linear-gradient(to bottom, #3a749f 0%, #fff 70%);
          }

          &.L2 {
            background-image: linear-gradient(to bottom, #238d6f 0%, #fff 70%);
          }

          &.L1 {
            background-image: linear-gradient(to bottom, #748084 0%, #fff 70%);
          }


          .info {
            position: relative;
            z-index: 2;

            h3 {
              font-size: 16px;
              margin-bottom: 5px;
            }

            p {
              font-size: 12px;
              font-family: "微软雅黑";
            }
          }
        }
      }
    }
  }

}

.filters {
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.itemColor {
  display: flex;
  flex-direction: column;
  justify-content: center;

  li {
    display: flex;
    width: 80px;
    height: 80px;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.2s;

    .checkitem {
      display: flex;
    }

    p {
      font-size: 12px;
      margin-top: 2px;
    }

    i {
      font-size: 22px;
    }

    &:hover {
      background-color: #0000000f;
    }
  }
}

.nav {
  --sb-track-color: #dadada;
  --sb-thumb-color: #6186b1;
  --sb-size: 3px;
}

.nav::-webkit-scrollbar {
  width: var(--sb-size)
}

.nav::-webkit-scrollbar-track {
  background: var(--sb-track-color);
  border-radius: 3px;
}

.nav::-webkit-scrollbar-thumb {
  background: var(--sb-thumb-color);
  border-radius: 3px;

}

@supports not selector(::-webkit-scrollbar) {
  .nav {
    scrollbar-color: var(--sb-thumb-color) var(--sb-track-color);
  }
}
/* 手机布局适配：保留原有配色与卡片风格 */
@media (max-width: 768px) {
  .deltaForceData {
    min-height: 100vh;
    height: auto;
    background-image: linear-gradient(to right, #9bb3c3, #9eb0ca);
  }

  .deltaForceData .ner .search {
    height: 150px;
    padding: 12px;
  }

  .deltaForceData .ner .search .tt {
    top: 18px;
    font-size: 21px;
    text-align: center;
  }

  .deltaForceData .ner .search .line {
    width: 100%;
    max-width: 600px;
    height: 46px;
    padding-left: 12px;
  }

  .deltaForceData .ner .search .line input {
    width: 100%;
    min-width: 0;
    padding-right: 80px;
  }

  .deltaForceData .ner .search .line .sbtn {
    width: 68px;
    height: 36px;
    top: 5px;
    right: 5px;
  }

  .deltaForceData .ner .searchContent {
    display: block;
    padding: 8px;
  }

  .deltaForceData .ner .searchContent .nav {
    position: static;
    width: 100%;
    height: auto;
    max-height: none;
    margin: 0 0 10px;
    padding: 10px;
    display: flex;
    gap: 14px;
    overflow-x: auto;
    overflow-y: hidden;
    border-radius: 10px;
  }

  .deltaForceData .ner .searchContent .nav > li {
    flex: 0 0 auto;
    min-width: max-content;
  }

  .deltaForceData .ner .searchContent .nav > li p {
    padding: 0 8px;
    height: 26px;
    line-height: 26px;
  }

  .deltaForceData .ner .searchContent .nav .child {
    display: flex;
    gap: 5px;
  }

  .deltaForceData .ner .searchContent .nav .child li {
    white-space: nowrap;
    padding: 7px 10px;
    margin: 0;
  }

  .deltaForceData .ner .searchContent .filters {
    height: 44px;
  }

  .deltaForceData .ner .searchContent .objectList {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 8px;
  }

  .deltaForceData .ner .searchContent .objectList > li {
    width: 100%;
    max-width: none;
    min-width: 0;
    padding: 10px;
    border-radius: 10px;
  }

  .deltaForceData .ner .searchContent .objectList > li::before {
    top: 125px;
  }

  .deltaForceData .ner .searchContent .objectList > li .image {
    padding: 4px;
    margin-bottom: 8px;
  }

  .deltaForceData .ner .searchContent .objectList > li .image img {
    width: 100%;
    max-width: 120px;
    height: auto;
    aspect-ratio: 1 / 1;
  }

  .deltaForceData .ner .searchContent .objectList > li .info h3 {
    font-size: 14px;
    line-height: 1.5;
    overflow-wrap: anywhere;
  }

  .deltaForceData .ner .searchContent .objectList > li .info p {
    font-size: 12px;
    line-height: 1.6;
    overflow-wrap: anywhere;
  }

  .deltaForceData .ner .searchContent .objectList > li .taglist {
    gap: 4px;
  }

  .deltaForceData .ner .searchContent .objectList > li .tag {
    font-size: 11px;
    padding: 3px 6px;
  }

  .loadDtaa .sb {
    width: calc(100% - 40px);
    max-width: 400px;
  }

  .top {
    width: 46px;
    height: 46px;
    right: 12px;
    bottom: 16px;
  }
}

@media (max-width: 360px) {
  .deltaForceData .ner .searchContent .objectList {
    grid-template-columns: minmax(0, 1fr);
  }
}

</style>