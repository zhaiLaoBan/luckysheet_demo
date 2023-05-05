<template>
  <div class="hello">
    <!-- <div style="position: absolute;top:0;">


        <input style="font-size:16px;" type="file" @change="uploadExcel" />

        <span>Or Load remote xlsx file: </span>

        <select v-model="selected" @change="selectExcel">
          <option disabled value="">Choose</option>
          <option v-for="option in options" :key="option.text" :value="option.value">
            {{ option.text }}
          </option>
        </select>

        <a href="javascript:void(0)" @click="downloadExcel">Download source xlsx file</a>
    </div> -->

    <div
      id="luckysheet"
      style="margin:0px;padding:0px;position:absolute;width:100%;left: 0px;top: 0px;bottom:0px;"
    ></div>

    <div v-show="isMaskShow" style="position: absolute;z-index: 1000000;left: 0px;top: 0px;bottom: 0px;right: 0px; background: rgba(255, 255, 255, 0.8); text-align: center;font-size: 40px;align-items:center;justify-content: center;display:flex;">Downloading</div>

  </div>
</template>

<script>
import LuckyExcel from 'luckyexcel'
//导入库export.js 这个文件是es6的，不能在普通的HTML文件直接引入js文件（虽然都是js文件，但是有区别，具体请百度es6与es5）！需要把es6转es5才可以直接引入使用！
import {testaaa,exportExcel} from './export'

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return {
      selected:"",
      excelData: {
        SN: 'M20230601',
        PrintDate: '20230506',
        SampleName: 'test-sample',
        InjectTime: '2023-01-05 10:36:59',
        Plot: [],
        ResultaList: [
        {
            id: 1,
            ResultCpd: 'TVOC', 
            ResultRt: '0.15',
            ResultAmount: 31,
            ResultUnit: 'ppmV'
          },
          {
            id: 2,
            ResultCpd: 'CH4', 
            ResultRt: '0.4',
            ResultAmount: 10,
            ResultUnit: 'ppmV'
          },
          {
            id: 3,
            ResultCpd: 'NMHC', 
            ResultRt: '0.2',
            ResultAmount: 21,
            ResultUnit: 'ppmV'
          }
        ],
        Operator: 'User01'
      },
      options: [
        { text: 'Money Manager.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/money-manager-2.xlsx' },
        { text: 'Activity costs tracker.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/Activity%20costs%20tracker.xlsx' },
        { text: 'House cleaning checklist.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/House%20cleaning%20checklist.xlsx' },
        { text: 'Student assignment planner.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/Student%20assignment%20planner.xlsx' },
        { text: 'Credit card tracker.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/Credit%20card%20tracker.xlsx' },
        { text: 'Blue timesheet.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/Blue%20timesheet.xlsx' },
        { text: 'Student calendar (Mon).xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/Student%20calendar%20%28Mon%29.xlsx' },
        { text: 'Blue mileage and expense report.xlsx', value: 'https://minio.cnbabylon.com/public/luckysheet/Blue%20mileage%20and%20expense%20report.xlsx' },
      ],
      isMaskShow: false,
    }

  },
  mounted() {
    // In some cases, you need to use $nextTick
    // this.$nextTick(() => {
          // $(function () {
          //   luckysheet.create({
          //     container: "luckysheet",
          //     title:'luckysheet233', //表 头名
          //     lang: 'zh', //中文
          //     plugins:['chart'],
          //   });
          // });

    // });

    this.$nextTick(()=>{
      console.log(233);
      this.selectExcel()
    })
  },
  methods:{
    selectExcel(evt){
        const value = '../demo_test.xlsx';
        const name = 'demo_test';

        if(value==""){
            return;
        }
        this.isMaskShow = true;

        LuckyExcel.transformExcelToLuckyByUrl(value, name, (exportJson, luckysheetfile) => {

            if(exportJson.sheets==null || exportJson.sheets.length==0){
                alert("Failed to read the content of the excel file, currently does not support xls files!");
                return;
            }

            console.log(exportJson,'exportJson');

            this.isMaskShow = false;
            window.luckysheet.destroy();
            exportJson.sheets[0].celldata.forEach(item=>{
              let value = item.v.v
              if(value && value.match(/\{(.+?)\}/) !== null){
                let str = item.v.v.match(/\{(.+?)\}/)[1]
                if(!Array.isArray(this.excelData[str])){
                  item.v.v = value.replace('{'+str+'}',this.excelData[str])
                }
                
              }
            })
            window.luckysheet.create({
                container: 'luckysheet', //luckysheet is the container id
                showinfobar:false,
                data:exportJson.sheets,
                title:exportJson.info.name,
                userInfo:exportJson.info.name.creator,
                lang: 'zh', //中文
                plugins:['chart'],
            });
        });
    },

    uploadExcel(evt){
        const files = evt.target.files;
        if(files==null || files.length==0){
            alert("No files wait for import");
            return;
        }

        let name = files[0].name;
        let suffixArr = name.split("."), suffix = suffixArr[suffixArr.length-1];
        if(suffix!="xlsx"){
            alert("Currently only supports the import of xlsx files");
            return;
        }
        LuckyExcel.transformExcelToLucky(files[0], function(exportJson, luckysheetfile){

          debugger
            if(exportJson.sheets==null || exportJson.sheets.length==0){
                alert("Failed to read the content of the excel file, currently does not support xls files!");
                return;
            }
            // window.luckysheet.destroy();
            debugger
            console.log('exportJson',exportJson);
            window.luckysheet.create({
                container: 'luckysheet', //luckysheet is the container id
                // showinfobar:false,
                data:exportJson.sheets,
                title:exportJson.info.name,
                // userInfo:exportJson.info.name.creator,
                // forceCalculation: false,
                plugins:['chart']
            });
        });
    },
    downloadExcel(){
          // const value = this.selected;;
          //
          // if(value.length==0){
          //     alert("Please select a demo file");
          //     return;
          // }
          //
          // var elemIF = document.getElementById("Lucky-download-frame");
          // if(elemIF==null){
          //     elemIF = document.createElement("iframe");
          //     elemIF.style.display = "none";
          //     elemIF.id = "Lucky-download-frame";
          //     document.body.appendChild(elemIF);
          // }
          // elemIF.src = value;
      exportExcel(luckysheet.getAllSheets(),"下载")
      // testaaa();

    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
