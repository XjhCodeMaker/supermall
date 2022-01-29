<template>
  <div class="bottom-bar">
    <div class="check-content">
      <check-button :is-checked="isSelectAll"
                    class="check-button-cpn" @click.native="checkClick"/>
      <span>全选</span>
    </div>

    <div class="price">
      合计: {{totalPrice}}
    </div>

    <div class="calculate" @click="calcClick">
      去计算({{checkLength}})
    </div>

  </div>
</template>

<script>
import CheckButton from "@/components/content/checkButton/CheckButton";
import {mapGetters} from "vuex";

export default {
  name: "CartBottomBar",
  components: {
    CheckButton
  },
  computed: {
    ...mapGetters(['cartList','cartLength']),
    totalPrice() {
      return '￥' + this.cartList.filter(item => {
        return item.checked == true
      }).reduce((preValue, item) => {
        return item.price * item.count + preValue
      }, 0).toFixed(2)
    },
    checkLength() {
      return this.cartList.filter(item => item.checked == true).length
    },
    isSelectAll() {
      // return this.checkLength === this.cartLength

      //使用filter
      // return !(this.cartList.filter(item => !item.checked).length)
      if(this.cartList.length==0) {
        return false
      }
      //使用find
      // return !this.cartList.find(item => !item.checked)

      //使用普遍历
      for(let item of this.cartList) {
        if(!item.checked) {
          return false
        }
      }

      return true
    }
  },
  methods: {
    checkClick() {
      if(this.isSelectAll) { //全部选中
        this.cartList.forEach(item => item.checked = false)
      }else { //有部分或全部不选中
        this.cartList.forEach(item => item.checked = true)
      }

      //这样写是错误的，因在遍历过程中其中的checked发生变化
      //而导致this.isSelectAll也在变化
      // this.cartList.forEach(item => item.checked = !this.isSelectAll)

    },
    calcClick() {
      if(!this.isSelectAll) {
        this.$toast.show('请选择购买的商品', 2000)
      }
    }
  }
}
</script>

<style scoped>
  .bottom-bar {
    position: relative;
    display: flex;

    height: 40px;
    line-height: 40px;

    background-color: #eee;

    font-size: 14px;
  }

  .check-content {
    display: flex;
    align-items: center;
    margin-left: 10px;
    width: 60px;
  }

  .check-button-cpn {
    width: 20px;
    height: 20px;
    line-height: 20px;
    margin-right: 5px;
  }

  .price {
    margin-left: 20px;
    flex: 1;
  }

  .calculate {
    width: 100px;
    background-color: orange;
    color: #fff;
    text-align: center;
  }
</style>
