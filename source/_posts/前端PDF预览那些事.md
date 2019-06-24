---
title: 前端预览PDF那些事
date: 2018-03-10 00:25:07
tags:
     - PDF
     - 前端
---
+   最近项目中需要在网页上预览 PDF，目前了解到 有``<iframe>、<object>、<embed>``标签 和 pdf.js、pdfObject等几种第三方解决方案， 但还有一些注意事项。
 <!-- more -->

# 一、``<iframe>`` 预览 PDF

在 PDF 预览中``<iframe> ``算是一种古老但高效的解决方案。现在所有浏览器都支持 < iframe > 标签，直接将src设置为指定的PDF文件就可以预览了。src可以是url、bolb 或者 base64,一切问题都容易解决， 关键是要标记 资源类型`` type=“application/pdf” ``

# 二、``<embed>``预览 PDF

``< embed > ``标签定义嵌入的内容，比如插件。但是这个标签没有提供回退方案，与``< iframe > < / iframe > ``
不同，这个标签是自闭合的的，也就是说如果浏览器不支持PDF的嵌入，那么这个标签的内容什么都看不到，经测试在 Safari 下无法预览。用法如下：

```bash

<embed src="/my.pdf" type="application/pdf" width="100%" height="100%">
```

# 三、``<object>``预览 PDF

``< object >``定义了一个嵌入的对象，可以使用此元素向页面添加多媒体。该元素允许您规定插入 HTML 文档中的对象的数据和参数，以及可用来显示和操作数据的代码。用于包含对象，比如图像、音频、视频、Java applets、ActiveX、PDF 以及 Flash。几乎所有主流浏览器都拥有部分对 ``< object > ``标签的支持。这个标签在这里的用法和``< iframe >``很类似，也支持回退；还可以和``<iframe>``结合使用， 为我们提供更强大的回退方案。

> 以上的标签都是无需 JavaScript 支持的 PDF 预览方案， 接下来谈谈 JS 库下的 pdf 预览方案。

# 四、pdf.js 预览 PDF

PDF.js可以实现在html下直接浏览pdf文档，是一款开源的pdf文档读取解析插件，非常强大，能将PDF文件渲染成Canvas。PDF.js主要包含两个库文件，一个pdf.js和一个pdf.worker.js，一个负责API解析，一个负责核心解析。 

由于我是在 React 下使用该库， 因此使用了 [react-pdf-js](https://github.com/mikecousins/react-pdf-js) 这个react 组件实现 pdf.js相同的效果。

具体用法：

```
import React from 'react';
import PDF from 'react-pdf-js';

class MyPdfViewer extends React.Component {
  state = {};

  onDocumentComplete = (pages) => {
    this.setState({ page: 1, pages });
  }

  onPageComplete = (page) => {
    this.setState({ page });
  }

  handlePrevious = () => {
    this.setState({ page: this.state.page - 1 });
  }

  handleNext = () => {
    this.setState({ page: this.state.page + 1 });
  }

  renderPagination = (page, pages) => {
    let previousButton = <li className="previous" onClick={this.handlePrevious}><a href="#"><i className="fa fa-arrow-left"></i> Previous</a></li>;
    if (page === 1) {
      previousButton = <li className="previous disabled"><a href="#"><i className="fa fa-arrow-left"></i> Previous</a></li>;
    }
    let nextButton = <li className="next" onClick={this.handleNext}><a href="#">Next <i className="fa fa-arrow-right"></i></a></li>;
    if (page === pages) {
      nextButton = <li className="next disabled"><a href="#">Next <i className="fa fa-arrow-right"></i></a></li>;
    }
    return (
      <nav>
        <ul className="pager">
          {previousButton}
          {nextButton}
        </ul>
      </nav>
      );
  }

  render() {
    let pagination = null;
    if (this.state.pages) {
      pagination = this.renderPagination(this.state.page, this.state.pages);
    }
    return (
      <div>
        <PDF
          file="somefile.pdf"
          onDocumentComplete={this.onDocumentComplete}
          onPageComplete={this.onPageComplete}
          page={this.state.page}
        />
        {pagination}
      </div>
    )
  }
}

module.exports = MyPdfViewer;

```

### 在实际使用中， 效果确实不错， 还提供 webworker 来实现分页渲染，但因为我目前的项目是要预览含有电子签章的 PDF 文件，
在测试中发现，pdf.js 并不支持该特性，若要显示电子签章， 需要在源文件中更改一些地方， 故此我在项目中放弃使用 pdf.js

# 五、pdfObject

看官网上的介绍，PDFObject并不是一个PDF渲染工具，它也是通过< embed >标签实现PDF预览。PDFObject提供了一个PDFObject.supportsPDFs用于判断该浏览器能否使用PDFObject。

```bash
if(PDFObject.supportsPDFs){
   console.log("Yay, this browser supports inline PDFs.");
} else {
   console.log("Boo, inline PDFs are not supported by this browser");
}

```


这个库我是最近才了解到，本打算看看如何隐藏``<iframe>``预览的 pdf 中的工具栏， 无意中发现该方案，目前尚未尝试，基本用法也很简单:

```bash

<script type="text/javascript">
    if(PDFObject.supportsPDFs){
        // PDF嵌入到网页
        PDFObject.embed("index.pdf", "#pdf_viewer" );
    } else {
        location.href = "/canvas";
    }
</script>
```

# 六、结尾

总的来说，pdf 预览的方案挺多选择的， 适合自己项目的才是最好。 像我的需求是要显示电子签章， 验证多种方案后选择使用 iframe 标签显示。当然你也可以再后端把 pdf 文件转换成图片传回前端显示。

