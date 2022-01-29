<template>
	<div class="cart">
    <!-- 导航  -->
    <nav-bar class="cart-nav-bar">
      <template v-slot:center><div>购物车({{length}})</div></template>
    </nav-bar>
    <!--商品列表-->
    <cart-list/>
    <!--底部汇总-->
    <cart-bottom-bar/>
	</div>
</template>

<script>
  import NavBar from "@/components/common/navbar/NavBar";

  import CartList from "@/views/cart/childComps/CartList";
  import CartBottomBar from "@/views/cart/childComps/CartBottomBar";

  import {mapGetters} from 'vuex'

	export default {
		name: "Cart",
    components: {
      NavBar,
      CartList,
      CartBottomBar
    },
    computed: {
      //两种用法
      // ...mapGetters(['cartLength','cartList'])
      ...mapGetters({
        length: 'cartLength'
      })
    }
	}
</script>

<style>
  .cart {
    height: 100vh;
  }

 .cart-nav-bar {
   background-color: var(--color-tint);
   color: #fff;
   font-weight: 700;
 }
</style>
