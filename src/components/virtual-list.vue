<template>
  <div>
    {{ startIndex }}
    {{ endIndex }}
    <el-table ref="table" :max-height="maxHeight" :data="visibleList" style="width: 100%">
      <slot></slot>
    </el-table>
  </div>
</template>

<script>
import { throttle, debounce } from '../utils'
export default {
  name: 'virtual-list',
  props: {
    list: {
      type: Array,
      default: () => []
    },
    maxHeight: {
      type: [String, Number],
      default: 500
    },
    visibleCount: {
      type: Number,
      default: 10
    },
    maxLen: {
      // 每次获取数据最大长度
      type: Number,
      default: 20
    },
    total: {
      type: Number,
      default: 10
    }
  },
  data() {
    return {
      startIndex: 0,
      tcEl: null,
      tbEl: null,
      scrollTopArr: [],
      scrollDirection: 'bottom'
    }
  },
  computed: {
    endIndex() {
      const { startIndex, visibleCount, list } = this
      if (!Array.isArray(list)) return 0
      return Math.min(startIndex + visibleCount, list.length)
    },
    visibleList() {
      const { startIndex, endIndex, list } = this
      const res = list.slice(startIndex, endIndex)
      res.forEach((item, index) => {
        item.$index = startIndex + index + 1
      })
      return res
    }
  },
  mounted() {
    this.initElAndEvent()
  },
  watch: {
    startIndex(newVal, oldVal) {
      const { scrollDirection } = this
      if (scrollDirection === 'bottom' && newVal < oldVal)
        this.scrollDirection = 'top'
      else if (scrollDirection === 'top' && newVal > oldVal)
        this.scrollDirection = 'bottom'
    }
  },
  methods: {
    initElAndEvent() {
      const { $refs } = this
      const tableEl = ($refs['table'] && $refs['table'].$el) || null
      if (!tableEl) return false
      this.tcEl = tableEl.querySelector('.el-table__body-wrapper') || null
      this.tbEl =
        tableEl.querySelector('.el-table__body-wrapper .el-table__body') || null
      if (!this.tbEl) return false
      this.initScrollTopArr()
      this.tcEl.addEventListener('scroll', this.handleScroll)
      return true
    },
    settbElPadding() {
      const {
        startIndex,
        endIndex,
        total,
        scrollTopArr,
        tbEl,
        scrollDirection
      } = this
      const { length } = scrollTopArr
      const pt = startIndex > 0 ? scrollTopArr[startIndex - 1] : 0
      const pb =
        endIndex < total
          ? scrollTopArr[endIndex - 1] - scrollTopArr[endIndex - 2]
          : 0
      tbEl.style.paddingTop = `${pt}px`
      tbEl.style.paddingBottom = `${pb}px`
    },
    initScrollTopArr() {
      this.$nextTick(function() {
        const { scrollTopArr, startIndex, endIndex } = this
        const { length: len } = scrollTopArr
        if (endIndex >= len) {
          const si = len - startIndex || 0
          const rowArr = this.tbEl.querySelectorAll('.el-table__row') || []
          let height = scrollTopArr[len - 1] || 0
          for (let i = si; i < rowArr.length; i++) {
            height += rowArr[i].clientHeight
            scrollTopArr.push(height)
          }
        }
        this.settbElPadding()
      })
    },
    handleScroll: throttle(function() {
      const { tcEl, scrollTopArr, endIndex, total, visibleCount, maxLen } = this
      if (!scrollTopArr.length) {
        this.initScrollTopArr()
        return
      }
      const st = tcEl.scrollTop
      this.startIndex = scrollTopArr.findIndex(top => st < top)
      this.initScrollTopArr()
      if (endIndex < total && endIndex > total - visibleCount / 2)
        this.$emit('fetch')
    }, 60)
  }
}
</script>

<style>
</style>