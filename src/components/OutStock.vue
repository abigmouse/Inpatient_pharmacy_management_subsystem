<template>
  <el-form ref="form" :model="form" label-width="80px" :rules="rules">
    <el-col :span="12">
      <el-form-item label="出库单位" prop="out">
        <el-input v-model="form.out"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="12">
      <el-form-item label="入库单位" prop="in">
        <el-input v-model="form.in"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="10">
      <el-form-item label="药品名称">
        <el-input v-model="item.name"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="10">
      <el-form-item label="ID">
        <el-input v-model="item.id" :disabled="found"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="4">
      <el-form-item>
        <el-button @click="searchItem" :loading="isSearching">查询</el-button>
      </el-form-item>
    </el-col>
    <el-col :span="6">
      <el-form-item label='数量'>
        <el-input v-model.number="item.num"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="7">
      <el-form-item label='单价'>
        <el-input v-model.number="item.price" :disabled="found"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="7">
      <el-form-item label='总价'>
        <el-input v-model.number="itemSum" :disabled="true"></el-input>
      </el-form-item>
    </el-col>
    <el-col :span="4">
      <el-form-item>
        <el-button @click="addItem">添加</el-button>
      </el-form-item>
    </el-col>
    <el-table :data="itemList" style="width: 100%" header-cell-class-name="cell" class="el-form-item">
      <el-table-column prop="name" label="药品名称"></el-table-column>
      <el-table-column prop="id" label="药品ID"></el-table-column>
      <el-table-column prop="num" label="数量"></el-table-column>
      <el-table-column prop="price" label="药品单价"></el-table-column>
      <el-table-column prop="sum" label="总价"></el-table-column>
      <el-table-column>
        <template slot-scope="scope">
          <el-button size="mini" type="danger" @click="onDelete(scope.$index)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-form-item></el-form-item>
    <el-form-item label="创建时间">
      <el-col :span="11">
        <el-date-picker placeholder="选择日期" v-model="form.date1" style="width: 100%;" prop="date1"></el-date-picker>
      </el-col>
      <el-col class="line" :span="2">-</el-col>
      <el-col :span="11">
        <el-time-picker placeholder="选择时间" v-model="form.date2" style="width: 100%;" prop="date2"></el-time-picker>
      </el-col>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">立即创建</el-button>
      <el-button>取消</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  name: 'OutStock',
  data () {
    return {
      isSearching: false,
      found: true,
      form: {
        out: '',
        in: '',
        date1: '',
        date2: '',
        itemList: ''
      },
      item: {
        name: '',
        id: '',
        num: 0,
        price: 0
      },
      itemList: [],
      rules: {
        out: [{required: true, message: '请输入出库单位', trigger: 'blur'}],
        in: [{required: true, message: '请输入入库单位', trigger: 'blur'}]
      }
    }
  },
  computed: {
    itemSum: function () {
      return this.item.price * this.item.num
    },
    date: function () {
      let date = this.form.date1
      date.setHours(this.form.date2.getHours())
      date.setMinutes(this.form.date2.getMinutes())
      date.setSeconds(this.form.date2.getSeconds())
      return date
    }
  },
  methods: {
    onSubmit () {
      this.form.itemList = JSON.stringify(this.itemList)
      this.form.date = this.date.toString()
      console.log(this.form)
      this.$axios
        .post('/outStock/itemList', this.form)
        .then(res => {
          if (res.data.status === 'success') {
            console.log(res)
          }
        })
        .catch(err => console.log(err))
    },
    onDelete (index) {
      this.itemList.splice(index, 1)
      console.log(index)
    },
    addItem () {
      let titem = {}
      titem.id = this.item.id; titem.name = this.item.name; titem.num = this.item.num; titem.price = this.item.price
      titem.sum = this.itemSum
      this.itemList.forEach(item => {
        if (item.id === titem.id) {
          item.num += titem.num
          item.sum += titem.sum
          titem = null
        }
      })
      if (titem) this.itemList.push(titem)
      this.item.id = ''; this.item.name = ''; this.item.num = 0; this.item.price = 0
      console.log(titem)
    },
    searchItem () {
      this.isSearching = true
      this.$axios
        .post('/outStock/item', {name: this.item.name})
        .then(res => {
          let data = res.data
          if (data.status === 'success') {
            this.item.id = data.id
            this.item.price = data.price
            this.item.num = 0
          } else {
            this.item.id = '请手动输入'
            this.item.price = 0
            this.item.num = 0
            this.found = false
          }
        })
        .catch(err => console.log(err))
      /* if (this.item.name === '123') {
        this.item.id = 456
        this.item.price = 7
        this.item.num = 0
      } else {
        this.item.id = '请手动输入'
        this.item.price = 0
        this.item.num = 0
        this.found = false
      } */
      this.isSearching = false
    }
  }
}
</script>

<style scoped>

</style>
