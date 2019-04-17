# request_reponse
## 例一：重定向（设置响应头）
com.my.a_response.a_location
* 问题描述： index.html直接跳转到服务器servlet1(/Loc1),servlet1中doget()response重定向到servlet2(Loc2)
* index.html
```(html)
<body>
	<a href = "/rr/Loc1">a_重定向1（响应头）</a>
</body>
```
* servlet1.java
```(java)
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		System.out.println("我说：没零钱");
		System.out.println("我说：去找2");
		/*
		//方式1：（理解）
		//设置状态码302
		response.setStatus(302);
		//设置响应头
		response.setHeader("location","/rr/Loc2");
		*/
		//方式2：（掌握）
		response.sendRedirect("/rr/Loc2");
	}
```
* servlet2.java
```(java)
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		System.out.println("2说：给你20");
	}

```
## 例二：定时刷新（设置响应头）
* 问题描述：从index.html上跳转到refresh1.html上，然后3秒后跳转到refresh2.html上，要求用js数秒
* 方案一：设置头refresh
* 方案二：html的meta标签
  *
```(html)
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<!-- 
	http-equiv:响应头
	content:响应头value
	 -->
<meta http-equiv="refresh" content="3;/rr/refresh2.html">
<title>Insert title here</title>
</head>
<body>
	xxx成功了，<span id="sid">3</span>秒之后跳转到refresh2.html上。
</body>
<script type="text/javascript">
	onload = function(){
		//设置定时器
		setInterval(changeS,1000);
	}
	//全局变量
	i = 3;
	//时间变化
	function changeS(){
		//获取元素
		var obj = document.getElementById("sid");
		//操作元素标签体
		obj.innerHTML=--i;
	}
</script>
</html>
```
## 例三：让浏览器显示表格
* 问题描述：编写Printservlet.java设置响应体，获得向客户端进行数据输出的流对象，在浏览器上显示表格
* 有中文输出，设置输出数据的编码格式（否则乱码）
* 
```(html)
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//设置处理相应的中文乱码
		response.setContentType("text/html;charset=utf-8");
		//response.setHeader("content-type", "text/html;charset=utf-8");
		//打印表格
		//获取字符流
		PrintWriter w = response.getWriter();
		w.print("<table border='1'><tr>");
		w.print("<td>用户名：</td>");
		w.print("<td>tom</td>");
		w.print("<tr><td>密码：</td>");
		w.print("<td>123</td>");
		w.print("</tr><table>");
	}
```
## 案例1 文件下载案例
* 技术分析：
  * response
  * 文件下载
  
