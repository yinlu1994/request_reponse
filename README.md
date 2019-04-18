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
## 扩展：通过response生成验证码
* 验证码：作用：防止暴力攻击
* code.html
```(html)
<body>
	<img alt="验证码" src="/rr/codes" title="看不清楚换一张">
</body>
```
* codeServlet.java(在web.xml里配置为/codes)
```(java)
package com.my.a_response.d_code;

import java.awt.Color;
import java.awt.Font;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.util.Random;

import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CodeServlet extends HttpServlet {

		public void doGet(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {

			// 使用java图形界面技术绘制一张图片

			int charNum = 4;
			int width = 30 * 4;
			int height = 30;

			// 1. 创建一张内存图片
			BufferedImage bufferedImage = new BufferedImage(width, height,BufferedImage.TYPE_INT_RGB);

			// 2.获得绘图对象
			Graphics graphics = bufferedImage.getGraphics();

			// 3、绘制背景颜色
			graphics.setColor(Color.YELLOW);
			graphics.fillRect(0, 0, width, height);

			// 4、绘制图片边框
			graphics.setColor(Color.BLUE);
			graphics.drawRect(0, 0, width - 1, height - 1);

			// 5、输出验证码内容
			graphics.setColor(Color.RED);
			graphics.setFont(new Font("宋体", Font.BOLD, 20));

			// 随机输出4个字符
			Graphics2D graphics2d = (Graphics2D) graphics;
			 String s = "ABCDEFGHGKLMNPQRSTUVWXYZ23456789";
			Random random = new Random();
			//session中要用到
			String msg="";
			int x = 5;
			for (int i = 0; i < 4; i++) {
				int index = random.nextInt(32);
				String content = String.valueOf(s.charAt(index));
				msg+=content;
				double theta = random.nextInt(45) * Math.PI / 180;
				//让字体扭曲
	            graphics2d.rotate(theta, x, 18);
				graphics2d.drawString(content, x, 18);
				graphics2d.rotate(-theta, x, 18);
				x += 30;
			}

			// 6、绘制干扰线
			graphics.setColor(Color.GRAY);
			for (int i = 0; i < 5; i++) {
				int x1 = random.nextInt(width);
				int x2 = random.nextInt(width);

				int y1 = random.nextInt(height);
				int y2 = random.nextInt(height);
				graphics.drawLine(x1, y1, x2, y2);
			}

			// 释放资源
			graphics.dispose();

			// 图片输出 ImageIO
			ImageIO.write(bufferedImage, "jpg", response.getOutputStream());
		

		}
	

	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

	}

}

```
  
