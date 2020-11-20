<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">

             <span>青春是一个短暂的美梦, 当你醒来时, 它早已消失无踪</span>
             <el-divider></el-divider>
             <div >
              <el-input placeholder="请输入标题" v-model="title">
                <template slot="prepend">标题</template>
              </el-input>
            </div>
            <div style="margin-top: 20px;">
             <span>文章标签:</span>
              <el-tag
                   :key="tag"
                    v-for="tag in dynamicTags"
                    closable
                    :disable-transitions="false"
                    @close="handleClose(tag)">
                    {{tag}}
              </el-tag>
              <el-input
                 class="input-new-tag"
                 v-if="inputVisible"
                 v-model="inputValue"
                 ref="saveTagInput"
                 size="small"
                 @keyup.enter.native="handleInputConfirm"
                 @blur="handleInputConfirm"
                 >
               </el-input>
              <el-button v-else class="button-new-tag" size="small" @click="showInput">+ 添加文章标签</el-button>
            </div>
            <mavon-editor style="margin-top: 20px;"  ref="md" :style="'height:' + height" @imgAdd="imgAdd" v-model="content" @change="change" />
            <div slot="footer" style="margin-top: 20px; text-align:center" class="dialog-footer">
                            <el-button type="success" round icon="el-icon-edit" @click="submit('draft')">保存草稿</el-button>
                           <el-button type="primary" round  icon="el-icon-success" @click="submit('release')">发布文章</el-button>
            </div>
    </div>
  </div>
</template>

<style>
  .el-tag + .el-tag {
    margin-left: 10px;
  }
  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }
  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }
</style>

<script>
// 全局注册
// import with ES6
import Vue from 'vue'
import {add} from '@/api/article'
import { upload } from '@/utils/upload'
import { mapGetters } from 'vuex'
export default {
  name: 'Markdown',
   data() {
        return {
          title: "",
          content:"",
          html:'',
          id:'',
          height: document.documentElement.clientHeight - 200 + 'px',
          dialogVisible: false,
          dynamicTags: [],
          inputVisible: false,
          inputValue: ''
       }
    },
    computed: {
        ...mapGetters([
          'imagesUploadApi'
        ])
      },
      mounted() {
        const that = this
        window.onresize = function temp() {
          that.height = document.documentElement.clientHeight - 200 + 'px'
        }
      },
      methods: {
        imgAdd(pos, $file) {
          upload(this.imagesUploadApi, $file).then(data => {
            this.$refs.md.$img2Url(pos, data.data.url)
          })
        },
        submit(types){
            let status=''
            if(types=='release'){
              status='2'; //发布待审核
            }else{
              status='0';  //草稿
            }
            let data;
            if(this.id==''){
              data={
                title:this.title,
                content:this.content,
                html:this.html,
                tags:this.dynamicTags.join(','),
                status:status,
              }
            }else{
              data={
                id:this.id,
                title:this.title,
                content:this.content,
                html:this.html,
                tags:this.dynamicTags.join(','),
                status:status,
              }
            }

           if(this.title==''){
             this.$message({
                message: '请输入文章标题',
                type: 'error'
            });
            return;
           }

           if(this.dynamicTags.length<=0){
             this.$message({
                message: '请添加文章标签！',
                type: 'error'
            });
            return;
           }
           if(this.dynamicTags.length>3){
              this.$message({
                 message: '最多只能添加三个标签哦,快去干掉几个吧！',
                 type: 'error'
             });
             return;

           }

           if(this.content==''){
             this.$message({
                message: '请填写文章内容',
                type: 'error'
            });
            return;
           }
             add(data).then(res => {
                  if(types=='release'){
                    this.$message({
                      message: '提交成功！静静等待管理员审核吧！',
                      type: 'success'
                    });
                  }else{
                    this.$message({
                      message: '保存草稿完成！',
                      type: 'success'
                    });
                    this.id=res.id
                  }

                    // console.log(res)
                  }).catch(err => {
                    this.isDisabled=false;
                    this.$message.error('@、@ 程序员哥哥也不知道哪里出了问题，请联系管理员！');
                    console.log(err.response.data.message)
                  })
            },
        change(value, render){

           this.html = render;
        },
        handleClose(tag) {
                this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1);
              },

              showInput() {
                this.inputVisible = true;
                this.$nextTick(_ => {
                  this.$refs.saveTagInput.$refs.input.focus();
                });
              },

              handleInputConfirm() {
                let inputValue = this.inputValue;
                if (inputValue) {
                  this.dynamicTags.push(inputValue);
                }
                this.inputVisible = false;
                this.inputValue = '';
              }
      }
}
</script>
