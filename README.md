# SpreadJS_-DynamicColumnDataBinding
动态列数据绑定
### SpreadJS 示例，基于 JavaScript组件实现包含合并单元格的数据绑定

该示例包括使用 SpreadJS API 的演示脚本，可用于实现包含合并单元格的数据绑定。
有关 SpreadJS API 的更多信息，请参阅[SpreadJS API指南]( https://demo.grapecity.com.cn/spreadjs/help/api/) 和[帮助手册]( https://help.grapecity.com.cn/pages/viewpage.action?pageId=5963808)。
 

目录：
-	运行步骤
-	控件初始化
-	示例代码
-	关于 SpreadJS
外部文件：
-	临时授权申请



### 运行步骤
1、在开始之前，请确保您已满足以下先决条件：

要运行 SpreadJS，浏览器必须支持 HTML5，客户端导入和导出 Excel 需要 IE10及以上。请先了解 [SpreadJS 的产品使用环境]( https://www.grapecity.com.cn/developer/spreadjs/selection-guide/product-use-environment)，并申请临时部署授权激活
安装并更新NodeJS和NPM
2、克隆或下载此代码库
3、初始化控件，并运行示例脚本

#### 控件初始化
1、	首先，创建一个新页面，并在页面上输入以下代码：
```
<!DOCTYPE html>
    <html>
    <head>
        <title>Spread HTML test page</title>
```
2、在页面中添加对 Spread.JS 的引用。代码如下。需要注意的是，Spread 提供压缩过
```
（minified）的 JavaScript 文件和和用于调试的文件：
<script src="[Your_Scripts_Path]/gc.spread.sheets.all.xxxx.min.js" type="text/javascript"></script>
```

3、添加 CSS 文件以改变Spread.JS 的外观。默认的CSS文件名为： 
gc.spread.sheets.xxxx.css，里面包含了所有的默认样式。该 CSS 文件将会影响滚动条，筛选框及其子元素，单元格和下方标签栏的样式。引入 CSS 的代码如下：

```
//<link href="[Your_CSS_Path]/gc.spread.sheets.xxxx.css" rel="stylesheet" type="text/css"/>
//OR
<link href="[Your_CSS_Path]/bootstrap/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="[Your_CSS_Path]/bootstrap/bootstrap-theme.min.css" rel="stylesheet" type="text/css"/>
```
4、添加产品授权，代码为：
```
GC.Spread.Sheets.LicenseKey = "xxx";
```
5. 添加控件初始化代码。本例会在一个 id 为“ss”的 DOM 元素上初始化 Spread.Sheets：
```
<script type="text/javascript">
// Add your license
 GC.Spread.Sheets.LicenseKey = "xxx";
// Add your code
 window.onload = function(){
var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"),{sheetCount:3});
var sheet = spread.getActiveSheet();
 }
</script>
</head>
<body>
```
6、 创建一个 id 为 “ss”的元素，Spread.Sheets 将在该 DOM 中初始化：
```
<div id="ss" style="height: 500px; width: 800px"></div>
</body>
</html>
```
#### 示例代码
```
HTML：
    <p>表格动态列数据绑定</p>
    <button id="bind_btn">绑定</button>
    <div id='ss'></div>
CSS：
    #ss {
        height: 400px;
        width: 100%
    }
    p{
        color: #336699;
        text-align: center;
    }
    button{
        margin-bottom: 6px;
    }
JavaScript：
    GC.Spread.Common.CultureManager.culture("zh-cn")
    $(document).ready(() => {
        // 创建空表单
            let spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"));
    		spread.fromJSON(template);
    		
    		// 点击按钮，执行绑定。
            $("#bind_btn").click(function () {
    			// 挂起绘制
                spread.suspendPaint();
    			// 挂起公式计算
    			spread.suspendCalcService(false);
    			// 获取sheet
    			let sheet = spread.getActiveSheet();
    			// 挂起脏数据
    			sheet.suspendDirty();
    			
    			// 获取动态列的列标
    			let cols = [];
    			let startColIndex = 0;
    			for(let i=0; i<sheet.getColumnCount(); i++){
    				if(sheet.getTag(-1, i) == "dynamicColumn"){
    					if(startColIndex === 0){
    						startColIndex = i;
    					}
    					cols.push(i);
    				}
    			}
    			
    			// 分析data，插入对应的列数
    			let gongsi = data.table[0].gongsi;
    			let table = sheet.tables.all()[0];
    			let range = table.range();
    			let addColCount = 0;
    			for(let i=0; i<gongsi.length-1; i++){
    				sheet.addColumns(range.col + range.colCount, cols.length);
    				addColCount += cols.length;
    			}
    			
    			// 扩展表格区域
    			range.colCount += addColCount;
    			sheet.tables.resize(table, range);
    			spread.commandManager().execute({
    				cmd: "clipboardPaste", 
    				sheetName: sheet.name(),
    				fromSheet: sheet, 
    				// 表头行区域可以通过加tag的方式做标记，这里作为示例直接hardcode
    				fromRanges: [new GC.Spread.Sheets.Range(1,5, 2, 2)], 
    				pastedRanges: [new GC.Spread.Sheets.Range(1,5+2, 2, addColCount)], 
    				isCutting: false,  
    				pasteOption: GC.Spread.Sheets.ClipboardPasteOptions.all
    			});
    			data.table[0].gongsi.forEach(function(item, index){
    				sheet.setValue(1, index*2+startColIndex, item.name);
    			});
    			
    			// 整理数据
    			data.table.forEach(function(item){
    				for(let i=0; i<item.gongsi.length; i++){
    					item["gongsi_"+item.gongsi[i].name+"_pingjia"] = item.gongsi[i].pingjia;
    					item["gongsi_"+item.gongsi[i].name+"_quanzhong"] = item.gongsi[i].quanzhong;
    				}
    			});
    			
    			let dataItem = data.table[0];
    			let columns = [];
    			for(let key in dataItem){
    				if(key.startsWith("gongsi_")){
    					columns.push(key);
    				}
    			}
    			for(let i=range.col, j=0; i<range.col+range.colCount; i++){
    				if(table.getColumnDataField(i).length == 0){
    					j = j == 0? i : j;
    					table.setColumnDataField(i, columns[i-j-range.col]);
    				}
    			}
    			
    			// 执行绑定
    			let dataSource = new GC.Spread.Sheets.Bindings.CellBindingSource(data);
    			sheet.setDataSource(dataSource);
    			
    			// 自动合并
    			let mergeRange = table.dataRange();
    			mergeRange.col = 1;
    			mergeRange.colCount = 2;
    			sheet.autoMerge(mergeRange, GC.Spread.Sheets.AutoMerge.AutoMergeDirection.rowColumn);
    			
    			sheet.resumeDirty();
    			spread.resumePaint();
    			spread.resumeCalcService(true);
            });
    })
    
    
    
    
    
    var template = {"version":"13.0.2","tabStripRatio":0.6,"customList":[],"sheets":{"三级项目":{"name":"三级项目","rowCount":9,"columnCount":8,"activeRow":6,"activeCol":7,"zoomFactor":0.9,"theme":{"name":"Office","themeColor":{"name":"Office","background1":{"a":255,"r":255,"g":255,"b":255},"background2":{"a":255,"r":238,"g":236,"b":225},"text1":{"a":255,"r":0,"g":0,"b":0},"text2":{"a":255,"r":31,"g":73,"b":125},"accent1":{"a":255,"r":79,"g":129,"b":189},"accent2":{"a":255,"r":192,"g":80,"b":77},"accent3":{"a":255,"r":155,"g":187,"b":89},"accent4":{"a":255,"r":128,"g":100,"b":162},"accent5":{"a":255,"r":75,"g":172,"b":198},"accent6":{"a":255,"r":247,"g":150,"b":70},"hyperlink":{"a":255,"r":0,"g":0,"b":255},"followedHyperlink":{"a":255,"r":128,"g":0,"b":128}},"headingFont":"Cambria","bodyFont":"Calibri"},"data":{"dataTable":{"0":{"0":{"value":"三级项目项目公司“千分制”制度落实评价打分汇总表","style":"__builtInStyle31"},"1":{"style":"__builtInStyle31"},"2":{"style":"__builtInStyle31"},"3":{"style":"__builtInStyle31"},"4":{"style":"__builtInStyle31"},"5":{"style":"__builtInStyle31"},"6":{"style":"__builtInStyle31"}},"1":{"0":{"value":"序号","style":"__builtInStyle25"},"1":{"value":"类型","style":"__builtInStyle25"},"2":{"value":"部门","style":"__builtInStyle25"},"3":{"value":"因子系数","style":"__builtInStyle27"},"4":{"value":"权重","style":"__builtInStyle27"},"5":{"value":"公司名称","style":{"parentName":"__builtInStyle32"}},"6":{"style":"__builtInStyle32"}},"2":{"0":{"style":"__builtInStyle26"},"1":{"style":"__builtInStyle26"},"2":{"style":"__builtInStyle26"},"3":{"style":"__builtInStyle28"},"4":{"style":"__builtInStyle28"},"5":{"value":"评价分","style":"__builtInStyle13"},"6":{"value":"权重得分","style":"__builtInStyle3"}},"5":{"0":{"value":"合计","style":{"hAlign":1}},"3":{"value":0,"formula":"SUBTOTAL(109,Table1[yinzi])"},"4":{"value":0,"formula":"SUBTOTAL(109,Table1[quanzhong])"},"5":{"value":0,"formula":"SUBTOTAL(109,Table1[Column5])"},"6":{"value":0,"formula":"SUBTOTAL(109,Table1[Column6])"}}},"columnDataArray":[null,null,null,{"style":"__builtInStyle11"},{"style":"__builtInStyle11"},{"style":"__builtInStyle2","tag":"dynamicColumn"},{"style":"__builtInStyle2","tag":"dynamicColumn"},{"style":"__builtInStyle2"}],"defaultDataNode":{"style":{"backColor":null,"foreColor":"Text 1 0","vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"diagonalDown":null,"diagonalUp":null}}},"rowHeaderData":{"defaultDataNode":{"style":{"themeFont":"Body"}}},"colHeaderData":{"defaultDataNode":{"style":{"themeFont":"Body"}}},"rows":[{"size":34},{"size":35},{"size":31}],"columns":[{"size":30},{"size":29},{"size":111},{"size":61},{"size":61},{"size":66},{"size":77},{"size":63}],"leftCellIndex":0,"topCellIndex":0,"spans":[{"row":1,"rowCount":1,"col":5,"colCount":2},{"row":1,"rowCount":2,"col":0,"colCount":1},{"row":1,"rowCount":2,"col":1,"colCount":1},{"row":1,"rowCount":2,"col":2,"colCount":1},{"row":1,"rowCount":2,"col":3,"colCount":1},{"row":1,"rowCount":2,"col":4,"colCount":1},{"row":0,"rowCount":1,"col":0,"colCount":7},{"row":5,"rowCount":1,"col":0,"colCount":2}],"selections":{"0":{"row":6,"rowCount":1,"col":7,"colCount":1},"length":1},"defaults":{"colHeaderRowHeight":20,"colWidth":64,"rowHeaderColWidth":40,"rowHeight":20},"cellStates":{},"outlineColumnOptions":{},"autoMergeRangeInfos":[],"printInfo":{"margin":{"top":72,"bottom":72,"left":67,"right":67,"header":29,"footer":29},"orientation":2,"pageOrder":1,"paperSize":{"width":826.7716535433073,"height":1169.2913385826773,"kind":9}},"tables":[{"name":"Table1","row":3,"col":0,"rowCount":3,"colCount":7,"showHeader":false,"showFooter":true,"useFooterDropDownList":true,"showResizeHandle":true,"style":{"name":"表样式 1","wholeTableStyle":{"borderLeft":{"color":"Text 1","style":1},"borderTop":{"color":"Text 1","style":1},"borderRight":{"color":"Text 1","style":1},"borderBottom":{"color":"Text 1","style":1},"borderHorizontal":{"color":"Text 1","style":1},"borderVertical":{"color":"Text 1","style":1}}},"autoGenerateColumns":false,"bindingPath":"table","rowFilter":{"range":{"row":3,"rowCount":2,"col":0,"colCount":7},"typeName":"HideRowFilter","dialogVisibleInfo":{},"filterButtonVisibleInfo":{},"showFilterButton":false},"columns":[{"name":"xuhao","dataField":"xuhao","footerValue":"合计"},{"id":1,"name":"leixing","dataField":"leixing"},{"id":2,"name":"bumen","dataField":"bumen"},{"id":3,"name":"yinzi","dataField":"yinzi","footerFormula":"SUBTOTAL(109,Table1[yinzi])"},{"id":4,"name":"quanzhong","dataField":"quanzhong","footerFormula":"SUBTOTAL(109,Table1[quanzhong])"},{"id":5,"name":"Column5","footerFormula":"SUBTOTAL(109,Table1[Column5])"},{"id":6,"name":"Column6","footerFormula":"SUBTOTAL(109,Table1[Column6])"}]}],"index":0}},"namedStyles":[{"backColor":"Accent 1 80","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"20% - Accent1"},{"backColor":"Accent 2 80","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"20% - Accent2"},{"backColor":"Accent 3 80","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"20% - Accent3"},{"backColor":"Accent 4 80","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"20% - Accent4"},{"backColor":"Accent 5 80","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"20% - Accent5"},{"backColor":"Accent 6 80","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"20% - Accent6"},{"backColor":"Accent 1 60","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"40% - Accent1"},{"backColor":"Accent 2 60","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"40% - Accent2"},{"backColor":"Accent 3 60","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"40% - Accent3"},{"backColor":"Accent 4 60","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"40% - Accent4"},{"backColor":"Accent 5 60","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"40% - Accent5"},{"backColor":"Accent 6 60","foreColor":"Text 1 0","font":"14.7px Calibri","themeFont":"Body","name":"40% - Accent6"},{"backColor":"Accent 1 40","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"60% - Accent1"},{"backColor":"Accent 2 40","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"60% - Accent2"},{"backColor":"Accent 3 40","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"60% - Accent3"},{"backColor":"Accent 4 40","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"60% - Accent4"},{"backColor":"Accent 5 40","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"60% - Accent5"},{"backColor":"Accent 6 40","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"60% - Accent6"},{"backColor":"Accent 1 0","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"Accent1"},{"backColor":"Accent 2 0","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"Accent2"},{"backColor":"Accent 3 0","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"Accent3"},{"backColor":"Accent 4 0","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"Accent4"},{"backColor":"Accent 5 0","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"Accent5"},{"backColor":"Accent 6 0","foreColor":"Background 1 0","font":"14.7px Calibri","themeFont":"Body","name":"Accent6"},{"backColor":"#ffc7ce","foreColor":"#9c0006","font":"14.7px Calibri","themeFont":"Body","name":"Bad"},{"backColor":"#f2f2f2","foreColor":"#fa7d00","font":"bold 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#7f7f7f","style":1},"borderTop":{"color":"#7f7f7f","style":1},"borderRight":{"color":"#7f7f7f","style":1},"borderBottom":{"color":"#7f7f7f","style":1},"name":"Calculation","diagonalDown":null,"diagonalUp":null},{"backColor":"#a5a5a5","foreColor":"Background 1 0","font":"bold 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#3f3f3f","style":6},"borderTop":{"color":"#3f3f3f","style":6},"borderRight":{"color":"#3f3f3f","style":6},"borderBottom":{"color":"#3f3f3f","style":6},"name":"Check Cell","diagonalDown":null,"diagonalUp":null},{"backColor":null,"formatter":"_(* #,##0.00_);_(* (#,##0.00);_(* \"-\"??_);_(@_)","name":"Comma"},{"backColor":null,"formatter":"_(* #,##0_);_(* (#,##0);_(* \"-\"_);_(@_)","name":"Comma [0]"},{"backColor":null,"formatter":"_(\"$\"* #,##0.00_);_(\"$\"* (#,##0.00);_(\"$\"* \"-\"??_);_(@_)","name":"Currency"},{"backColor":null,"formatter":"_(\"$\"* #,##0_);_(\"$\"* (#,##0);_(\"$\"* \"-\"_);_(@_)","name":"Currency [0]"},{"backColor":null,"foreColor":"#7f7f7f","font":"italic 14.7px Calibri","themeFont":"Body","name":"Explanatory Text"},{"backColor":"#c6efce","foreColor":"#006100","font":"14.7px Calibri","themeFont":"Body","name":"Good"},{"backColor":null,"foreColor":"Text 2 0","font":"bold 20px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":{"color":"Accent 1 0","style":5},"name":"Heading 1","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 2 0","font":"bold 17.3px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":{"color":"Accent 1 50","style":5},"name":"Heading 2","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 2 0","font":"bold 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":{"color":"Accent 1 40","style":2},"name":"Heading 3","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 2 0","font":"bold 14.7px Calibri","themeFont":"Body","name":"Heading 4"},{"backColor":"#ffcc99","foreColor":"#3f3f76","font":"14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#7f7f7f","style":1},"borderTop":{"color":"#7f7f7f","style":1},"borderRight":{"color":"#7f7f7f","style":1},"borderBottom":{"color":"#7f7f7f","style":1},"name":"Input","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"#fa7d00","font":"14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":{"color":"#ff8001","style":6},"name":"Linked Cell","diagonalDown":null,"diagonalUp":null},{"backColor":"#ffeb9c","foreColor":"#9c6500","font":"14.7px Calibri","themeFont":"Body","name":"Neutral"},{"backColor":null,"foreColor":"Text 1 0","hAlign":3,"vAlign":1,"font":"14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"Normal","diagonalDown":null,"diagonalUp":null},{"backColor":"#ffffcc","borderLeft":{"color":"#b2b2b2","style":1},"borderTop":{"color":"#b2b2b2","style":1},"borderRight":{"color":"#b2b2b2","style":1},"borderBottom":{"color":"#b2b2b2","style":1},"name":"Note","diagonalDown":null,"diagonalUp":null},{"backColor":"#f2f2f2","foreColor":"#3f3f3f","font":"bold 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#3f3f3f","style":1},"borderTop":{"color":"#3f3f3f","style":1},"borderRight":{"color":"#3f3f3f","style":1},"borderBottom":{"color":"#3f3f3f","style":1},"name":"Output","diagonalDown":null,"diagonalUp":null},{"backColor":null,"formatter":"0%","name":"Percent"},{"backColor":null,"foreColor":"Text 2 0","font":"bold 24px Cambria","themeFont":"Headings","name":"Title"},{"backColor":null,"foreColor":"Text 1 0","font":"bold 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":{"color":"Accent 1 0","style":1},"borderRight":null,"borderBottom":{"color":"Accent 1 0","style":6},"name":"Total","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"#ff0000","font":"14.7px Calibri","themeFont":"Body","name":"Warning Text"},{"backColor":null,"foreColor":"Text 1 0","hAlign":3,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle1","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle2","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle3","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle4","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle5","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle6","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle7","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle8","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","formatter":"0.0_);[Red]\\(0.0\\)","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle9","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","formatter":"0.0_);[Red]\\(0.0\\)","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle10","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":3,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","formatter":"0.0_);[Red]\\(0.0\\)","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle11","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle12","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle13","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle14","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":3,"vAlign":1,"font":"normal normal 20px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle15","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","formatter":"0.00%","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle16","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle17","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle18","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle19","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle20","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":0,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle21","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle22","diagonalDown":null,"diagonalUp":null,"isVerticalText":true,"textOrientation":-165},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":null,"borderRight":{"color":"#000000","style":1},"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle23","diagonalDown":null,"diagonalUp":null,"isVerticalText":true,"textOrientation":-165},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":null,"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle24","diagonalDown":null,"diagonalUp":null,"isVerticalText":true,"textOrientation":-165},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle25","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":null,"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle26","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","formatter":"0.0_);[Red]\\(0.0\\)","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle27","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","formatter":"0.0_);[Red]\\(0.0\\)","borderLeft":{"color":"#000000","style":1},"borderTop":null,"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle28","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle29","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle30","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 18.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":null,"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle31","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle32","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle33","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":true,"name":"__builtInStyle34","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle35","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":null,"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle36","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":{"color":"#000000","style":1},"borderTop":{"color":"#000000","style":1},"borderRight":null,"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle37","diagonalDown":null,"diagonalUp":null},{"backColor":null,"foreColor":"Text 1 0","hAlign":1,"vAlign":1,"font":"normal normal 14.7px Calibri","themeFont":"Body","borderLeft":null,"borderTop":{"color":"#000000","style":1},"borderRight":{"color":"#000000","style":1},"borderBottom":{"color":"#000000","style":1},"locked":true,"textIndent":0,"wordWrap":false,"name":"__builtInStyle38","diagonalDown":null,"diagonalUp":null}],"designerBindingPathSchema":{"$schema":"http://json-schema.org/draft-04/schema#","properties":{"table":{"dataFieldType":"table","type":"array","items":{"type":"object","properties":{"xuhao":{"dataFieldType":"text","type":"string"},"leixing":{"type":"string"},"bumen":{"type":"string"},"yinzi":{"type":"string"},"quanzhong":{"type":"string"}}}},"field-1":{"dataFieldType":"text","type":"string"},"field-2":{"dataFieldType":"text","type":"string"},"field-3":{"dataFieldType":"text","type":"string"}},"type":"object"}}
    var data = {
    			table: [
    				{
    					xuhao: 1,
    					leixing: "职能部门",
    					bumen: "办公室",
    					yinzi: 0.8,
    					quanzhong: 0.0625,
    					gongsi: [{
    						name: "1公局1公司项目得分",
    						pingjia: 10,
    						quanzhong: 100
    					},{
    						name: "2公局2公司项目得分",
    						pingjia: 20,
    						quanzhong: 200
    					},{
    						name: "3公局3公司项目得分",
    						pingjia: 30,
    						quanzhong: 300
    					}]
    				},
    				{
    					xuhao: 2,
    					leixing: "职能部门",
    					bumen: "党委",
    					yinzi: 0.8,
    					quanzhong: 0.0625,
    					gongsi: [{
    						name: "1公局1公司项目得分",
    						pingjia: 10,
    						quanzhong: 100
    					},{
    						name: "2公局2公司项目得分",
    						pingjia: 20,
    						quanzhong: 200
    					},{
    						name: "3公局3公司项目得分",
    						pingjia: 30,
    						quanzhong: 300
    					}]
    				},
    				{
    					xuhao: 3,
    					leixing: "职能部门",
    					bumen: "人力资源",
    					yinzi: 0.8,
    					quanzhong: 0.0625,
    					gongsi: [{
    						name: "1公局1公司项目得分",
    						pingjia: 10,
    						quanzhong: 100
    					},{
    						name: "2公局2公司项目得分",
    						pingjia: 20,
    						quanzhong: 200
    					},{
    						name: "3公局3公司项目得分",
    						pingjia: 30,
    						quanzhong: 300
    					}]
    				},
    				{
    					xuhao: 4,
    					leixing: "资源中心",
    					bumen: "资源中心",
    					yinzi: 0.8,
    					quanzhong: 0.0625,
    					gongsi: [{
    						name: "1公局1公司项目得分",
    						pingjia: 10,
    						quanzhong: 100
    					},{
    						name: "2公局2公司项目得分",
    						pingjia: 20,
    						quanzhong: 200
    					},{
    						name: "3公局3公司项目得分",
    						pingjia: 30,
    						quanzhong: 300
    					}]
    				},
    				{
    					xuhao: 5,
    					leixing: "事业部",
    					bumen: "施工部",
    					yinzi: 0.8,
    					quanzhong: 0.0625,
    					gongsi: [{
    						name: "1公局1公司项目得分",
    						pingjia: 10,
    						quanzhong: 100
    					},{
    						name: "2公局2公司项目得分",
    						pingjia: 20,
    						quanzhong: 200
    					},{
    						name: "3公局3公司项目得分",
    						pingjia: 30,
    						quanzhong: 300
    					}]
    				}
    			]
    		}
```
#### 关于 SpreadJS
[SpreadJS]( https://www.grapecity.com.cn/developer/spreadjs) 是一款基于 HTML5 的纯前端表格控件，兼容 450 多种 Excel 公式，具备“高性能、跨平台、与 Excel 高度兼容”的产品特性。使用 SpreadJS，可直接在 Angular、 React、 Vue 等前端框架中实现高效的模板设计、在线编辑和数据绑定等功能，为最终用户提供高度类似 Excel 的使用体验。
 

