<template>
  <div id="appHome">
    <div class="top-box">
      <div class="pagesList-box">
        <div :class="['content-box','content-box-'+index]"
             v-for="(page,index) in appPageList"
             :key="index"
             :id="'itxst-'+index">
          <div class="app-item-box"
               v-for="(item,appIndex) in page.list"
               :key="item.id"
               :data-id="item.id">
            <div class="logo-box"
                 @mousedown="mousedown"
                 @mouseup="mouseup">
              <!-- <img :class="['logo-item',isEdit?'':'isCur']"
                   :src="item.src"
                   alt="" /> -->
              <div :class="['logo-item',isEdit?'':'isCur']"
                   :style="'background-color:'+item.color"></div>
              <img v-if="isEdit"
                   class="dele-item isCur"
                   src="../assets/dele_icon.png"
                   @click="deleApp(item,index,appIndex)"
                   alt="" />
            </div>
            <div class="app-name">{{item.name}}</div>
          </div>
        </div>
      </div>

      <div :class="['left-box','icon-box',listQuery.pageNum>1?'isCur icon-isCur':'noCur']"
           @click="iconClick(0)">
        《
      </div>
      <div :class="['right-box','icon-box',listQuery.pageNum<listQuery.pageTotal?'isCur icon-isCur':'noCur']"
           @click="iconClick(1)">
        》
      </div>
    </div>
    <div class="page-box">
      <div class="foot-box">
        <div :class="['pot-item',foot.status?'pot-current':'pot-other']"
             v-for="foot in listQuery.pagePot"
             @click="potClick(foot)"></div>
      </div>
      <el-button class="closeEdit-box"
                 v-if="isEdit"
                 plain
                 @click="closeEdit">取消编辑</el-button>
    </div>
  </div>
</template>

<script>

import Sortable from 'sortablejs';
export default {
  name: 'appHome',
  data () {
    return {
      defaultAppList: [],// 初始化数据
      appPageList: [],
      mouseEvent: null, // 长按计时器
      mouseTime: null, // 长按时间
      isEdit: false,
      clickOne: null, // 点击项,避免非编辑状态下的拖拽导致进入编辑状态
      listQuery: {
        pageNum: 1,
        pageSize: 30,
        total: 0,
        pageTotal: 0,
        pagePot: []
      },
      // 控制动画
      appDoms: [],
      aniKeyframes: [
        {
          transform: 'scale(1)',
        },
        {
          offset: 0.05,
          transform: 'rotate(-4.5deg)',
        },
        {
          offset: 0.2,
          transform: 'rotate(-5deg)',
        },
        {
          offset: 0.35,
          transform: 'rotate(-4.5deg)',
        },
        {
          offset: 0.4,
          transform: 'rotate(0)',
        },
        {
          offset: 0.5,
          transform: 'rotate(5deg)',
        },
        {
          offset: 0.55,
          transform: 'rotate(4.5deg)',
        },
        {
          offset: 0.7,
          transform: 'rotate(5deg)',
        },
        {
          offset: 0.85,
          transform: 'rotate(4.5deg)',
        },
        {
          offset: 0.9,
          transform: 'rotate(5deg)',
        },
        {
          transform: 'rotate(0)',
        }
      ],
      aniOption: {
        duration: 200,
        iterations: 'Infinity'
      }
    }
  },
  computed: {
  },
  mounted () {
    this.$nextTick(() => {
      this.init()
    })
  },
  methods: {
    init () {
      this.initDedaultList()
      // 初始化appPageList和基础数据
      const _appPageList = JSON.parse(sessionStorage.getItem('appPageList'))
      if (_appPageList && _appPageList.length > 0) {
        this.appPageList = _appPageList
        let _appList = []
        _appList = _appList.concat(..._appPageList)
        this.listQuery.total = _appList.length
        this.listQuery.pageTotal = _appPageList.length
      } else {
        this.listQuery.total = this.defaultAppList.length
        this.listQuery.pageTotal = Math.ceil(this.listQuery.total / this.listQuery.pageSize)
        for (let i = 0; i < this.listQuery.pageTotal; i++) {
          this.appPageList[i] = {
            list: this.defaultAppList.slice((this.listQuery.pageNum + i - 1) * 30, (this.listQuery.pageNum + i) * 30)
          }
        }
      }
      // 初始化box宽度
      let _pageBox = document.getElementsByClassName('pagesList-box')
      _pageBox[0].style.width = `${this.listQuery.pageTotal * 100}%`
      // console.log(this.appPageList)
      // 初始化下方点
      for (let i = 0; i < this.listQuery.pageTotal; i++) {
        if (i + 1 === this.listQuery.pageNum) {
          this.listQuery.pagePot.push({
            id: i,
            status: true
          })
        } else {
          this.listQuery.pagePot.push({
            id: i,
            status: false
          })
        }
      }
      this.setSession()
    },
    initDedaultList () {
      let _list = []
      let range = ['8', '9', 'A', 'B', 'C', 'D', 'E', 'F']
      for (let i = 0; i < 66; i++) {
        let _col = '#'
        for (let j = 0; j < 6; j++) {
          let _i = Math.floor(range.length * Math.random())
          _col += range[_i]
        }
        _list.push({
          id: i + 1,
          name: 'app-' + (i + 1),
          src: require('../assets/app-logo.jpg'),
          color: _col
        })
      }
      this.defaultAppList = _list
    },
    setSession () {
      // console.log(this.appPageList)
      let _appPageList = this.appPageList.map(i => {
        let _arr = []
        i.list.forEach(item => {
          if (item) {
            _arr.push(item)
          }
        })
        return { list: _arr }
      })
      // console.log(_appPageList);
      sessionStorage.setItem('appPageList', JSON.stringify(_appPageList))
    },
    // 左右切换页面
    iconClick (val) {
      this.listQuery.pageNum = val ? this.listQuery.pageNum + 1 : this.listQuery.pageNum - 1
      this.pageNumChange()
    },
    // 下方点切换页面
    potClick (val) {
      this.listQuery.pageNum = val.id + 1
      this.pageNumChange()
    },
    // 切换页面
    pageNumChange () {
      let pageDoms = document.getElementsByClassName('content-box')
      // console.log(pageDoms)
      for (let i = 0; i < this.listQuery.pageTotal; i++) {
        pageDoms[i].style.transform = `translateX(-${(this.listQuery.pageNum - 1) * 100}%)`
      }
      // 切换原点
      let _id = this.listQuery.pageNum - 1
      this.listQuery.pagePot.forEach((item, i) => {
        if (item.id === _id) {
          item.status = true
        } else {
          item.status = false
        }
      })
    },
    // 长按开始
    mousedown () {
      // 按下鼠标，开始计时
      if (this.isEdit) {
        return
      } else {
        this.mouseEvent = setInterval(() => {
          this.mouseTime += 500
          if (this.mouseTime >= 1500) {
            this.editChange()
            clearInterval(this.mouseEvent)
          }
        }, 500);
      }
    },
    // 长按结束
    mouseup () {
      // 松开鼠标
      clearInterval(this.mouseEvent)
    },
    // 进入编辑状态
    editChange () {
      clearInterval(this.mouseEvent)
      this.isEdit = true
      let _appDoms
      let _state = false
      _appDoms = document.getElementsByClassName('logo-item')
      // console.log(_appDoms)       
      if (this.appDoms.length === 0) {
        _state = true
      }
      _appDoms.forEach((app, index) => {
        if (_state) {
          let _app = {}
          _app.myAnimation = app.animate(this.aniKeyframes, this.aniOption);
          this.appDoms.push(_app)
        }
        this.appDoms[index].myAnimation.play()
      })
      this.initSort()
    },
    // 关闭编辑状态
    closeEdit () {
      this.isEdit = false
      this.appDoms.forEach(app => {
        app.myAnimation.cancel()
      })
    },
    // 删除app
    deleApp (item, index, appIndex) {
      this.appPageList[index].list.splice(appIndex, 1)
      this.$set(this.appPageList, index, this.appPageList[index])
      console.log(this.appPageList)
      this.setSession()
      let _state = false
      this.appPageList.forEach((list, index) => {
        if (list.sort) {
          _state = true
          list.sort.destroy()
        }
      })
      if (_state) {
        this.$nextTick(() => {
          this.initSort()
        })
      }
    },
    // 初始化拖拽
    initSort () {
      let _this = this
      this.appPageList.forEach((list, index) => {
        let _id = 'itxst-' + index
        let _el = document.getElementById(_id)
        list.sort = new Sortable(_el, {
          group: "name-" + index,
          //是否允许列内部排序，如果为false当有多个排序组时,多个组之间可以拖拽，本身不能拖拽
          sort: true,
          // 按下鼠标后多久可以拖拽 1000表示1秒
          delay: 0,
          //动画效果
          animation: 500,
          draggable: '.app-item-box',
          handle: '.logo-box',
          easing: "cubic-bezier(1, 0, 0, 1)",
          //指定获取拖动后排序的data属性
          dataIdAttr: 'data-id',
          //忽略HTML5原生拖拽行为
          forceFallback: false,

          // 列表内元素顺序更新的时候触发
          onUpdate: function (/**Event*/evt) {
            // console.log(list)
            var arr = list.sort.toArray()
            // console.log(arr)
            let _app = arr.map((item, i) => {
              let _item
              _this.appPageList[index].list.forEach((list) => {
                if (list.id === parseInt(item)) {
                  _item = list
                }
              })
              return _item
            })
            _this.appPageList[index].list = _app
            _this.setSession()
          },
        })
      })
    },
  }
}
</script>

<style lang="scss" scoped>
$iconWid: 40px;
#appHome {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  background-color: #333;
  .noCur {
    cursor: not-allowed;
  }
  .isCur {
    cursor: pointer;
  }
  .top-box {
    position: relative;
    width: 100%;
    height: 90%;
    overflow: hidden;
    .pagesList-box {
      position: relative;
      // width: 300%;
      height: 100%;
      display: flex;
      flex-direction: row;
      justify-content: flex-start;
      align-items: center;
      .content-box {
        position: relative;
        padding: 12px $iconWid;
        width: 100%;
        height: 100%;
        display: grid;
        grid-row-gap: 20px;
        grid-column-gap: 20px;
        transition: 0.5s all ease-out;
        @media (min-width: 0px) {
          grid-template-columns: repeat(5, 1fr);
          grid-template-rows: repeat(6, 1fr);
        }
        @media (min-width: 1000px) {
          grid-template-columns: repeat(6, 1fr);
          grid-template-rows: repeat(5, 1fr);
        }
        .app-item-box {
          position: relative;
          width: 100%;
          height: 100%;
          // background-color: #fff;
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          transition: 0.5s all ease-out;
          .logo-box {
            background-color: transparent;
            position: relative;
            width: 36%;
            min-width: 45px;
            min-height: 45px;
            padding-top: 36%;
            // overflow: hidden;
            margin-bottom: 10px;
            .logo-item {
              position: absolute;
              width: 100%;
              height: 100%;
              min-width: 45px;
              min-height: 45px;
              border-radius: 20%;
              top: 0;
              left: 0;
              // animation: shake 1.5s infinite;
              // -webkit-animation: shake 1.5s infinite;
              // animation-play-state: paused;
              // -webkit-animation-play-state: paused;
            }
            .dele-item {
              position: absolute;
              width: 18px;
              height: 18px;
              top: -6px;
              right: -6px;
            }
          }
          .app-name {
            font-size: 16px;
            color: #d6d6d6;
          }
        }
      }
    }

    .icon-box {
      position: absolute;
      width: $iconWid;
      height: $iconWid;
      top: 50%;
      transform: translateY(-50%);
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      font-size: 24px;
      color: #a5a5a5;
    }
    .left-box {
      left: 0;
    }
    .right-box {
      right: 0;
    }
    .icon-isCur {
      cursor: pointer;
      &:hover {
        color: aliceblue;
      }
    }
  }
  .page-box {
    position: relative;
    width: 100%;
    height: 10%;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    .foot-box {
      position: relative;
      width: auto;
      height: auto;
      display: flex;
      flex-direction: row;
      justify-content: flex-start;
      align-items: center;
      padding: 12px;
      .pot-item {
        cursor: pointer;
        position: relative;
        height: 10px;
        border-radius: 5px;
        border: 1px solid #999;
        transition: 0.2s all ease-in-out;
        box-shadow: 1px 1px 6px #888;
      }
      .pot-item + .pot-item {
        margin-left: 6px;
      }
      .pot-current {
        width: 24px;
        background-color: #c4c4c4;
      }
      .pot-other {
        width: 10px;
        background-color: #e6e6e6;
      }
    }
    .closeEdit-box {
      padding: 6px;
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      right: 36px;
      font-size: 14px;
    }
  }
}
@keyframes shake {
  0% {
    transform: scale(1);
  }

  5% {
    transform: scale(1) rotate(-4.5deg);
  }

  20% {
    transform: scale(1) rotate(-5deg);
  }

  35% {
    transform: scale(1) rotate(-4.5deg);
  }

  40% {
    transform: scale(1) rotate(0);
  }

  50% {
    transform: scale(1) rotate(-5deg);
  }

  80% {
    transform: scale(1) rotate(5deg);
  }

  100% {
    transform: scale(1) rotate(0);
  }
}
</style>
