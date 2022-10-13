- VUE导出easyExcel
```js
function exportExl() {
       const token_ = localStorage.getItem('authorization')
      const options = {
        params: {
         // token: token_,
          createStartTime:this.times?this.times[0]:null,
          createEndTime:this.times?this.times[1]:null,
          responseType:'arraybuffer'
      }
      try {
        const res = await PaysModel.exportExcel(options)
        var blob = new Blob([res], {type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"});
        var objectUrl = URL.createObjectURL(blob);
        var a = document.createElement("a");
        document.body.appendChild(a);
        a.style = "display: none";
        a.href = objectUrl;
        a.download ='银联申请用户列表'
        a.click();
        document.body.removeChild(a);
      } catch (err) {
        console.log(err)
      }
    }
```

- js 直接将返回值输出成文件并下载
```js
//创建一个<a>标签
let ele_a = document.createElement("a");
//文件名
let fileName = 'ES23044.txt';
//h5特性
ele_a.download = fileName;
ele_a.style.display="none";
let blob = new Blob(["文本信息"]);
ele_a.href = URL.createObjectURL(blob);
document.body.appendChild(ele_a);
ele_a.click();
document.body.removeChild(ele_a);
```






