<template>
  <el-container class="list">
    <el-aside width="120px">
      <el-menu :default-active="activeName"
               background-color="#171b22"
               text-color="#fff"
               active-text-color="#00baff"
               @select="handleSelect">
        <el-menu-item :index="item.categoryValue"
                      :key="item.categoryValue"
                      v-for="item in typelist"
                      @click="getList(item.categoryValue)">
          <i class="el-icon-s-order"></i>
          {{ item.categoryKey }}
        </el-menu-item>
      </el-menu>

    </el-aside>
    <el-container>
      <el-header class="page">

        <el-pagination layout="total, sizes, prev, pager, next, jumper"
                       background
                       size="mini"
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :page-size="page.size"
                       :current-page.sync="page.page"
                       :total="page.total">
        </el-pagination>
        <el-input style="width: 150px" size="mini" v-model="nm" placeholder="请输入名称搜索"></el-input>
        <el-button @click="getList(activeName)" size="mini" style="height: 28px">搜索</el-button>
		<el-select size="mini" style="margin-left: 10px;" v-model="sort" placeholder="请选择">
		    <el-option
		      v-for="item in dicSort"
		      :key="item.value"
		      :label="item.label"
		      :value="item.value">
		    </el-option>
		  </el-select>
      </el-header>
      <el-main class="content">
        <div class="content__item content__item--add"
             @click="handleAdd">
          <div>
            <i class="el-icon-plus"></i>
            <p>新建大屏</p>
          </div>
        </div>
        <div class="content__item"
             v-for="(item,index) in list"
             :key="index"
             @mouseover="item._menu=true"
             @mouseout="item._menu=false">
          <div class="content__info">
            <img v-if="item.backgroundUrl"
                 :src="item.backgroundUrl"
                 alt=""/>
            <div class="content__menu"
                 v-show="item._menu">
              <div class="content__btn"
                   @click="handleEdit(item)">
                编辑
              </div>
              <div class="content__btn"
                   @click="handleExport(item)">
                导出
              </div>
            </div>
          </div>
          <div class="content__main">
            <span class="content__name">{{ item.title }}</span>
            <div class="content__menulist">
              <div class="content__view">
                <el-tooltip content="删除">
                  <i class="el-icon-delete"
                     @click="handleDel(item,index)"></i>
                </el-tooltip>
                <el-tooltip content="修改">
                  <i class="el-icon-edit"
                     @click="handleUpdate(item,index)"></i>
                </el-tooltip>
                <el-tooltip content="预览">
                  <i class="el-icon-view"
                     @click="handleViews(item,index)"></i>
                </el-tooltip>
                <el-tooltip content="复制">
                  <i class="el-icon-copy-document"
                     @click="handleCopy(item,index)"></i>
                </el-tooltip>
              </div>
<!--              <span class="content__status"-->
<!--                    :class="{'content__status&#45;&#45;active':item.status}">-->
<!--                {{ item.status ? '已发布' : '未发布' }}-->
<!--              </span>-->
            </div>

          </div>
        </div>
      </el-main>
    </el-container>
    <el-dialog :title="type==='add'?'新建大屏':'编辑大屏'"
               width="35%"
               :visible.sync="box">
      <avue-form :option="option"
                 v-model="form"
                 v-if="box"
                 @submit="handleSave"></avue-form>
    </el-dialog>
  </el-container>
</template>
<script>
import {getList, addObj, updateObj, delObj, getCategory, copyObj} from '@/api/visual';
import {getObj} from '@/api/visual'

export default {
  name: "list",
  data() {
    return {
      typelist: [],
      index: 0,
      type: '',
      nm:'',
      option: {
        column: [{
          label: '分组',
          prop: 'category',
          span: 24,
          labelWidth: 100,
          type: 'select',
          dicUrl: '/api/blade-visual/category/list',
          props: {
            label: 'categoryKey',
            value: 'categoryValue',
          },
          rules: [{
            required: true,
            message: "请选择分组",
            trigger: "blur"
          }]
        }, {
          label: '大屏名称',
          span: 24,
          labelWidth: 100,
          prop: 'title',
          rules: [{
            required: true,
            message: "请输入大屏名称",
            trigger: "blur"
          }]
        }, {
          label: '大屏尺寸',
          span: 14,
          labelWidth: 100,
          prop: 'width',
          placeholder: '请输入宽度',
          rules: [{
            required: true,
            message: "请输入大屏尺寸",
            trigger: "blur"
          }]
        }, {
          label: '',
          span: 10,
          labelWidth: 1,
          prop: 'height',
          placeholder: '请输入高度',
          rules: [{
            required: true,
            message: "请输入大屏尺寸",
            trigger: "blur"
          }]
        }, 
		// {
  //         label: '导航高亮',
  //         span: 24,
  //         labelWidth: 100,
  //         prop: 'navSelect',
  //       }, 
		{
          label: '密码',
          span: 24,
          type: 'password',
          labelWidth: 100,
          prop: 'password',
        }, {
          label: '发布状态',
          prop: 'status',
          span: 24,
          labelWidth: 100,
          type: 'select',
          dicData: [{
            label: '未发布',
            value: 0
          }, {
            label: '已发布',
            value: 1
          }]
        }]
      },
      page: {
        page: 1,
        size: 40,
        total: 0,
      },
      form: {},
      box: false,
      activeName: '',
      list: [],
	  sort: '',
	  dicSort: [{
		label: '大屏名称升序',
		value: 'title-ascs'
	  }, {
		label: '大屏名称降序',
		value: 'title-descs'
	  }, {
		label: '创建时间升序',
		value: 'create_time-ascs'
	  }, {
		label: '创建时间降序',
		value: 'create_time-descs'
	  }, {
		label: '更新时间升序',
		value: 'update_time-ascs'
	  }, {
		label: '更新时间降序',
		value: 'update_time-descs'
	  }]
    }
  },
  created() {
    this.getCategory()
  },
  methods: {
    vaildData(id) {
      const list = [];
      for (var i = 0; i < 20; i++) {
        list.push(i)
      }
      return list.includes(id)
    },
    getCategory() {
      getCategory().then(res => {
        const data = res.data.data;
        this.typelist = data;
        this.activeName = (data[0] || {}).categoryValue;
        this.getList();
      })
    },
    handleExport(item) {
      getObj(item.id).then(res => {
        const data = res.data.data;
        let mode = {
          detail: JSON.parse(data.config.detail),
          component: JSON.parse(data.config.component)
        }
        var zip = new window.JSZip();
        zip.file("view.js", `const option =${JSON.stringify(mode, null, 4)}`);
        zip.file('index.html', window.html);
        zip.generateAsync({type: "base64"})
            .then(function (content) {
              location.href = "data:application/zip;base64," + content;
            });
      })
    },
    handleCopy(item) {
      this.$confirm('确认复制当前大屏', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        copyObj(item.id).then(() => {
          this.$message.success('复制成功');
          this.getList();
        })
      }).catch(() => {

      });
    },
    handleDel(item, index) {
      this.$confirm('是否确认永久删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // if (this.vaildData(index)) {
        //   this.$message.error('例子模板不允许修改')
        //   return false;
        // }
        delObj(item.id).then(() => {
          this.list.splice(index, 1)
          this.$message.success('删除成功')
        })
      }).catch(() => {

      });
    },
    handleAdd() {
      this.type = 'add';
      this.findObject(this.option.column, 'status').display = false
      this.findObject(this.option.column, 'width').display = true
      this.findObject(this.option.column, 'height').display = true
      this.form.title = '';
      this.form.status = '';
      this.form.password = '';
	  this.form.navSelect = '';
      this.form.category = this.activeName;
      this.form.width = 1920;
      this.form.height = 1080;
      this.box = true;
    },
    handleUpdate(item, index) {
      this.type = 'edit';
      this.findObject(this.option.column, 'status').display = true
      this.findObject(this.option.column, 'width').display = false
      this.findObject(this.option.column, 'height').display = false
      this.form = item
      this.form.category = this.form.category + '';
      this.box = true;
      this.index = index;
    },
    handleEdit(item) {
      let routeUrl = this.$router.resolve({
        path: '/build/' + item.id
      })
      window.open(routeUrl.href, '_blank');
    },
    handleViews(item) {
      let routeUrl = this.$router.resolve({
        path: '/view/' + item.id
      })
      window.open(routeUrl.href, '_blank');
    },
    handleSave(form, done) {
      done();
      if (this.type == 'add') {
        addObj(Object.assign({
          category: this.activeName,
        }, this.form)).then(res => {
          this.box = false;
          this.$message.success('新增成功');
          this.getList();
          const id = res.data.data.id;
          this.handleEdit({id})
        })
      } else {
        // if (this.vaildData(Number(this.index))) {
        //   this.$message.error('例子模板不允许修改')
        //   return false;
        // }
        updateObj({
          id: this.form.id,
          category: this.form.category,
          password: this.form.password,
          status: this.form.status,
          title: this.form.title,
		  navSelect: this.form.navSelect
        }).then(() => {
          this.box = false;
          this.$message.success('修改成功');
          this.getList();
        })
      }
    },
    handleSelect(key) {
      this.activeName = key;
      this.page.page = 1;
      this.getList();
    },
    handleCurrentChange(val) {
      this.page.page = val;
      this.getList();
    },
    handleSizeChange(val) {
      this.page.size = val;
      this.getList();
    },


    getList(category) {
      this.list = []
	  let query = {}
	  if(this.sort) {
		query[this.sort.split('-')[1]] = this.sort.split('-')[0]
	  }
      getList({
		title:this.nm,
        category_equal: category || this.activeName,
        current: this.page.page,
        size: this.page.size,
		...query
      }).then(res => {
        const data = res.data.data;
        this.page.total = data.total;
        this.list = data.records
        this.initData();
      })
    },
    initData() {
      this.list.forEach((ele, index) => {
        this.$set(this.list[index], '_menu', false)
      })
    }
  },
  watch: {
	  sort() {
		this.getList()
	  }
  }
}
</script>
<style lang="scss">
@import "@/styles/list.scss";
</style>
