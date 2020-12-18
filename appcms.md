## 0x01 SQL injection vulnerability exists in /dawn/download_frame.php(Login required)

 First, the loopholes should be reappeared, and then the reasons should be analyzed :



 After logging in the background ,We know that if we need to add an app, we need a key:

![image-20201214232542148](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201214232542148.png)

![image-20201214232820225](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201214232820225.png)

 So before testing, I need to create a new table in the database and add data .

![image-20201214232954211](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201214232954211.png)

The table name is "app"_ cms_list" , It then contains two pieces of data, as shown in the figure.

 Next, we can visit this link to perform blind SQL injection in the "now":("dawn" is the original "admin", but the system needs us to change the background name)

http://www.dmsj.com:8081/dawn/download_frame.php?m=list&s=1&end=10&now=1+and+1=1%23

Pay attention to the use of "+" instead of "space", and unsuccessful words will lead to 302. At the same time, remember to log in to the background.

![image-20201215085330757](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215085330757.png)

![image-20201215085532770](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215085532770.png)



![image-20201215085559722](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215085559722.png)

 Little surprise, we also found that its cookie did not change before and after login, but it was in the header, interesting. 



 Next, we analyze the code :

dawn\download_frame.php 

We can control them completely,Then go to "get_ List":

Our "$now" was handed over to "$params['where']"

![image-20201215091425014](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215091425014.png)

core\common.class.php

![image-20201215091541201](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215091541201.png)

 In any case, our $where is not filtered and goes directly into the SQL statement :

![image-20201215091710976](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215091710976.png)

 We follow the "query": 

![image-20201215091827656](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215091827656.png)

core\database.class.php

![image-20201215091925659](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215091925659.png)

nice!



## 0x02 I found out in /dawn/app.php After logging in, allow me to delete any file(Login required)

 First, we find that there is a sensitive function for "del_resource" of core\function.php

![image-20201215095032367](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095032367.png)

 We follow it to dawn\app.php:

![image-20201215095145031](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095145031.png)

![image-20201215095237149](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095237149.png)

How do we call the "m__del_resource" function?

![image-20201215095323046](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095323046.png)

 Good. I already know how to use it :

Let's first create a test file 123.php in the root directory:

![image-20201215095518042](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095518042.png)

Open burp and pay attention to the data of get and post:(Note that you have to log in to the background first:

http://your_site/your_backstage)

![image-20201215095545792](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095545792.png)

 Click send, it shows failed? 

![image-20201215095842642](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095842642.png)

 But when we look at the local files, "123.php" has disappeared :

![image-20201215095855339](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215095855339.png)



## 0x03 I found out in /dawn/info.php After logging in, allow me to delete any file(Login required)

 In the same way, we found another point :

dawn\info.php

![image-20201215100653075](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215100653075.png)

![image-20201215100850989](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215100850989.png)

## 0x04 I found that at /dawn/template_wap.php, I was allowed to log in and write arbitrary files(Login required)

 Of course, you have to log in to the background first .

 At "dawn\template_wap.php", there is a sensitive function to write to the file:

![image-20201215103337144](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215103337144.png)

We can control the filename and content, We have nothing to do with this "escape_stripslashes".

The method to call this function is the same as above. We construct the data package:

![image-20201215104525647](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215104525647.png)

![image-20201215104213752](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215104213752.png)



![image-20201215104602716](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215104602716.png)





## 0x05 I found that at /dawn/template.php, I was allowed to log in and write arbitrary files(Login required)

 In the same way, we found another point :

![image-20201215104748937](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215104748937.png)

![image-20201215104814363](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215104814363.png)

![image-20201215104840398](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201215104840398.png)



## 0x06  I found Reflective XSS in tpl_app.php/tpl_info.php..........(Login required)

Of course, we have to log in to the background first.

 Let's go look at the code, it's very easy :   /dawn/templates/tpl_app.php

![image-20201218135417851](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201218135417851.png)

payload:

```
/dawn/templates/tpl_info.php?cate_id=ddddhm); });</script><script>alert(1);</script>
```

Of course, there are many files with the same problem.





