<template>
  <div class="swp">
    <div class="swp-header">
      <span class="swp-header_left">{{ now + 1 }} / {{ imgs.length }}</span>
      <span class="swp-header_right" @click="goArticle">返回</span>
    </div>
    <div class="swp-main" ref="swpmain">
      <div class="img_card" v-for="item in sortedImgs" :key="item.id">
        <div class="img_card_item">
          <img :src="imgBaseUrl + item.img.img_url" alt />
        </div>
      </div>
    </div>
    <div v-show="left" @click="goLeft" class="swp_left">左箭头</div>
    <div v-show="right" @click="goRight" class="swp_right">右箭头</div>
  </div>
</template>

<script>
export default {
  props: {
    imgId: Number,
    imgs: Array,
    articleId: Number
  },
  data() {
    return {
      imgBaseUrl: "http://localhost:3000",
      sortedImgs: [],
      pre: 0,
      now: 0,
      next: 0,
      count: 0,
      clientWidth: 0,
      left: false,
      right: false
    };
  },
  created() {
    this.sortData();
    this.count = this.imgs.length;
  },
  mounted() {
    this.initData();
  },
  methods: {
    sortData() {
      let imgIndex = 0;
      for (let i = 0; i < this.imgs.length; i++) {
        if (this.imgs[i].article_id === this.articleId) {
          if (this.imgs[i].img.img_id === this.imgId) {
            imgIndex = i;
            break;
          }
        }
      }
      this.sortedImgs = this.sortedImgs.concat(this.imgs.slice(imgIndex));
      this.sortedImgs = this.sortedImgs.concat(this.imgs.slice(0, imgIndex));
    },
    initData() {
      this.clientWidth = document.querySelectorAll(".img_card")[0].clientWidth;
      const imgCard = document.querySelectorAll(".img_card");
      if (this.count === 1) {
        this.now = 0;
        imgCard[this.now].style.transform = "translateX(0px)";
      } else if (this.count === 2) {
        this.left = true;
        this.now = 0;
        this.next = 1;
        imgCard[this.now].style.transform = "translateX(0px)";
        imgCard[this.next].style.transform =
          "translateX(" + this.clientWidth + "px)";
      } else {
        this.left = true;
        this.right = true;
        this.pre = imgCard.length - 1;
        this.now = 0;
        this.next = 1;
        imgCard[this.pre].style.transform =
          "translateX(" + -this.clientWidth + "px)";
        imgCard[this.now].style.transform = "translateX(0px)";
        imgCard[this.next].style.transform =
          "translateX(" + this.clientWidth + "px)";
      }
    },
    goLeft() {
      console.log("左箭头被点击了1");

      const imgCard = document.querySelectorAll(".img_card");
      this.showPre(imgCard);
      if (this.count === 2 && this.now > this.next) {
        this.left = false;
        this.right = true;
      }
    },
    goRight() {
      console.log("右箭头被点击了2");
      const imgCard = document.querySelectorAll(".img_card");
      this.showNext(imgCard);
      if (this.count === 2 && this.pre >= this.now) {
        this.left = true;
        this.right = false;
      }
    },
    //往左走
    showPre(imgCard) {
      //   imgCard[this.pre].style.transition = "none";
      imgCard[this.pre].style.transform =
        "translateX(" + this.clientWidth * 2 + "px)";
      this.pre = this.now;
      this.now = this.next;
      this.next++;
      if (this.next > imgCard.length - 1) {
        this.next = 0;
      }
      this.setTransformLeft(imgCard);
    },
    // 往右走
    showNext(imgCard) {
      //   imgCard[this.next].style.transition = "none";
      imgCard[this.next].style.transform =
        "translateX(" + 2 * this.clientWidth + "px)";
      this.next = this.now;
      this.now = this.pre;
      this.pre--;
      if (this.pre < 0) {
        this.pre = imgCard.length - 1;
      }
      this.setTransformRight(imgCard);
    },
    // 变换函数
    setTransformLeft(imgCard) {
      imgCard[this.pre].style.transform =
        "translateX(" + -this.clientWidth + "px)";
      imgCard[this.now].style.transform = "translateX(0px)";
      imgCard[this.next].style.transform =
        "translateX(" + this.clientWidth + "px)";
    },
    setTransformRight(imgCard) {
      imgCard[this.pre].style.transform =
        "translateX(" + -this.clientWidth + "px)";
      imgCard[this.now].style.transform = "translateX(0px)";
      imgCard[this.next].style.transform =
        "translateX(" + this.clientWidth + "px)";
    },
    goArticle() {
      this.$emit("change");
    }
  }
};
</script>

<style scoped lang="less">
.swp {
  position: fixed;
  background-color: rgba(0, 0, 0, 0.3);
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 999;
  .swp-header {
    background-color: #000;
    height: 50px;
    line-height: 50px;
    display: flex;
    .swp-header_left {
      flex: 1;
      color: #fff;
      line-height: 50px;
      text-align: center;
    }
    .swp-header_right {
      color: #fff;
      line-height: 50px;
      float: right;
      margin-right: 30px;
    }
  }
  .swp-main {
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.3);
    overflow: hidden;
    position: relative;
    .img_card {
      width: 80%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 10%;
      transform: translate(200%);
      .img_card_item {
        width: 80%;
        height: 100%;
        margin-left: 10%;
        display: flex;
        justify-content: center;
        align-items: center;
        img {
          transform: scale(1);
          max-width: 100%;
          max-height: 100%;
        }
      }
    }
  }
  .swp_left,
  .swp_right {
    background-color: rgba(255, 255, 255, 0.8);
    border-radius: 50%;
    width: 60px;
    height: 60px;
    text-align: center;
    line-height: 60px;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
  }
  .swp_left {
    left: 50px;
  }
  .swp_right {
    right: 50px;
  }
}
</style>
