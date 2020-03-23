<template>
  <main>
    <div class="input-area">
      <el-input
        v-model="input"
        type="textarea"
        :rows="5"
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
      </el-table>
    </div>
    <div class="export">
      <el-button type="success" @click="exportExcel">导出Excel</el-button>
    </div>

    <el-dialog title="解析结果" :visible.sync="dialogTableVisible">
      <el-table :data="dialogData">
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
  </main>
</template>

<script>
// import XLSX from 'xlsx'

export default {
  name: 'Home',
  data() {
    return {
      input: '',
      dialogTableVisible: false,
      parseButtonClickable: true,
      tableData: [],
      dialogData: []
    }
  },
  methods: {
    // 清空input
    resetInput() {
      this.input = ''
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
    cancelAddData() {
      this.dialogTableVisible = false
      this.dialogData = []
    },
    // 数据解析函数
    _dataParse() {
      const mark = [',', '，']

      return new Promise((resolve, reject) => {
        let rawData = this.input.trim() // 去除收尾空格
        mark.forEach(item => {
          rawData = rawData.replace(new RegExp(item, 'gm'), ' ')
        })

        const array = rawData.split(/\s+/)
        const result = {}

        // console.log(array)
        if (array.length < 3) reject('解析错误，请检查内容格式')

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
          if (/省|市|自治区|自治州|县|区/g.test(array[i]) && array[i].length > 4) {
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
          if (array[i].length <= 3) {
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
            result.comment = result.comment + array[i]
          }
        }
        // console.log(result)
        // console.log(array)

        return resolve([result])
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
  width: 60%;
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
