<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">主键</label>
        <el-input v-model="query.id" clearable placeholder="主键" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">内容</label>
        <el-input v-model="query.content" clearable placeholder="内容" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">类型</label>
        <el-input v-model="query.type" clearable placeholder="类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">状态:0初始化待审核,1审核中,2审核失败,3审核成功</label>
        <el-input v-model="query.state" clearable placeholder="状态:0初始化待审核,1审核中,2审核失败,3审核成功" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">作者</label>
        <el-input v-model="query.author" clearable placeholder="作者" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">出自</label>
        <el-input v-model="query.contentFrom" clearable placeholder="出自" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">创建人</label>
        <el-input v-model="query.openId" clearable placeholder="创建人" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">扩展字段</label>
        <el-input v-model="query.ext" clearable placeholder="扩展字段" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">删除标记，0代表有效，1代表无效</label>
        <el-input v-model="query.delFlag" clearable placeholder="删除标记，0代表有效，1代表无效" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="主键">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="内容" prop="content">
            <el-input v-model="form.content" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="摘要">
            <el-input v-model="form.contentAbstract" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="类型" prop="type">
            <el-radio v-model="form.type" v-for="item in dict.poetry_type" :key="item.id" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="状态:0初始化待审核,1审核中,2审核失败,3审核成功">
            <el-radio v-model="form.state" v-for="item in dict.poetry_status" :key="item.id" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="作者" prop="author">
            <el-input v-model="form.author" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="出自" prop="contentFrom">
            <el-input v-model="form.contentFrom" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建人">
            <el-input v-model="form.openId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="长图">
            <el-input v-model="form.longPhoto" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="扩展字段">
            <el-input v-model="form.ext" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="删除标记，0代表有效，1代表无效">
            <el-radio v-model="form.delFlag" v-for="item in dict.del_flag" :key="item.id" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="主键" />
        <el-table-column ref="table" :show-overflow-tooltip="true" prop="content" label="内容">
          <template slot-scope="{row}">
            {{ row.content }}
          </template>
        </el-table-column>
        <el-table-column ref="table" :show-overflow-tooltip="true" prop="longPhoto" label="长图" style="width: 60px">
          <template slot-scope="{row}">
            <el-image
              :src="row.longPhoto"
              :preview-src-list="[row.longPhoto]"
              fit="contain"
              lazy
            />
          </template>
        </el-table-column>
        <el-table-column prop="contentAbstract" label="摘要" />
        <el-table-column prop="type" label="类型">
          <template slot-scope="scope">
            {{ dict.label.poetry_type[scope.row.type] }}
          </template>
        </el-table-column>
        <el-table-column prop="state" label="状态:0初始化待审核,1审核中,2审核失败,3审核成功">
          <template slot-scope="scope">
            {{ dict.label.poetry_status[scope.row.state] }}
          </template>
        </el-table-column>
        <el-table-column prop="author" label="作者" />
        <el-table-column prop="contentFrom" label="出自" />
        <el-table-column prop="openId" label="创建人" />
        <el-table-column ref="table" :show-overflow-tooltip="true" prop="ext" label="扩展字段">
          <template slot-scope="{row}">
            {{ row.ext }}
          </template>
        </el-table-column>
        <el-table-column prop="delFlag" label="删除标记，0代表有效，1代表无效">
          <template slot-scope="scope">
            {{ dict.label.del_flag[scope.row.delFlag] }}
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','poetry:edit','poetry:del']" label="操作" width="150px" align="center">
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
import crudPoetry from '@/api/poetry'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, content: null, contentAbstract: null, type: null, state: null, author: null, contentFrom: null, openId: null, originalPhoto: null, standardPhoto: null, longPhoto: null, widePhoto: null, ellipsisPhoto: null, ext: null, delFlag: null, createTime: null, updateTime: null, saveTime: null }
export default {
  name: 'Poetry',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['poetry_type', 'poetry_status', 'del_flag'],
  cruds() {
    return CRUD({ title: '诗歌后台', url: 'api/poetry', sort: 'id,desc', crudMethod: { ...crudPoetry }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'poetry:add'],
        edit: ['admin', 'poetry:edit'],
        del: ['admin', 'poetry:del']
      },
      rules: {
        content: [
          { required: true, message: '内容不能为空', trigger: 'blur' }
        ],
        type: [
          { required: true, message: '类型不能为空', trigger: 'blur' }
        ],
        author: [
          { required: true, message: '作者不能为空', trigger: 'blur' }
        ],
        contentFrom: [
          { required: true, message: '出自不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'id', display_name: '主键' },
        { key: 'content', display_name: '内容' },
        { key: 'type', display_name: '类型' },
        { key: 'state', display_name: '状态:0初始化待审核,1审核中,2审核失败,3审核成功' },
        { key: 'author', display_name: '作者' },
        { key: 'contentFrom', display_name: '出自' },
        { key: 'openId', display_name: '创建人' },
        { key: 'ext', display_name: '扩展字段' },
        { key: 'delFlag', display_name: '删除标记，0代表有效，1代表无效' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
