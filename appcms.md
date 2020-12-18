## 0x01 SQL injection vulnerability exists in /dawn/download_frame.php(Login required)

 First, the loopholes should be reappeared, and then the reasons should be analyzed :



 After logging in the background ,We know that if we need to add an app, we need a key:

![1](https://user-images.githubusercontent.com/55305492/102581125-e329ad00-413a-11eb-87c4-8c4327daa808.png)

![2](https://user-images.githubusercontent.com/55305492/102581148-ed4bab80-413a-11eb-9c40-0138747c45fe.png)

 So before testing, I need to create a new table in the database and add data .

![3](https://user-images.githubusercontent.com/55305492/102581170-f8064080-413a-11eb-9272-5613019d6e5e.png)

The table name is "app"_ cms_list" , It then contains two pieces of data, as shown in the figure.

 Next, we can visit this link to perform blind SQL injection in the "now":("dawn" is the original "admin", but the system needs us to change the background name)

http://www.dmsj.com:8081/dawn/download_frame.php?m=list&s=1&end=10&now=1+and+1=1%23

Pay attention to the use of "+" instead of "space", and unsuccessful words will lead to 302. At the same time, remember to log in to the background.

![4](https://user-images.githubusercontent.com/55305492/102581188-ffc5e500-413a-11eb-9ea8-771838b6ee2b.png)

![5](https://user-images.githubusercontent.com/55305492/102581195-05232f80-413b-11eb-8f64-f185be370a74.png)



![6](https://user-images.githubusercontent.com/55305492/102581208-0a807a00-413b-11eb-9efc-19c9c40c73bb.png)

 Little surprise, we also found that its cookie did not change before and after login, but it was in the header, interesting. 



 Next, we analyze the code :

dawn\download_frame.php 

We can control them completely,Then go to "get_ List":

Our "$now" was handed over to "$params['where']"

![7](https://user-images.githubusercontent.com/55305492/102581223-11a78800-413b-11eb-929c-f1ccba64af8a.png)

core\common.class.php

![8](https://user-images.githubusercontent.com/55305492/102581231-18ce9600-413b-11eb-8493-ef5e40ed0b94.png)

 In any case, our $where is not filtered and goes directly into the SQL statement :

![9](https://user-images.githubusercontent.com/55305492/102581245-1d934a00-413b-11eb-95f1-c7a0a2b86a4a.png)

 We follow the "query": 
 
![10](https://user-images.githubusercontent.com/55305492/102581254-2257fe00-413b-11eb-8c3a-142a372ea6ff.png)

core\database.class.php

![11](https://user-images.githubusercontent.com/55305492/102581259-25eb8500-413b-11eb-86d1-9cd06bf447dd.png)

nice!



## 0x02 I found out in /dawn/app.php After logging in, allow me to delete any file(Login required)

 First, we find that there is a sensitive function for "del_resource" of core\function.php

![1](https://user-images.githubusercontent.com/55305492/102581510-ae6a2580-413b-11eb-8d0f-54d642e6e09d.png)

 We follow it to dawn\app.php:

![2](https://user-images.githubusercontent.com/55305492/102581519-b4600680-413b-11eb-9cf4-257f71aa2692.png)

![3](https://user-images.githubusercontent.com/55305492/102581525-b924ba80-413b-11eb-85d6-cd97126158e3.png)

How do we call the "m__del_resource" function?

![4](https://user-images.githubusercontent.com/55305492/102581538-bfb33200-413b-11eb-92b3-8e3837f6668f.png)

 Good. I already know how to use it :

Let's first create a test file 123.php in the root directory:

![5](https://user-images.githubusercontent.com/55305492/102581547-c346b900-413b-11eb-9c08-9e6f73f13493.png)

Open burp and pay attention to the data of get and post:(Note that you have to log in to the background first:

http://your_site/your_backstage)

![6](https://user-images.githubusercontent.com/55305492/102581560-c8a40380-413b-11eb-892c-ab6786d5c422.png)

 Click send, it shows failed? 

![7](https://user-images.githubusercontent.com/55305492/102581574-cf327b00-413b-11eb-9202-5e930e3b82ea.png)

 But when we look at the local files, "123.php" has disappeared :

![8](https://user-images.githubusercontent.com/55305492/102581586-d35e9880-413b-11eb-9ad0-21571bc7f8a0.png)



## 0x03 I found out in /dawn/info.php After logging in, allow me to delete any file(Login required)

 In the same way, we found another point :

dawn\info.php

![1](https://user-images.githubusercontent.com/55305492/102581864-5ed82980-413c-11eb-9359-eb166ac894bb.png)

![2](https://user-images.githubusercontent.com/55305492/102581873-63044700-413c-11eb-990f-516ca677196b.png)

## 0x04 I found that at /dawn/template_wap.php, I was allowed to log in and write arbitrary files(Login required)

 Of course, you have to log in to the background first .

 At "dawn\template_wap.php", there is a sensitive function to write to the file:

![1](https://user-images.githubusercontent.com/55305492/102582187-02c1d500-413d-11eb-934c-ad34d600c922.png)

We can control the filename and content, We have nothing to do with this "escape_stripslashes".

The method to call this function is the same as above. We construct the data package:

![2](https://user-images.githubusercontent.com/55305492/102582193-06555c00-413d-11eb-8e2f-84f689edd356.png)

![3](https://user-images.githubusercontent.com/55305492/102582200-0c4b3d00-413d-11eb-8c24-cd37670990e6.png)



![4](https://user-images.githubusercontent.com/55305492/102582217-153c0e80-413d-11eb-8ef1-68cb40fbf2f4.png)





## 0x05 I found that at /dawn/template.php, I was allowed to log in and write arbitrary files(Login required)

 In the same way, we found another point :

![1](https://user-images.githubusercontent.com/55305492/102582302-43b9e980-413d-11eb-9884-90685c3c49b6.png)

![2](https://user-images.githubusercontent.com/55305492/102582308-474d7080-413d-11eb-96cd-fb3f48783081.png)

![3](https://user-images.githubusercontent.com/55305492/102582312-4ae0f780-413d-11eb-8160-d495c6f295c8.png)



## 0x06  I found Reflective XSS in tpl_app.php/tpl_info.php..........(Login required)

Of course, we have to log in to the background first.

 Let's go look at the code, it's very easy :   /dawn/templates/tpl_app.php

![1](https://user-images.githubusercontent.com/55305492/102582407-7bc12c80-413d-11eb-8b70-1b0df2c33d26.png)

payload:

```
/dawn/templates/tpl_info.php?cate_id=ddddhm); });</script><script>alert(1);</script>
```

Of course, there are many files with the same problem.

![2](https://user-images.githubusercontent.com/55305492/102582413-811e7700-413d-11eb-9aa5-858bb0f7f7f7.png)



