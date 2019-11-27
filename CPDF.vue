<template>
  <div id="CPdf">
    <canvas id="the-canvas" style="width:100%;"></canvas>
    <div id="pdfbottom" class="pagestyle">
      <van-icon name="arrow-left" @click="onPrevPage" />
      <span class="pagenum_style">{{page_num}}/{{page_count}}</span>
      <van-icon name="arrow" @click="onNextPage" />
    </div>
  </div>
</template>
<script>
import PDFJS from "../../node_modules/pdfjs-dist/build/pdf.js";
export default {
  data() {
    return {
      pdfDoc: null,
      pageNumPDF: 1,
      pageRendering: false,
      pageNumPending: null,
      page_num: 0, //当前页数
      page_count: 0, //总页数
      pdfjsLib: window["pdfjs-dist/build/pdf"]
    };
  },
  props: {
    pdfUrl: String
  },
  created() {
    this.pageNumPDF = 1;
    this.initPdf();
  },
  computed: {
    ctx() {
      let id = document.getElementById("the-canvas");
      return id.getContext("2d");
    }
  },
  methods: {
    //初始化方法
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
    },
    throwerr(err) {
      this.$emit("pdferr", err);
    },
    //渲染pdf
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
    },
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
    },
    closepdf() {
      //关闭PDF
      this.pdfDoc = null;
      this.$emit("closepdf");
    }
  }
};
</script>

<style scoped>
.pagestyle {
  -moz-transform: none;
  -webkit-transform: none;
  -o-transform: none;
  -ms-transform: none;
  transform: none;
  width: 35%;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
  background-color: rgba(0, 0, 0, 0.5);
  /*background-color: red;*/
  border-radius: 20px;
  /*margin-top: -25%;*/
  position: fixed;
  bottom: 3%;
  left: 58%;
  margin-left: -25%;
}
.pagenum_style {
  font-size: 0.4rem;
  padding: 0 1.3rem 0 1.3rem;
  color: white;
}
</style>
