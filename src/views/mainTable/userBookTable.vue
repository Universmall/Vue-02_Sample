<template>
  <div>
    <!-- 分页栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="padding: 5px 0">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>书籍借阅</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 查询框 -->
    <div style="padding: 10px 0">
      <el-input style="width: 200px;margin-right: 5px" placeholder="条形码索书号"
                prefix-icon="el-icon-document-copy" v-model="bookID"></el-input>
      <el-button type="primary" @click="findOne" style="margin-right: 25px">精确搜索</el-button>

      <el-input style="width: 450px;margin-right: 5px" placeholder="支持 作品名 / 作者 / 出版商 / ISBN号 查询"
                prefix-icon="el-icon-search" @keyup.enter.native="fuzzySearch" v-model="bookInfo"></el-input>
      <el-button type="primary" @click="fuzzySearch"><i class="el-icon-search preIcon" />模糊搜索</el-button>
      <el-button type="primary" @click="load">重置</el-button>
    </div>

    <!-- 表格内容 -->
    <el-table :data="tableData" border stripe v-loading="loading" >
      <el-table-column prop="book_name" label="书籍名称" width="140" />
      <el-table-column prop="author" label="作者" width="140" />
      <el-table-column prop="publisher" label="出版商" />
      <el-table-column prop="isbn" label="ISBN号" />
      <el-table-column prop="book_id" label="索书号" />
      <el-table-column prop="amount" label="可借阅数" width="70"></el-table-column>
      <el-table-column>
        <template slot-scope="scope">
          <el-button type="primary" @click="infoId = scope.row.book_id; infoDialog = true;">详情</el-button>
          <el-button type="success" @click="borrow(scope.row)"><i class="el-icon-reading preIcon" /> 借阅</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 表格底部分页栏 -->
    <div style="padding: 10px 0;">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                     :current-page="pageNum" :page-sizes="[2, 5, 10, 20]" :page-size="pageSize"
                     layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <book-detail :book-id.sync="infoId" :show.sync="infoDialog" />
  </div>
</template>

<script>
import bookDetail from "@/components/bookDetail";

export default {
  name: "userBookTable",
  components:{
    bookDetail,
  },
  data(){
    return{
      tableData:[],
      username:"",
      userID:"",

      bookID: '',
      bookInfo: '',

      pageNum: 1,
      pageSize: 5,
      total: 0,

      infoDialog: false,//书籍详情：对话触发
      infoId: '',//书籍详情：传入书籍id

      loading: false//目前仅有表格loading
    }
  },
  created() {
    this.username=this.$route.query
    this.load()
  },
  methods: {


    load() {//加载所有图书信息
      this.loading = true
      this.$axios.get('/SearchBook/findAll').then(res => {
        this.tableData = res.data
        this.total = res.data.length

        this.bookInfo = ""
        this.bookID = ""
        this.loading = false
        this.handleCurrentChange(1)
        this.handleSizeChange(5)
        this.$axios.get('/findUserId?username='+this.username['username']).then(data=>data.data).then(data=>{
          this.userID=data.data
        })
      })
    },
    findOne() {//根据条形码索书号完全匹配查询
      this.$axios.get('/SearchBook/findOne?bookId=' + this.bookID).then(res => {
        this.tableData = res.data
        this.total = res.data.length
      })
    },
    fuzzySearch() {//根据图书相关信息模糊查询
      this.$axios.get('/SearchBook/fuzzySearch?pageNum='
          + this.pageNum + '&pageSize=' + this.pageSize + '&searchCond=' + this.bookInfo).then(res => {
        this.tableData = res.data
      })
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.fuzzySearch()
    },
    handleCurrentChange(pageNum) {
      this.pageNum = pageNum
      this.fuzzySearch()
    },
    borrow(row) {//点击借阅时触发
      this.$confirm("确认借阅该书?", "提示", {
        iconClass: "el-icon-question",
        confirmButtonText: "确认",
        cancelButtonText: "取消",
        showClose: true,
        type: "info"
      }).then(() => {
        // 确认操作
        console.log("borrow", this.userID, row.book_id);
        this.$axios.post('/BorrowBook/addBorrow?u_id=' + this.userID + '&b_id=' + row.book_id)
            .then(() => {
              this.load()
            })
            .catch((err) => {
              // 捕获异常，输出错误信息
              console.error("借阅书籍失败：", err);
            });
      }).catch(() => {
        // 取消操作
      });
    },
  }
}
</script>

<style scoped>

</style>