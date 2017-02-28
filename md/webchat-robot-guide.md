# WebChat自动应答接口说明 #
9/22/2016 2:58:08 PM  
### 通过知识库维护机器人自动应答的相关文章后（知识库的文章需要开启自动应答，才回被机器人检索到: forRobot = 1），可以通过http接口来获取机器人自动应答的信息 ###

接口采用http协议，post方式，请求参数是form格式，返回结果为json格式

- 查询问题： 
> 	http://xxxxx/EliteKM/RobotServlet
> 	发送参数：
		action: "search",
		question: "测试"
> 	返回参数：
> 	{
> 	  "question":"测试",
> 	  "srs":[ //返回一个搜索结果的数组
> 	    {
> 	     "preview":"<span style=\"color:#FFFFF0;background-color:#1E90FF;font-weight:bold;\" id=\"kmhighlighter\">测试<\/span>", //结果预览
> 	     "question":"",
> 	     "ra":{ //文章相关信息，包括id，路径，标题
> 	       "articleAId":"3baf0f47-f232-1737-7610-e65f77b2d6d0",
> 	       "path":"/oldkm/ArticleStore/E9D9C25E_1741_48C9_915E_2B6EE7FA6385/3baf0f47-f232-1737-7610-e65f77b2d6d0.htm",
> 	       "title":"klkljhk"
> 	     },
> 	     "title":"klkljhk" //文章标题
> 	     }
> 	   ]
> 	}

