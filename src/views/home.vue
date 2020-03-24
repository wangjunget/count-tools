<template>
  <main>
    <div class="input-area">
      <el-input
        v-model="input"
        type="textarea"
        :rows="10"
        placeholder="请输入内容"
      >
      </el-input>
    </div>
    <div class="button-tools">
      <el-button type="primary" @click="parseData">解析</el-button>
      <el-button type="warning" @click="resetInput">清空</el-button>
    </div>
    <div class="table-list">
      <el-table :data="tableData" border style="width: 100%">
        <el-table-column prop="name" label="姓名" width="100">
        </el-table-column>
        <el-table-column prop="phone" label="电话" width="180">
        </el-table-column>
        <el-table-column prop="address" label="地址"></el-table-column>
        <el-table-column prop="comment" label="备注"></el-table-column>
        <el-table-column label="操作" width="100">
          <template slot-scope="scope">
            <el-button
              size="small"
              type="text"
              @click="openEditItem(scope.$index, scope.row)"
            >编辑</el-button>
            <el-button
              size="mini"
              type="text"
              @click="deleteItem(scope.$index, scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="export">
      <el-button type="success" @click="exportExcel">导出Excel</el-button>
      <el-button type="danger" @click="resetResult">清空</el-button>
    </div>

    <el-dialog title="解析结果" :visible.sync="dialogTableVisible" width="70%">
      <el-table :data="dialogData" height="400">
        <el-table-column
          property="name"
          label="姓名"
          width="80"
        ></el-table-column>
        <el-table-column
          property="phone"
          label="电话"
          width="150"
        ></el-table-column>
        <el-table-column property="address" label="地址"></el-table-column>
        <el-table-column property="comment" label="备注"></el-table-column>
      </el-table>
      <div style="margin-top: 30px;">
        <el-button type="primary" @click="addData">添加</el-button>
        <el-button type="danger" @click="cancelAddData">取消</el-button>
      </div>
    </el-dialog>

    <el-dialog title="编辑信息" :visible.sync="dialogEditVisiable">
      <el-form :model="editForm">
        <el-form-item label="姓名" label-width="80px">
          <el-input v-model="editForm.name" autocomplete="off" width="300"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-width="80px">
          <el-input v-model="editForm.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="地址" label-width="80px">
          <el-input v-model="editForm.address" autocomplete="off" type="textarea"></el-input>
        </el-form-item>
        <el-form-item label="备注" label-width="80px">
          <el-input v-model="editForm.comment" autocomplete="off" type="textarea"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogEditVisiable = false">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </div>
    </el-dialog>
  </main>
</template>

<script>
// import XLSX from 'xlsx'

export default {
  name: 'Home',
  data() {
    return {
      input: '',
      dialogTableVisible: false, // 是否显示添加确认对话框
      dialogEditVisiable: false, // 是否显示编辑表单
      editForm: {}, // 正在编辑的表单
      openedItemIndex: '', // 正在编辑的条目索引
      parseButtonClickable: true,
      tableData: [],
      dialogData: []
    }
  },
  watch: {
    tableData: {
      handler(value) {
        window.sessionStorage.setItem('table-data', JSON.stringify(value))
      }
    }
  },
  created() {
    const oldData = window.sessionStorage.getItem('table-data')
    if (oldData) {
      this.tableData = JSON.parse(oldData)
    }
  },
  methods: {
    // 清空input
    resetInput() {
      this.input = ''
    },
    // 清空结果数据
    resetResult() {
      this.tableData = []
    },
    // 解析数据
    parseData() {
      if (!this.input) return this.$message.error('内容不能为空')
      this._dataParse()
        .then(result => {
          this.dialogData = result
          this.dialogTableVisible = true
        })
        .catch((err) => {
          this.$message.error(err)
        })
    },
    // 添加数据到数据列表中
    addData() {
      if (this.dialogData.length === 0) {
        this.dialogTableVisible = false
      } else {
        this.tableData = this.tableData.concat(this.dialogData)
        this.dialogTableVisible = false
        this.$message.success('添加成功')
      }
    },
    // 取消添加
    cancelAddData() {
      this.dialogTableVisible = false
      this.dialogData = []
    },
    // 删除tableData条目
    deleteItem(index) {
      this.$confirm('确认删除？')
        .then(_ => {
          this.tableData.splice(index, 1)
        })
        .catch(_ => {})
    },
    // 编辑tableData条目
    openEditItem(index, item) {
      this.editForm = Object.assign({}, item)
      this.openedItemIndex = index
      this.dialogEditVisiable = true
    },
    // 保存修改
    saveEdit() {
      this.tableData.splice(this.openedItemIndex, 1, this.editForm)
      this.dialogEditVisiable = false
    },
    // 数据解析函数
    _dataParse() {
      const mark = [',', '，', '。']
      const content = [] // 存放处理后数据

      return new Promise((resolve, reject) => {
        let rawData = this.input.trim() // 去除收尾空格
        rawData = rawData.replace(/(^\n*)|(\n*$)/g, '') // 去除首位换行符
        const rowList = rawData.split(/\n{2,}/) // 解析每一条条内容

        rowList.forEach(col => {
          const result = {
            name: '',
            phone: '',
            address: '',
            comment: ''
          } // 存放单条数据内容
          let newCol = col.trim() // 去除单条首位空格
          mark.forEach(item => {
            newCol = newCol.replace(new RegExp(item, 'gm'), ' ') // 替换指定字符为空格
          })

          const array = newCol.split(/\s+/) // 以空格分隔内容

          // if (array.length < 3) reject('解析错误，请检查内容格式')

          // 解析电话号码
          for (let i = 0; i < array.length; i++) {
            if (/^1[3456789]\d{9}$/.test(array[i])) {
              result.phone = array[i]
              array.splice(i, 1)
              break
            }
          }
          // console.log(result)
          // console.log(array)

          // 解析地址
          for (let i = 0; i < array.length; i++) {
            if (/省|市|自治区|自治州|县|区/g.test(array[i]) && array[i].length >= 10) {
              result.address = array[i]
              array.splice(i, 1)
              break
            }
          }

          // console.log(result)
          // console.log(array)

          // 解析姓名
          for (let i = 0; i < array.length; i++) {
          // if (/赵|钱|孙|李|周|吴|郑|王/g.test(array[i])) {
          //   result.name = array[i]
          //   array.splice(i, 1)
          //   break
          // }
            if (array[i].length <= 4 && array[i].length >= 2) {
              result.name = array[i]
              array.splice(i, 1)
              break
            }
          }
          // console.log(result)
          // console.log(array)

          // 解析备注
          for (let i = 0; i < array.length; i++) {
            if (!result.comment) {
              result.comment = array[i]
            } else {
              result.comment = result.comment + ', ' + array[i]
            }
          }
          // console.log(result)
          // console.log(array)

          content.push(result)
        })

        return resolve(content)
      })
    },
    // 导出exce表格
    exportExcel() {
      if (!this.tableData.length) return this.$message.warning('暂无数据')
      const data = [
        ['姓名', '手机号', '地址', '备注']
      ]
      this.tableData.forEach(item => {
        data.push([item.name, item.phone, item.address, item.comment])
      })
      var worksheet = XLSX.utils.aoa_to_sheet(data)
      worksheet['!cols'] = [ // 设置Excel列宽
        { wch: 10 },
        { wch: 15 },
        { wch: 70 },
        { wch: 70 }
      ]
      var new_workbook = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(new_workbook, worksheet, 'SheetJS')
      XLSX.writeFile(new_workbook, '统计结果.xlsx')
    }
  }
}
</script>

<style scoped lang="less">
main {
  margin-top: 60px;
}
.input-area {
  width: 100%;
}
.button-tools {
  text-align: left;
  margin-top: 20px;
}
.table-list {
  margin-top: 100px;
}
.export {
  margin-top: 20px;
  text-align: left;
}
</style>
