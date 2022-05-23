# VUE导出easyExcel

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

