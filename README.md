# Vue-pdfjs
Vue使用pdfjs展示PDF文件

有两个文件，一个是带分页，另外一个不带有分页
自行选择使用

具体的注解，请参考：http://www.zxzhuxiao.com/index.php/archives/141/

## 一、不带有分页效果
### 1、vue中引入pdfjs依赖：
```
npm install pdfjs-dist --save
```
### 2.使用canvas当预览pdf文件的画布
```
<canvas v-for="page in pages" :id="'the-canvas'+page" :key="page" style="width:100%;"></canvas>
```
### 3.调用pdfjs api进行文档渲染
```
_renderPage (num) {
     this.pdfDoc.getPage(num).then((page) => {
       let canvas = document.getElementById('the-canvas' + num)
      let ctx = canvas.getContext('2d')
       let dpr = window.devicePixelRatio || 1
       let bsr = ctx.webkitBackingStorePixelRatio ||
         ctx.mozBackingStorePixelRatio ||
         ctx.msBackingStorePixelRatio ||
         ctx.oBackingStorePixelRatio ||
         ctx.backingStorePixelRatio || 1
       let ratio = dpr / bsr
       let viewport = page.getViewport(screen.availWidth / page.getViewport(1).width)
       canvas.width = viewport.width * ratio
       canvas.height = viewport.height * ratio
       canvas.style.width = viewport.width + 'px'
       canvas.style.height = viewport.height + 'px'
      ctx.setTransform(ratio, 0, 0, ratio, 0, 0)
       let renderContext = {
         canvasContext: ctx,
         viewport: viewport
       }
       page.render(renderContext)
       if (this.pages > num) {
         this._renderPage(num + 1)
       }
     })
   },
   _loadFile (url) {
     PDFJS.getDocument(url).then((pdf) => {
       this.pdfDoc = pdf
       console.log(pdf)
       this.pages = this.pdfDoc.numPages
       this.$nextTick(() => {
         this._renderPage(1)
       })
     })
   }
```
### 4.使用时传递url
```
this._loadFile(this.pdfUrl);	//这里传递PDF的地址或者base64即可
```
## 二、带有分页效果
### 1、vue中引入pdfjs依赖：
```
npm install pdfjs-dist --save
```
### 2.使用canvas当预览pdf文件的画布
```
<canvas id="the-canvas" style="width:100%;"></canvas>
```
### 3、初始化
```
	initPdf() {
      let vm = this;
      console.log("vm.pdfUrl===", vm.pdfUrl);
      PDFJS.getDocument(vm.pdfUrl)
        .then(function(pdfDoc_) {
          //初始化pdf
          console.log("pdfDoc_===", pdfDoc_);
          // Indicator.close();
          if (pdfDoc_) {
            vm.pdfDoc = pdfDoc_;
            vm.page_count = vm.pdfDoc.numPages;
            vm.renderPage(vm.pageNumPDF);
          } else {
            // Indicator.close();
            vm.showNavbar = false;
            vm.isPageShow = false;
            Toast({
              message: "PDF文档打开失败",
              position: "middle",
              duration: 3000
            });
          }
        })
        .catch(function(err) {
          if (err) {
            console.log(err);
            vm.throwerr(vm.pdfUrl);
          }
        });
    }
```
#### 4、渲染pdf
```
	renderPage(num) {
      let that = this;
      let canvas = document.getElementById("the-canvas");
      that.pageRendering = true;
      // Using promise to fetch the page
      that.pdfDoc.getPage(num).then(function(page) {
        var viewport = page.getViewport(2.0);
        console.log("viewPort================", viewport);
        canvas.height = viewport.height;
        canvas.width = viewport.width;

        // Render PDF page into canvas context
        var renderContext = {
          canvasContext: that.ctx,
          viewport: viewport
        };
        var renderTask = page.render(renderContext);

        // Wait for rendering to finish
        renderTask.promise.then(function() {
          that.pageRendering = false;
          if (that.pageNumPending !== null) {
            // New page rendering is pending
            tat.renderPage(that.pageNumPending);
            that.pageNumPending = null;
          }
        });
      });

      // Update page counters
      that.page_num = num;
    }
```
### 5、分页操作
```
	queueRenderPage(num) {
      if (this.pageRendering) {
        this.pageNumPending = num;
      } else {
        this.renderPage(num);
      }
    },
    onPrevPage() {
      if (this.pageNumPDF <= 1) {
        return;
      }
      this.pageNumPDF--;
      this.queueRenderPage(this.pageNumPDF);
    },
    onNextPage() {
      if (this.pageNumPDF >= this.pdfDoc.numPages) {
        return;
      }
      this.pageNumPDF++;
      this.queueRenderPage(this.pageNumPDF);
    }
```
## 三、页面中引用
```
//传递base64，如果你的base64已带有头部，则不需要像我这样再次添加
<cpdf :pdfUrl="'data:application/pdf;base64,' + detail.dzzs"></cpdf>
//传递URL
<cpdf :pdfUrl="url"></cpdf>
```
