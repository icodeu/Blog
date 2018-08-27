---
layout: post
title: "浅谈用 Java 搭建 web 服务器之 Socket 编程——更好的理解 Apache、Tomcat 等软件的工作模式"
date: 2014-10-18 15:20:33 +0800
comments: true
tags: [Java, Socket]
---

之前做web应用一直是在本地装个Apache、Tomcat之类的软件，然后把做好的网页文件放在他们的工作目录下（如Apache的htdocs），然后打开浏览器输入127.0.0.1或localhost就可以直接访问了，好神奇，可是为什么，怎么实现的呢，早就知道有Socket（套接字）这个东西，可之前就是没有把这两方面结合起来，今天我们就一起来看一看这究竟是为什么。

<!--more-->

有同学说还不懂Socket是什么，这东西很抽象，在计算机网络原理里讲协议时才会看到，今天咱们完全忽略太严谨、学术的定义，就来看看Socket到底是什么。想象一下，你把电脑的电源插在插座上，你的电脑就可以使用了，为什么？“这不是废话吗！”确实，咱们来想一下这个过程，你拿着插头插在了插座上，然后你的电脑和千里之外的供电厂就能“通信”了，把你的电脑想成是客户端，把千里之外的供电厂想成是服务器，通过插座和很长很长的线缆你们就可以勾搭上了，那么Socket在这其中相当于什么呢？“插座！”没错，就是插座！对于我的电脑来说，我想让它通电工作，我只需要个插座就行了啊，什么插座是什么材质的，线缆是什么型号的，供电厂到底在什么经纬度，电力到底怎么传输，我管它干嘛呢，都跟我没关系！我只要知道我需要的不是整个世界，而是。。。一个插座！读到这里，想必同学已经对“插座”有了很森的理解了；再举一个例子，你和基友的电脑通过有线的方式连上了同一个路由器，这个时候你们就可以直接通过内网IP地址进行访问了，在这个过程中，那个方方的接口（RJ45接口）就是“Socket”，反正插上“Socket”就能用，我不用管到底通过Socket怎么能够实现通信。在计算机编程的网络世界里，作为应用程序，我只需要一个“插座”就可以和任何服务器通信了，想想都有点小激动呢~~~

接下来要讲的就是，电脑电源需要一个socket去插上，那么发电厂呢，也同样需要一个插座插上去来给你供电——也就是说，发电厂需要一个“插座”！。。。废话，，，，没错，确实是这样，服务器端也需要一个“插座”，只不过它叫做ServerSocket（这看起来像是继承自Socket，我也不知道，待查）。

有了“插座”（Socket）的概念之后，我们就可以愉快地让电脑（客户端）与发电厂（服务器）通信了。无论是客户端还是服务器，都需要Socket，鉴于咱们今天的题目是“搭建web服务器”，所以咱们接下来就来看一下怎么创建服务器的ServerSocket。说道这里，有同学就会问到了，“难道客户端不需要Socket吗？”，确实需要，因为我们是用浏览器访问本机IP“127.0.0.1”，所以客户端的Socket就由浏览器自己维护了，不需要咱们动手写的。“可是我还是不明白为什么在浏览器里输入127.0.0.1之后就可以看到我的网页了？求解释” 好，那咱们慢慢来，先动手编写一个服务器端的ServerSocket吧啦啦啦~

创建服务器端Socket的步骤如下：

1、创建ServerSocket对象

	ServerSocket serverSocket = new ServerSocket(“80”);  
	//这里只需要指明当前程序监听80号端口就可以了，至于为什么是80，因为我喜欢！“好霸道。。。”因为我们要监听web请求，默认就是80号端口。其实，1-1024端口被操作系统占用了，1025-65535的端口你随便用，只要不会和其他应用程序冲突就可以（别用什么类似3389这么常用的端口就好了。。。）  

2、作为服务器，我要知道，我的使命就是要等待客户端发来请求，也就是客户端发来Socket，我首先要把它Hold住！

	Socket socket = serverSocket.accept(); 
	//这里需要特别说明一下，accept方法比较特殊，它是一个阻塞方法(block method)，因为只要它等不来客户端发来的请求（Socket），它就一直等下去而不会继续执行它下面的代码。唉，此等痴情人怎么跟我一样O(∩_∩)O  

3、客户端要向我表白，给我发来情书，那我作为服务器只要得到它的输入就好了
            
	InputStream inputStream = socket.getInputStream(); 
	//注意，客户端发来的表白信息都在socket里面，而不是serverSocket里面，这点要是弄错了，读不到情书内容，活该你单身。（我只有冷笑。。。）  

4、收到了情书，我好想知道里面究竟写了什么啊！迫！不！及！待！ 好，开始解析情书内容
       
	BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));  
	//java包装类，只为读到写给我的情书，耶~  
	String line = “”;  
	while ( (line = reader.readLine()) != null ){   
       System.out.println(line);   
	}  

5、组装前4步的代码，会要求try catch一下异常，正常捕获就好 下面贴代码

	public class MultiWebServer {   
    public static void main(String[] args) {   
        try {   
            ServerSocket serverSocket = new ServerSocket(80);  
            System.out.println("正在等待情书中...");   
            Socket socket = serverSocket.accept();  
            System.out.println("收到情书，我要开始解析！");   
            InputStream inputStream = socket.getInputStream();   
            BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));  
            String line = "";   
            while ( (line = reader.readLine()) != null ){   
                System.out.println(line);   
            }   
        } catch (Exception e) {   
            e.printStackTrace();   
          }   
    	}   
	}  

好了，服务器端的代码咱们写完了，那接下来干啥？不知道。。。不过还记得刚才提出的问题吗——“可是我还是不明白为什么在浏览器里输入127.0.0.1之后就可以看到我的网页了？”那就试试呗，看看咱们如果在浏览器里输入127.0.0.1或者localhost会怎么样

首先必须把刚才咱们编写的服务器端程序运行起来，然后再打开浏览器，记住，必须先运行服务器端程序，不然情书就发丢了。。。运行服务器端程序，如图：

![](http://img.blog.csdn.net/20141018222024890)

注意红圈中的两点：由于此时没有客户端发来情书，还记得刚才的accept()阻塞方法吗，它就一直等啊等，等不来我还等，所以红圈中会显示“正在等待情书中…”；那么右面那个箭头指向的是什么意思呢，一个红色停止的图标，也就是说，这个程序现在一直在执行着，没有结束，就好像死循环一样（当然这里绝对不是死循环，其实是阻塞，只是死的样子好像死循环，一会咱们会谈到死循环的，别着急，迟早会死的）

接下来打开浏览器，在地址栏输入127.0.0.1/index.html后回车，看看浏览器什么反应。。。。。一段时间过去了，浏览器居然一点反应都没有，然后告诉我该页无法显示。我去。。。难道讲了这么多咱们就这么失败了吗，我哭。那就打开eclipse看一眼吧，看看服务器端有没有什么动静啊。打开服务器端一看，卧槽，瞬间世界向我问好了！

![](http://img.blog.csdn.net/20141018222026250)

注意看红笔标注，我收到了情书！我要开始解析了！那到底情书里是什么内容，别问我，继续向下看。“好熟悉的一段报文，我们好像在哪见过，还记得吗，那是一个春天，你刚发芽儿。。。”没错，这就是计算机网络原理讲的http请求报文。没有学过计网怎么办，没关系，看前两行（其实我们一会用到的也只有第一行而已），“我看到了index.html” 是的，这是刚才我们在浏览器里面输入的地址；第二行，“我也看到了127.0.0.1”，是的，也是我们刚刚在浏览器里面输入的。这说明了什么？激动的我无法说出这到底说明了什么，但想必读者你已经揣测出了什么。
 
写到这里，作为服务器的我已经收到了从客户端发来的情书，那客户端（浏览器）为什么一点反应都没有呢，甚至过了一会就“该页无法显示”了。因为啊，人家给你写了情书，你没回复人家，人家等了一会觉得没戏了就伤心欲绝了！是啊，喜不喜欢人家都要和他说一声的，给他个答复，哪怕只说：“对不起，你是个好人。。。”
走神了吧？好像说到自己了吧？回来吧，咱们现在的任务呢，就是怎么给人家个答复。

怎么给，怎么给，怎么给。。。快想快想，既然人家都指明了想和127.0.0.1里的“index.html”表白，那当然就得由index.html来给他答复喽。怎么答，怎么答，怎么答。。。快想快想，既然index.html是个文件，那我读出文件内容后直接发给客户端不就行了吗？可是用什么发？没错，是socket！我们用socket把文件的内容返回给客户端就好了。

那么问题来了。。。“说的非常好，关键是怎么做！”——首先怎么读出文件来？

假定咱们的index.html在我电脑的E://课件/计算机网络原理/实验/实验1/ 文件夹下，并且假定不会跨域访问，则：

1、定义一个字符串，用来存咱们的工作目录
            
	String base_url = "E://课件/计算机网络原理/实验/实验1/";   
	//这只是我本机的目录，至于到了你的电脑上，你可以自己更改</span>  

2、我怎么通过报文知道客户端要和index.html表白？看情书第一行  GET /index.html HTTP/1.1，所以只需要获取情书的第一行字符串并解析出index.html就好办啦，easy，开始吧

    //由于目前只需要第一行，所以咱们就不像上面那样循环读取了，读一行就够了         
	String line = reader.readLine();  
	//用字符串截取函数，把“index.html”这个字符串给揪出来
	String url = line.substring(5, line.indexOf("HTTP") - 1);    

3、所以咱们index.html的绝对路径就是 base_url + url 了，终于把我爱的人从人山人海中找到了，看看她怎么答复我吧——获取文件内容

	inputStream = new FileInputStream(base_url + url);   
	OutputStream outputStream = socket.getOutputStream();   //我要从服务器给客户端答复了，对于服务器来说，这是发出去的内容，所以是Out！  
	byte[] buffer = new byte[4 * 1024];  //定义字节缓冲区   
	int len = 0;   
	while ((len = inputStream.read(buffer)) != -1) {   
        outputStream.write(buffer, 0, len);  //很重要！通过socket的outputStream把咱们解析出来的文件内容一字不落的发出去 如果没写这个，导致你爱的跟你表白的抑郁而死，活该你单身  
	}   
	outputStream.flush(); //如果最后一次write时没有把buffer写满，是不会自动发出去的，需要调用flush方法强制把内容从缓冲区发出去  

 
好了，文件读取出来了，也返回给客户端了，亲爱哒他能收到吗？还是一样，务必先运行服务器端程序，然后打开浏览器输入127.0.0.1/index.html 后回车。我紧张，我激动，能不能收到回复，会给我什么样的回复？如图。。。

![](http://img.blog.csdn.net/20141018222413101)

为什么会这样？？？！！！ 好吧，看看女神的index.html文件里都写了些什么。。。
	
	<html>   
	    <head>   
	        <meta charset="utf-8" />   
	        <title>Welcome</title>   
	    </head>   
	    <body>      
	        <h1>王欢，你是个好人... </h1>   
	    </body>  
	</html>  

 
看到这里，我到底应该高兴还是欲绝。。。高兴的是，我女神给我答复了；欲绝的是。。。那么问题来了，，，学表白技术哪家强？
 
玩笑归玩笑，那我们的针对这次的浅谈题目是不是就完成了？可以说是的，但是我表白一次失败就算了？我还要表白第二次！（其实我倒不是这样的，这里只能牺牲我的人品来为了大家更好的理解了，呵呵）。好吧，我刚才的工作目录下还有个another.html，这次我来跟她表白吧！好！继续在浏览器中输入127.0.0.1/another.html后回车，期待这次会表白成功。可是我等啊等，浏览器在那里打圈圈，难道浏览器都知道我太花心了，拒绝帮我传递情书？好吧，我再打开浏览器试一下，输入127.0.0.1/index.html ，嗯？连第一个女神都不理我了？！我靠！为毛！

冲动是魔鬼！冷静！我打开eclipse控制台，发现服务器根本就没有“正在等待情书中…”，所以我拜托浏览器发过去的情书当然就发丢了，因为根本没人在接收啊。（窃喜，还好不是因为我太花心了所以浏览器没有帮我投递情书）可是为什么呢？

冷静吧，分析代码。其实我们可以想到，这段代码执行完一次后不就结束了吗，那我第二次给她发请求她当然会收不到了。对啊，那为了解决这个问题，怎么办呢？跪求红娘支招！

红娘说：“给服务器程序个死循环吧，让她反复在等客户端的请求就好了。”（其实红娘就一直在死循环中）

红娘果然是红娘（不然是谁。。。），那就按照她的说法试一试呗！改代码，加入 while (true) 死循环：

	public class MultiWebServer {   
	  
	    public static void main(String[] args) {   
	  
	        String base_url = "E://课件/计算机网络原理/实验/实验1/";   
	  
	        while (true) {   
	            try {   
	                ServerSocket serverSocket = new ServerSocket(80);   
	                System.out.println("正在等待情书中...");   
	                Socket socket = serverSocket.accept();   
	                System.out.println("收到情书，我要开始解析！");   
	                InputStream inputStream = socket.getInputStream();   
	                BufferedReader reader = new BufferedReader(   
	                        new InputStreamReader(inputStream));   
	                String line = reader.readLine();   
	                System.out.println(line);   
	                String url = line.substring(5, line.indexOf("HTTP") - 1);  
	  
	                System.out.println("情书解析完毕，我要想想怎么回复了...");   
	     
	  
	                // 获取文件内容   
	                inputStream = new FileInputStream(base_url + url);   
	                OutputStream outputStream = socket.getOutputStream();   
	                byte[] buffer = new byte[4 * 1024];   
	                int len = 0;   
	                while ((len = inputStream.read(buffer)) != -1) {   
	                    outputStream.write(buffer, 0, len);   
	                }   
	                outputStream.flush();   
	  
	                System.out.println("情书请求已发送给客户端");  
	   
	                //关闭对应的资源   
	                serverSocket.close();   
	                socket.shutdownInput();   
	                socket.close();   
	                inputStream.close();   
	                reader.close();   
	                outputStream.close();   
	            } catch (Exception e) {   
	            }   
	        }   
	    }   
	}
  
这样，这位红娘就在这里一直等啊等，来了一个客户端我就处理他的情书请求，处理完这个继续循环以相同的方式等，处理，等，处理。。。。

好吧，咱们接下来试一下，还是务必先运行服务器端程序，然后先和第一个女神表白，即 127.0.0.1/index.html 还是好朋友的话，就别问我返回结果。。。这个时候打开eclipse的控制台，有没有发现右上角的红色暂停标志可以点击，那就说明咱们的红娘还在兢兢业业的工作着！好了，抓紧时间赶紧向第二个女神表白，看她怎么说， 浏览器输入 127.0.0.1/another.html ，回车！好快啊，女神给我答复了。。。

![](http://img.blog.csdn.net/20141018222413725)

这。。。（她怎么知道不到十分钟？你是不是突然想到了cookie可以记录客户端的信息，不过咱们这里没用到cookie）还是看看another.html文件里写了什么吧

	<html>   
	    <head>   
	        <meta charset="utf-8" />   
	        <title>Welcome</title>   
	    </head>   
	    <body>      
	        <h1>我记得你刚和别人表白吧，还不过十分钟，你怎么会是个好人！</h1>   
	    </body>  
	</html>  

 
好吧，我啥也不说了，亲们，我还要向第三个女神表白吗。。。？浏览器主动跟我说：“你表白吧，这次你发多少封表白信我都能给你送到服务器那里，因为她一直在等待着我给她发送呢！”想想，还是算了，人生如此，何须多言。。。
 
代码都贴出来了，其实看起来挺简单的，但是实际操作中会碰到各种各样的问题。
 
还有一些要再继续唠叨的边角料：

* 1、Q：什么是端口？
* 
A：这是一个比较抽象的概念，是为了进程间通信，每一个进程只能占用一个端口，也就是说多个进程绝不能同时占用一个端口

* 2、Q：既然多个进程不能同时占用一个端口，那么咱们常说的web服务默认使用的是80端口，我电脑有三个浏览器，谷歌，360，IE他们却可以同时上网，这不是端口冲突了吗？
* 
A：常说的web服务使用80端口指的是服务器监听web请求的端口，是服务器，不是你自己的客户机。一般来说，一个应用程序打开后访问网络本地操作系统为其分配的端口号是随机的，所以三个浏览器虽然同时接收web服务器的回复报文，由于他们三个各自占用的端口不一样，所以不会产生冲突。

* 3、Q：既然我的应用程序使用的端口都是随机的，服务器接收到请求后怎么知道它要把应答报文发给谁？
* 
A：靠Socket！通过刚才的编程实战，在我理解，Socket肯定会至少包括四部分内容：IP地址，端口号，输入流和输出流。也就是说，从客户机发给服务器的Socket里一定会有客户机的IP地址和相应应用程序的端口号，这样服务器自然就知道应该把应答报文发给谁了。

* 4、Q：非要使用80端口吗？
* 
A：不一定。我们刚才在编程的时候确实使用的是80端口，所以我们在浏览器中输入127.0.0.1/index.html，浏览器会默认认为我们会向127.0.0.1主机的80号端口发送请求。但是，这个80端口号只是默认的而已，我们完全可以自己改掉，比如在java代码里把服务器端的ServerSocket改成   ServerSocket serverSocket = new ServerSocket(3456);  这时候我们在浏览器中就要输入 127.0.0.1:3456/index.html  了，效果是一样的，可以浅尝辄止一下。

* 5、Q：谁是客户端，谁是服务器？
* 
A：咱们只有一台电脑，这台电脑既充当着客户端的角色，又充当着服务器的角色。当浏览器请求网页时，它是客户端；当80端口收到请求报文并应答时，它就是服务器。实在不理解，就想想什么是自恋吧，或者，自交也勉强可以。。

* 6、Q：还有什么问题，欢迎留言~
 
对于此用java编写的web服务器的一点简单说明：此段代码非常简单，所以肯定不会是真正web服务器所用到的代码，咱们这个只是能够应答最最基本的web请求，不能检测是否跨域访问等等。不过最基本的，用的socket编程是肯定的。另外，对于此段程序，只给出了处理输入输出流的一种方式。对于输入流，除了咱们刚才用到的BufferedReader包装类，还可以直接用InputStream的read()方法等；对于输出流，除了咱们刚才用的OutputStream的write()方法，还可用BufferedWritter，PrintWritter等，这些都是java IO的基本用法，根据网络环境，根据所要读取的文件大小来时时变通，这就是仁者见仁智者见智了。
 
文章写到了最后，不知道该怎么收尾了，安安静静做个程序员吧，挺好。

 
###个人github: [http://github.com/icodeu](http://github.com/icodeu)

###代码托管地址:[https://github.com/icodeu/JavaForWebServer](https://github.com/icodeu/JavaForWebServer)

###CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

###个人微信号：`qqwanghuan`  只为技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)


