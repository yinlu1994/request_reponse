# request_reponse
## 例一：重定向
com.my.a_response.a_location
* 问题描述： index.html直接跳转到服务器servlet1,servlet1中doget()response重定向到servlet2
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
## 案例1 文件下载案例
* 技术分析：
  * response
  * 文件下载
  
