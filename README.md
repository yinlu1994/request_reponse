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
* 下载方式
  * 超链接下载
  * 编码下载：通过servlet完成
    * 设置文件的mime类型
      * 获取文件类型：String mimeType = context.getMimeType(文件名)
      * 设置文件类型：response.setContentType(mimeType);
    * 设置下载头信息 content-disposition
      * response.setHeader("content-disposition","attachment;filename="+文件名称);
    * 提供流(以流的方式返回给浏览器)
      * 要先从本地获取一个输入流，再给他输出出去
      * response.getOutputStream();
 * DownloadServlet.java
 ```(java)
 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//文件下载
		//1.设置文献下载的mime类型
		//1.1获取下载文件的名称
		String filename = request.getParameter("name");
		//1.2获取文件类型
		ServletContext context = this.getServletContext();
		String mimeType = context.getMimeType(filename);
		
		//2.设置下载的头信息
		response.setHeader("content-disposition", "attchment;filename="+filename);
		//3.对拷流
		//3.1获取输入流,以流的方式返回一个文件
		InputStream is = context.getResourceAsStream("/download/"+filename);
		//3.2获取输出流，向浏览器写东西
		ServletOutputStream os = response.getOutputStream();
		//3.3对拷流
		int len = -1;
		byte[] b = new byte[1024];
		while((len = is.read(b)) != -1) {
			os.write(b,0,len);
		}
		os.close();
		is.close();
	}
 ```
 * 可以用commons-io-1.4jar包省略部分代码
 ```(jar)
 		//3.3对拷流
		/*
		 * int len = -1;
		byte[] b = new byte[1024];
		while((len = is.read(b)) != -1) {
			os.write(b,0,len);
		}
		 */
		
		IOUtils.copy(is, os);
 ```
 * download.html
 ```(html)
 <body>
	<a href="/rr/download?name=download1.doc">download1.doc</a>
	<a href="/rr/download?name=download2.jpg">download2.jpg</a>
</body>
```
  
  
