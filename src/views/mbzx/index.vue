<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="700px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="140px">
          <el-form-item label="标题">
            <el-input v-model="form.title" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="描述">
            <el-t v-model="form.describe" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="图片">
            <template>
              <div class="upload">
                <div class="upload_warp">
                  <div class="upload_warp_left" @click="fileClick">
                    <img src="@/assets/images/upload.png">
                  </div>
                </div>
                <div class="upload_warp_text">
                  选中{{ imgList.length }}个文件，共{{ bytesToSize(this.size) }}
                  <span class="ml20 c-red">[单个附件的最大尺寸为10MB]</span>
                </div>
                <input id="upload_file" type="file" multiple style="display: none" @change="fileChange($event)">
                <div v-show="imgList.length!=0" class="upload_warp_img">
                  <div v-for="(item,index) of imgList" class="upload_warp_img_div">
                    <div class="upload_warp_img_div_top">
                      <div class="upload_warp_img_div_text">
                        {{ item.file.name }}
                      </div>
                      <img src="@/assets/images/del.png" class="upload_warp_img_div_del" @click="fileDel(index)">
                    </div>
                    <img :src="item.file.src">
                  </div>
                </div>
              </div>
            </template>
            <!--            <el-input v-model="form.image" style="width: 370px;" />-->
          </el-form-item>
          <el-form-item label="优惠价格">
            <el-input v-model="form.priceNew" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="原价">
            <el-input v-model="form.priceOld" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="活动开始时间">
            <el-input v-model="form.beginTime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="活动结束时间">
            <el-input v-model="form.endTime" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="title" label="活动标题" />
        <el-table-column prop="describe" label="活动描述" />
        <el-table-column prop="image" label="图片" />
        <el-table-column prop="priceNew" label="优惠价格" />
        <el-table-column prop="priceOld" label="原价" />
        <el-table-column prop="beginTime" label="活动开始时间" />
        <el-table-column prop="endTime" label="活动结束时间" />
        <el-table-column prop="createTime" label="创建日期" />
        <el-table-column v-if="checkPer(['admin','activity:edit','activity:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudActivity from '@/api/mbzx/activity'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, createBy: null, updateBy: null, createTime: null, updateTime: null, title: null, describe: null, image: null, priceNew: null, priceOld: null, beginTime: null, endTime: null }
export default {
  name: 'Activity',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: 'api/mbzx/activity', url: 'api/mbzx/activity', idField: 'id', sort: 'id,desc', crudMethod: { ...crudActivity }})
  },
  data() {
    return {
      imgList: [],
      size: 0,
      permission: {
        add: ['admin', 'activity:add'],
        edit: ['admin', 'activity:edit'],
        del: ['admin', 'activity:del']
      },
      rules: {
      }}
  },
  methods: {
    fileClick() {
      document.getElementById('upload_file').click()
    },
    fileChange(el) {
      if (!el.target.files[0].size) return
      this.fileList(el.target.files)
      el.target.value = ''
    },
    fileList(files) {
      for (let i = 0; i < files.length; i++) {
        this.fileAdd(files[i])
      }
    },
    fileAdd(file) {
      this.size = this.size + file.size// 总大小
      const reader = new FileReader()
      reader.vue = this
      reader.readAsDataURL(file)
      reader.onload = function() {
        file.src = this.result
        this.vue.imgList.push({
          file
        })
      }
    },
    fileDel(index) {
      this.size = this.size - this.imgList[index].file.size// 总大小
      this.imgList.splice(index, 1)
    },
    bytesToSize(bytes) {
      if (bytes === 0) return '0 B'
      const k = 1000 // or 1024
      const sizes = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return (bytes / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i]
    },
    dragenter(el) {
      el.stopPropagation()
      el.preventDefault()
    },
    dragover(el) {
      el.stopPropagation()
      el.preventDefault()
    },
    drop(el) {
      el.stopPropagation()
      el.preventDefault()
      this.fileList(el.dataTransfer.files)
    },
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>
  .upload_warp_img_div_del {
    position: absolute;
    top: 6px;
    width: 16px;
    right: 4px;
  }

  .upload_warp_img_div_top {
    position: absolute;
    top: 0;
    width: 100%;
    height: 30px;
    background-color: rgba(0, 0, 0, 0.4);
    line-height: 30px;
    text-align: left;
    color: #fff;
    font-size: 12px;
    text-indent: 4px;
  }

  .upload_warp_img_div_text {
    white-space: nowrap;
    width: 80%;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .upload_warp_img_div img {
    max-width: 100%;
    max-height: 100%;
    vertical-align: middle;
  }

  .upload_warp_img_div {
    position: relative;
    height: 100px;
    width: 120px;
    border: 1px solid #2196F3;
    margin: 0px 30px 10px 0px;
    float: left;
    line-height: 100px;
    display: table-cell;
    text-align: center;
    background-color: #eee;
    cursor: pointer;
  }

  .upload_warp_img {
    border-top: 1px solid #D2D2D2;
    padding: 14px 0 0 14px;
    overflow: hidden
  }

  .upload_warp_text {
    text-align: left;
    margin-bottom: 10px;
    padding-top: 10px;
    text-indent: 14px;
    border-top: 1px solid #2196F3;
    font-size: 14px;
  }

  .upload_warp_right {
    float: left;
    width: 56%;
    margin-left: 2%;
    height: 100%;
    border: 2px dashed #2196F3;
    border-radius: 4px;
    line-height: 130px;
    color: #2196F3;
  }

  .upload_warp_left img {
    margin-top: 32px;
  }

  .upload_warp_left {
    float: left;
    width: 40%;
    height: 100%;
    border: 2px dashed #2196F3;
    border-radius: 4px;
    cursor: pointer;
  }

  .upload_warp {
    margin: 14px;
    height: 130px;
  }

  .upload {
    border: 1px solid #2196F3;
    background-color: #fff;
    width: 430px;
    border-radius: 4px;
  }

  .hello {
    width: 800px;
    margin-left: 27%;
    text-align: center;
  }
  .ml20{
    margin-left: 20px;
  }
  .c-red{
    color: red;
  }
</style>
