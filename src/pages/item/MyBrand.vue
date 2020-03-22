<template>
  <div>
    <v-card>
      <!-- 卡片的头部 -->
      <v-card-title>
        <v-btn color="primary" dark @click="create">新增品牌</v-btn>
        <!--空间隔离工具-->
        <v-spacer />
        <v-spacer />
        <!--搜索框，与search属性关联  hide-details="false"是关闭错误提示空间-->
        <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
      </v-card-title>
      <!-- 分割线 -->
      <v-divider/>
      <!--卡片的中部-->
      <v-data-table
        :headers="headers"
        :items="brands"
        :search="search"
        :pagination.sync="pagination"
        :total-items="totalBrands"
        :loading="loading"
        class="elevation-1"
      >
        <template slot="items" slot-scope="props">
          <td>{{ props.item.id }}</td>
          <td class="text-xs-center">{{ props.item.name }}</td>
          <td class="text-xs-center"><img :src="props.item.image"></td>
          <td class="text-xs-center">{{ props.item.letter }}</td>
          <td class="justify-center layout">
            <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
            <v-btn color="warning">删除</v-btn>
          </td>
        </template>
      </v-data-table>

      <!--点击新增弹出模态框-->
      <v-layout row justify-center>
        <v-dialog v-model="dialog" max-width="500px">
          <v-card>
            <v-card-title class="primary">
              <span class="headline">{{isEdit ? '修改' : '新增'}}品牌</span>
              <v-spacer/>
              <!--关闭窗口的按钮-->
              <v-btn icon @click="closeCreate"><v-icon>close</v-icon></v-btn>
            </v-card-title>
            <v-card-text>
              <brand-form ref="myBrand" :isEdit="isEdit" :oldBrand="oldBrand" @close="closeCreate"/>
            </v-card-text>
          </v-card>
        </v-dialog>
      </v-layout>
    </v-card>
  </div>
</template>

<script>

  import BrandForm from './BrandForm'

    export default {
      name: "my-brand",
      data () {
        return {
            search: '', // 搜索过滤字段
            totalBrands: 0, // 总条数
            brands: [], // 当前页品牌数据
            loading: true, // 是否在加载中
            pagination: {}, // 分页信息
            dialog:false,
            isEdit:false,
            oldBrand: {}, // 即将被编辑的品牌数据
            headers: [ // 头信息
              {text: 'id', align: 'center', value: 'id'},
              {text: '名称', align: 'center', sortable: false, value: 'name'},
              {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
              {text: '首字母', align: 'center', value: 'letter', sortable: true,},
              {text: '操作', align: 'center', value: 'id', sortable: false}
            ]
        }
      },
      mounted(){ // 渲染后执行
        // 查询数据
        this.getDataFromServer();
      },
      methods:{
        getDataFromServer(){ // 从服务的加载数的方法。
          this.$http.get("/item/brand/page",{
            params:{
              pageSize:this.pagination.rowsPerPage,//当前页大小
              currentPage:this.pagination.page,//当前页码数
              search:this.search,//搜索条件
              sortBy:this.pagination.sortBy,//排序字段
              desc:this.pagination.descending//排序方式
            }
          }).then(resp=>{
            console.log(resp);
            this.brands=resp.data.items;
            this.totalBrands=resp.data.total;
            //完成赋值后，loading结束
            this.loading=false
          })
        },
        create(){
          // this.$refs.myBrand.clear();
          this.oldBrand=null;
          this.isEdit=false;
          this.dialog=true;
        },
        closeCreate(){
          //关闭窗口前需要重新加载数据
          this.getDataFromServer();

          this.dialog=false;
        },
        editBrand(oldBrand){
          this.isEdit=true;
          //因为没有种类，因此还得发起请求查询种类
          this.$http.get("/item/category/bid/"+oldBrand.id).then(
            resp=>{
              console.log(resp);
              this.oldBrand=oldBrand;
              this.oldBrand.categories=resp.data;
              this.dialog=true;
            }
          ).catch(resp=>{
            this.$message.error("分类有误")
          });
        }
      },
      //点击页面分页会改变Vue属性值，因此用watch监控来发送请求，注意，监控是属性中的属性 是深度监控
      //准确说，watch是实现动态操作页面的来源
      watch:{
        pagination:{
          deep:true,
          //变化后的回调函数，再次发送请求
          handler(){
            this.getDataFromServer();
          }
        },

        //监视搜索字段
        search:{
          handler(){
            this.getDataFromServer();
          }
        }
      },
      components:{
        BrandForm
      }
    }
</script>

<style scoped>

</style>
