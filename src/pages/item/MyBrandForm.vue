<template>
  <v-form v-model="valid" ref="myBrandForm">
    <v-text-field v-model="brand.name" label="请输入品牌名称" required :rules="nameRules"/>
    <v-text-field v-model="brand.letter" label="请输入品牌首字母" required :rules="letterRules"/>
    <!--下拉列表框可以通过url对后台进行实时查询-->
    <v-cascader url="/item/category/list" multiple required v-model="brand.categories" label="请选择商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image" url="/upload/image" :multiple="false" :pic-width="250" :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-spacer/>
      <v-btn @click="clear" color="grey darken-1" flat>重置</v-btn>
      <v-btn @click="submit" color="blue darken-1" flat>提交</v-btn>
    </v-layout>
  </v-form>
</template>
<script>
    export default {
      name: "my-brand-form",
      data(){
        return{
          valid:false,//表单校验结束
          brand:{
            name:"",
            letter:"",
            categories:[],
            image:""//品牌logo
          },
          nameRules: [
            v => !!v || "品牌名称不能为空",
            v => v.length > 1 || "品牌名称至少2位"
          ],
          letterRules: [
            v => !!v || "首字母不能为空",
            v => /^[a-zA-Z]{1}$/.test(v) || "品牌字母只能是1个字母"
          ]
        }
      },
      methods:{
        submit(){
          //提交时候一定要先看验证是否通过
          if(this.$refs.myBrandForm.validate()){
            //验证通过则把数据发送到后台
            // 定义一个请求参数对象，通过解构表达式来获取brand中的属性
            const {categories, letter, ...params} = this.brand;
            // 数据库中只要保存分类的id即可，因此我们对categories的值进行处理,只保留id，并转为字符串
            params.cids = categories.map(c => c.id).join(",");
            // 将字母都处理为大写
            params.letter = letter.toUpperCase();
            // 将数据提交到后台
            // this.$http.post('/item/brand', this.$qs.stringify(params))
            this.$http({
              method: this.isEdit ? 'put' : 'post',
              url: '/item/brand',
              //因为axios将默认参数会转化成json发送，这样导致后台无法接收，因此再这边将json转换成string格式参数发送
              data: this.$qs.stringify(params)
            }).then(() => {
              // 关闭窗口
              this.$emit("close");
              this.$message.success("保存成功！");
            })
              .catch(() => {
                this.$message.error("保存失败！");
              });
          }
        },
        clear(){
          this.$refs.myBrandForm.reset();
          // 需要手动清空商品分类
          this.categories = [];
        }
      },
      watch:{

      }
    }
</script>

<style scoped>

</style>
