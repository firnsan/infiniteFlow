# infiniteFlow
下拉获取数据

类的实现：
```js
function oInfiniteFlow(elId,type, getDataFunc) {
	this.el= $(elId)
	this.type=type;
	this.getData= getDataFunc;
	
	var that=this;
	this.addEventHandle= function(){
		var winHeight= $(window).height();
		$(window).bind("scroll",function(){
			if (that.isNoData || that.el.is(':hidden'))
				return;
			//that.isPending=true;//及时锁住
			var scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
			var pageHeight = $(document.body).height();
			var thres= (pageHeight-winHeight-scrollTop)/winHeight;
			if (thres<0.02) {
				that.getData();
			}
		});
	}
	this.getPage= function(){
		return this.page;
	}
	this.getPageSize= function(){
		return this.pagesize;
	}

	this.loading= function(){
		if (this.isPending)
			return false; 
		this.isPending=true;
		this.el.append('<li><div class="loading"></div></li>'); 
		return true;
	}
	this.reset= function(){
		this.page=0;
		this.pagesize=15;
		this.resultBuf= new Array();
		this.isNoData= false;
		this.isPending= false;
		this.el.empty();
	}
	this.display= function(newData){
		this.resultBuf= this.resultBuf.concat(newData);
		var resultBuf= this.resultBuf;
		var page= this.page;
		var pagesize= this.pagesize;
		var type= this.type;
		var content = '';
		for (var i= page*pagesize; i<(page+1)*pagesize && resultBuf[i]; i++){
			content += genContent(resultBuf[i],type);
		}
		this.el.find(".loading").remove();
		this.el.append(content);
		this.page++;
		if (newData.length<pagesize)
			this.noData();
		//稍后再释放锁
		setTimeout(function(){that.isPending=false},500);
	}
	

	this.noData= function(){
		this.isNoData= true;
		this.el.find(".loading").remove();
		this.el.append('<li><div class="nodata"></div></li>');
		that.pending=false;
	}
	this.reset();
	this.addEventHandle();
}

```


用法：
```js

//获取想去、去过XX地方的人或队伍
var urlTable={'loved':"/Travel/ajaxFindPersons",
		'visited':"/Travel/ajaxGetVisited",
		'teams':"/Travel/ajaxFindTeams"};
function commonGet(type,from,go,displayer){
	if (!displayer.loading())
		return; //如果被另外一个任务锁住,则退出
	var url=APP+urlTable[type];
	if (!url)
		return;
	$.post(url,
			{'from':from,'go':go,'page':displayer.getPage(),'page_size':displayer.getPageSize()},
			function(serverData, status) {
				var allData = eval(serverData);
				if (allData.status) {
					displayer.noData();
					return;
				}
				displayer.display(allData.data);
			}
		)
}

var infiniteFlow1= new oInfiniteFlow('#lPersons','loved',function(){
	commonGet('loved',globalFrom,globalGo,this);
});
```
