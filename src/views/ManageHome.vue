<template>
  <div class="ops-manage-home">
    <div class="ops-manage-home-header">
      <span>图片管理系统</span>
    </div>
    <div class="ops-manage-home-cantainer">
      <div class="ops-manage-home-aside">
        <ul class="aside_ul">
          <li class="aside_li">图片</li>
        </ul>
      </div>
      <div class="ops-manage-home-main">
        <el-table :data="showData" stripe style="width: 100%">
          <el-table-column
            align="center"
            header-align="center"
            prop="article_id"
            label="article_id"
            width="180"
          ></el-table-column>
          <el-table-column align="center" header-align="center" prop="title" label="标题" width="180"></el-table-column>
          <el-table-column align="center" header-align="center" prop="describe" label="图片描述"></el-table-column>
          <el-table-column align="center" header-align="center" label="图片">
            <template slot-scope="scope">
              <img :src="imgBaseUrl + scope.row.img.img_url" width="100" height="100" />
            </template>
          </el-table-column>
          <el-table-column align="center" header-align="center" label="操作">
            <template slot-scope="scope">
              <el-button v-show="scope.row.isShow" @click="accept(scope)" type="success">通过</el-button>
              <el-button v-show="scope.row.isShow" @click="reject(scope)" type="danger">拒绝</el-button>
              <el-button v-show="scope.row.isShow" @click="remove(scope)" type="warning">删除</el-button>
              <el-button @click="topping(scope)" type="info">置顶</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          background
          hide-on-single-page
          :current-page.sync="curPage"
          :page-size="pageSize"
          layout="prev, pager, next"
          :total="allDataCount"
          @prev-click="prevPage"
          @next-click="nextPage"
          @current-change="currentPage"
        ></el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import axios from "axios";
export default {
  name: "Home",
  data() {
    return {
      imgBaseUrl: "http://localhost:3000",
      allDataCount: 0,
      curPage: 1,
      pageSize: 5,
      showData: [],
      totalData: [],
      allData: [],
      isShow1: true
    };
  },
  created() {
    this.getInitData();
  },
  mounted() {
    this.initData();
  },
  methods: {
    initData() {
      let aside = document.querySelector(".ops-manage-home-aside");
      let homeMain = document.querySelector(".ops-manage-home-main");
      aside.style.height = window.innerHeight - 60 + "px";
      homeMain.style.height = window.innerHeight - 80 + "px";
      window.addEventListener("resize", () => {
        aside.style.height = window.innerHeight - 60 + "px";
        homeMain.style.height = window.innerHeight - 80 + "px";
      });
    },
    async getInitData() {
      await axios({
        method: "get",
        url: "http://localhost:3000/admin"
      }).then(res => {
        this.allData = res.data.articleData;
        let count = 0;
        for (let v of this.allData) {
          count += v.imgs.length;
          for (let i = 0; i < v.imgs.length; i++) {
            this.totalData.push({
              article_id: v.article_id,
              title: v.title,
              describe: v.describe,
              img: {
                img_id: v.imgs[i].img_id,
                img_url: v.imgs[i].img_url
              },
              isShow: true
            });
          }
        }
        this.allDataCount = count;
        this.showData = this.totalData.slice(0, this.pageSize);
      });
    },
    prevPage() {
      console.log("上一页");
      console.log(this.curPage);
      this.showData = this.totalData.slice(
        (this.curPage - 2) * this.pageSize,
        (this.curPage - 1) * this.pageSize
      );
    },
    nextPage() {
      this.showData = this.totalData.slice(
        this.curPage * this.pageSize,
        (this.curPage + 1) * this.pageSize
      );
    },
    currentPage() {
      this.showData = this.totalData.slice(
        (this.curPage - 1) * this.pageSize,
        this.curPage * this.pageSize
      );
    },
    accept(scope) {
      this.showData[scope.$index].isShow = false;
    },
    reject(scope) {
      this.goRemove(scope);
    },
    remove(scope) {
      this.goRemove(scope);
    },
    topping(scope) {
      for (let i = 0; i < this.allDataCount; i++) {
        if (
          this.totalData[i].article_id === scope.row.article_id &&
          this.totalData[i].img.img_id === scope.row.img.img_id
        ) {
          let tmp=this.totalData.splice(i, 1)
          this.totalData=tmp.concat(this.totalData);
          this.showData = this.totalData.slice(
            (this.curPage - 1) * this.pageSize,
            this.curPage * this.pageSize
          );
          break;
        }
      }
    },
    async goRemove(scope) {
      for (let i = 0; i < this.allData.length; i++) {
        if (this.allData[i].article_id === scope.row.article_id) {
          if (this.allData[i].imgs.length === 1) {
            await axios({
              method: "delete",
              url: "http://localhost:3000/admin/delete/" + scope.row.article_id
            }).then(res => {
              console.log(res.data);
            });
          } else {
            for (let j = 0; j < this.allData[i].imgs.length; j++) {
              if (this.allData[i].imgs[j].img_id === scope.row.img.img_id) {
                this.allData[i].imgs.splice(j, 1);
                break;
              }
            }
            await axios({
              method: "put",
              url: "http://localhost:3000/admin/put",
              data: {
                article_id:scope.row.article_id,
                imgs: this.allData[i].imgs
              }
            }).then(res => {
              console.log(res.data);
            });
          }
          break;
        }
      }
      for (let i = 0; i < this.allDataCount; i++) {
        if (
          this.totalData[i].article_id === scope.row.article_id &&
          this.totalData[i].img.img_id === scope.row.img.img_id
        ) {

          this.allDataCount--;
          this.totalData.splice(i, 1);
          this.showData = this.totalData.slice(
            (this.curPage - 1) * this.pageSize,
            this.curPage * this.pageSize
          );
          break;
        }
      }
    }
  }
};
</script>

<style lang="less" scoped>
.ops-manage-home {
  width: 100%;
  .ops-manage-home-header {
    width: 100%;
    position: fixed;
    top: 0;
    height: 60px;
    z-index: 999;
    text-align: center;
    background-color: #409eff;
    span {
      padding: 0 10px;
      line-height: 60px;
    }
  }
  .ops-manage-home-cantainer {
    margin-top: 60px;
    display: flex;
    .ops-manage-home-aside {
      width: 200px;
      background-color: #606266;
      .aside_ul {
        .aside_li {
          width: 200px;
          height: 60px;
          color: #fff;
          line-height: 60px;
          border-bottom: 1px solid #fff;
        }
      }
    }
    .ops-manage-home-main {
      padding: 20px 50px 0 50px;
      overflow: auto;
      flex: 1;
      .el-pagination {
        margin-top: 20px;
      }
    }
  }
}
</style>
