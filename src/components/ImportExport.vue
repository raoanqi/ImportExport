<template>
  <div>
    <el-card>
      <el-table
          :data="listData"
          border>
        <el-table-column
            label="通道号"
            prop="channelNumber">
        </el-table-column>
        <el-table-column
            label="编码ID"
            prop="encodeID">
        </el-table-column>
        <el-table-column label="人脸">
          <template v-slot="{row}">
            <el-checkbox v-model="row.faceCheck">
            </el-checkbox>
          </template>
        </el-table-column>
        <el-table-column label="图片">
          <template v-slot="{row}">
            <el-checkbox v-model="row.picCheck">
            </el-checkbox>
          </template>
        </el-table-column>
        <el-table-column label="机动车">
          <template v-slot="{row}">
            <el-checkbox v-model="row.carCheck">
            </el-checkbox>
          </template>
        </el-table-column>
      </el-table>
      <el-row>
        <el-button
            type="primary"
            @click="handleDisplayImportDialog">
          Import
        </el-button>
        <el-button
            type="primary"
            @click="handleExportTableData">
          Export
        </el-button>
      </el-row>
    </el-card>
    <el-dialog
        title="批量导入数据"
        :visible.sync="displayImportDialog">
      <el-upload
          drag
          action=""
          :on-change="handleSelectFileChange"
          :auto-upload="false"
          :show-file-list="false">
        <template v-if="!importFile.records.length">
          <div style="height: 100%">
            <p>
              请将文件拖拽至虚线区域中或者直接点击虚线区域
            </p>
          </div>
        </template>
        <template v-else>
          <div>
            <i class="el-icon-document"></i>
            <p>{{ importFile.file.name }}</p>
            <p>发现{{ importFile.records.length }}条数据</p>
          </div>
        </template>
      </el-upload>
      <span slot="footer" class="dialog-footer">
        <el-button @click="displayImportDialog=false">取消</el-button>
        <el-button
            type="primary"
            @click="handleSyncDataToTable">
          同步至表格
        </el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import XLSX from 'xlsx'

export default {
  name: "ImportExport",
  data() {
    return {
      // 控制导入对话框的显示隐藏
      displayImportDialog: false,
      // 导入后绑定的数据
      importFile: {
        file: null,
        records: []
      },
      // 项目启动时，表格中的初始化数据
      listData: [
        {
          channelNumber: '1',
          encodeID: '1111',
          faceCheck: true,
          picCheck: false,
          carCheck: false
        },
        {
          channelNumber: '2',
          encodeID: '2222',
          faceCheck: false,
          picCheck: true,
          carCheck: false
        },
        {
          channelNumber: '3',
          encodeID: '3333',
          faceCheck: false,
          picCheck: false,
          carCheck: true
        }
      ]
    }
  },
  methods: {
    // 显示导入数据对话框
    handleDisplayImportDialog() {
      this.displayImportDialog = true
    },
    // 选择的文件发生变化
    handleSelectFileChange(file) {
      console.log(file)
      this.importFile.file = file.raw
      let rABS = true
      let f = file.raw
      let reader = new FileReader()
      reader.onload = (e) => {
        let data = e.target.result
        if (!rABS) data = new Uint8Array(data)
        let workbook = XLSX.read(data, {type: rABS ? 'binary' : 'array'})
        let ws = workbook.Sheets[workbook.SheetNames[0]]
        let sheetArr = XLSX.utils.sheet_to_json(ws)
        if (!sheetArr.length) {
          this.$message({
            type: 'warning',
            message: '导入的文件中没有数据'
          })
          return
        }
        let checkFlag = true;
        ['通道号', '编码ID', '人脸', '图片', '机动车'].forEach(item => {
          if (!Object.prototype.hasOwnProperty.call(sheetArr[0], item)) {
            checkFlag = false
          }
        })
        if (!checkFlag) {
          this.$message({
            type: 'warning',
            message: '数据格式不正确'
          })
          return
        }
        this.importFile.records = [...sheetArr].map(item => {
          return {
            channelNumber: item['通道号'],
            encodeID: item['编码ID'],
            faceCheck: item['人脸'],
            picCheck: item['图片'],
            carCheck: item['机动车']
          }
        })
      }
      if (rABS) {
        reader.readAsBinaryString(f)
      } else {
        reader.readAsArrayBuffer(f)
      }
    },
    // 将选中文件同步到表格中
    handleSyncDataToTable() {
      this.listData = [...this.importFile.records]
      this.displayImportDialog = false
    },
    // 导出数据的辅助方法1
    s2ab(s) {
      let buf = new ArrayBuffer(s.length)
      let view = new Uint8Array(buf)
      for (let i = 0; i !== s.length; ++i) {
        view[i] = s.charCodeAt(i) & 0xFF
      }
      return buf
    },
    // 导出数据的辅助方法2
    sheet2blob(sheet, sheetName) {
      sheetName = sheetName || 'sheet1'
      let workbook = {
        SheetNames: [sheetName],
        Sheets: {}
      }
      workbook.Sheets[sheetName] = sheet
      let wopts = {
        bookType: 'xlsx',
        bookSST: false,
        type: 'binary'
      }
      let wbout = XLSX.write(workbook, wopts)
      let blob = new Blob([this.s2ab(wbout), {type: 'application/octet-stream'}])
      return blob
    },
    // 导出数据的辅助方法3
    openDownloadDialog(url, saveName) {
      if (typeof url === 'object' && url instanceof Blob) {
        url = URL.createObjectURL(url)
      }
      let aLink = document.createElement('a')
      aLink.href = url
      aLink.download = saveName || ''
      let event
      if (window.MouseEvent) {
        event = new MouseEvent('click')
      } else {
        event = document.createEvent('MouseEvents')
        event.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null)
      }
      aLink.dispatchEvent(event)
    },
    // 导出数据
    handleExportTableData() {
      let mapData = this.listData.map(item => {
        return [
          item.channelNumber,
          item.encodeID,
          item.faceCheck,
          item.picCheck,
          item.carCheck
        ]
      })
      const aoa = [
        ['通道号', '编码ID', '人脸', '图片', '机动车'],
        ...mapData
      ]
      console.log(aoa)
      const sheet = XLSX.utils.aoa_to_sheet(aoa)
      this.openDownloadDialog(this.sheet2blob(sheet), '导出数据文件.xlsx')
    }
  }
}
</script>

<style scoped>

</style>