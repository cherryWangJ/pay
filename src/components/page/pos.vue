<template>
  <div class="pos">
    <el-row>
      <el-col :span="8" id="list-height">
        <div class="grid-content bg-purple">
          <el-tabs>
          <el-tab-pane label="点餐">
            <el-table border  width="100%" :data="tableData">
              <el-table-column prop="goodsName" label="商品"></el-table-column>
              <el-table-column prop="count" label="数量" width="70"></el-table-column>
              <el-table-column prop="price" label="价格" width="90"></el-table-column>
              <el-table-column label="操作" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="deleteSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">添加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totals">
              数量 : {{totalCounts}}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;总价： {{totalMoney}} 元
            </div>
            <el-row id="button-top">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="deleteAll">删除</el-button>
              <el-button type="success" @click="checkOut">结账</el-button>
            </el-row>  
          </el-tab-pane>
          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>
        </div>
      </el-col>
      <el-col :span="16">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul v-for="goods in oftenGoods" @click="addOrderList(goods)">
              <li>
                <span>{{goods.goodsName}}</span>
                <span class="o-price">￥{{goods.price}}</span>
              </li>
            </ul>            
          </div>
          </div>
          <div class="goods-type">
            <el-tabs>
              <el-tab-pane label="汉堡">
                <ul class="hanber-list" v-for="goods in typeGoods0" @click="addOrderList(goods)">
                  <li>
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" alt="汉堡"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}</span>
                  </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label="小食">
                <ul class="hanber-list" v-for="goods in typeGoods1" @click="addOrderList(goods)">
                  <li>
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" alt="汉堡"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}</span>
                  </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label="饮料">
                <ul class="hanber-list" v-for="goods in typeGoods2" @click="addOrderList(goods)">
                  <li>
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" alt="汉堡"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}</span>
                  </li>
                </ul>
              </el-tab-pane>
              <el-tab-pane label="套餐">
                <ul class="hanber-list" v-for="goods in typeGoods3" @click="addOrderList(goods)">
                  <li>
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" alt="汉堡"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}</span>
                  </li>
                </ul>
              </el-tab-pane>
            </el-tabs>
          </div>
          </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data(){
    return {
      tableData :[],
      oftenGoods:[],
      typeGoods0:[],
      typeGoods1:[],
      typeGoods2:[],
      typeGoods3:[],
      totalCounts : 0,
      totalMoney : 0
    }
  },
  name: 'pos',
  created : function(){
    axios.get('http://jspang.com/DemoApi/oftenGoods.php')
    .then(response =>{
      this.oftenGoods = response.data;
    })
    .catch(error =>{
      alert('网络错误，请等待加载');
    })

    axios.get('http://jspang.com/DemoApi/typeGoods.php')
    .then(response =>{
      this.typeGoods0 = response.data[0];
      this.typeGoods1 = response.data[1];
      this.typeGoods2 = response.data[2];
      this.typeGoods3 = response.data[3];
    })
    .catch(error =>{
      alert('网络错误，请等待加载')
    })
  },
  mounted : function(){
    var listHeight = document.body.clientHeight;
    document.getElementById('list-height').style.height=listHeight+'px';
  },
  methods : {
    addOrderList(goods){
      //先判断是否在tableData中存在了
      let isHave = false;
      for(let i = 0;i<this.tableData.length;i++){
        if(this.tableData[i].goodsId == goods.goodsId){
          isHave = true;
        }
      }
      //isHave为真的话，改变列表中的count的值
      if(isHave){
        let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      }
      else{
        let newDatas = {goodsId : goods.goodsId,goodsName : goods.goodsName,price : goods.price,count : 1};
        this.tableData.push(newDatas);
      }
      this.getTotals();
    },
    deleteSingleGoods(goods){
      let arr = this.tableData.filter(o =>o.goodsId == goods.goodsId);
      arr[0].count--;
      if(arr[0].count == 0){
           this.tableData = this.tableData.filter(o =>o.goodsId !=goods.goodsId);
      }
      this.getTotals();
    },
    deleteAll(){
      this.totalCounts = 0;
      this.totalMoney = 0;
      this.tableData = [];
    },
    getTotals(){
      this.totalCounts = 0;
      this.totalMoney = 0;
      if(this.tableData){
        this.tableData.forEach(element => {
          this.totalCounts +=element.count;
          this.totalMoney = this.totalMoney + (element.count * element.price);
        });
      }
    },
    checkOut(){
      if(this.totalMoney != 0){
        this.tableData = [];
        this.totalCounts = 0;
        this.totalMoney = 0;
        this.$message({
          message : '结账成功。祝您用餐愉快',
          type : 'success'
        })
      }
      else{
        this.$message.error('不能空结，请您选择商品')
      }
    }
  } 
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#list-height {
  padding-left: 5px;
  border-right: 1px solid #20a0ff;
  text-align: center
}
#button-top{
  margin-top: 10px;
}
.title {
   height: 50px;
   border-bottom: 1px solid #D3DCE6;
   background-color: #F9FAFC;
   padding: 15px;
}
.often-goods-list {
  margin-top: 20px;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border:1px solid #E5E9F2;
  border-radius: 8px;
  padding:10px;
  margin:10px;
  background-color:#fff;
}
.o-price {
  color:#58B7FF;
}
.goods-type {
  clear: both;
  text-align: center;
  padding-left: 10px;
}
.hanber-list li {
  list-style: none;
  width: 30%;
  border: 1px solid #E5E9F2;
  height: auto;
  overflow: hidden;
  background-color:#fff;
  padding: 10px;
  float: left;
  margin: 10px;
}
.hanber-list li span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName{
    font-size: 20px;
    padding-left: 25px;
    color:brown;
 
   }
   .foodPrice{
    padding-left: 40px;
    font-size: 18px;
    padding-top:30px;
   }
.totals {
  background-color: #fff;
  padding: 10px;
  border-bottom: #d6dde2 1px solid;
  font-size: 14px;
  color: #909192;
}
</style>
