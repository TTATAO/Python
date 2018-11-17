# Python
基于Python Django框架
视图部分（views）
from django.shortcuts import render
import JobApp_Django_Project .settings as settings
import uuid
import datetime
import jwt
from django.http import HttpResponse,JsonResponse
import json


# from datetime import datetime
from app import models
from django.core.serializers import serialize
# Create your views here.
SECRECT_KEY = 'com.jobapp'
def index(request):
    return HttpResponse('index')
# 登陆并且签发token
def login(request):

    if request.method=="POST":
        data = json.loads(request.body)
        tel = data['telephone']
        pwd = data['password']
        res=models.UserUseryonghu.objects.filter(telephone=tel).values()
        if len(res):
            res01 = models.UserUseryonghu.objects.filter(telephone=tel, password=pwd).values('id','telephone')[0]
            print(list(res01))
            user = {
                "id":res01['id'],
                "telephone":res01['telephone'],
                "statusCode": ''
            }
            if len(res01):
                user['statusCode'] = "202"
                resp =HttpResponse(json.dumps(user), status=202, charset='utf-8',
                                             content_type='application/json')
                resp['token']=createtoken(tel)
                resp["Access-Control-Expose-Headers"] = "token"
                return resp
            else:
                return HttpResponse('{"statusCode":"404"}')
        else:
            return HttpResponse('{"statusCode":"403"}')


def createtoken(some, aud='webkit'):
    datetimeInt = datetime.datetime.utcnow() + datetime.timedelta(hours=1)
    option = {
        'iss': 'jobapp.com',
        'exp': datetimeInt,
        'aud': aud,
        'some': some
    }
    encoded2 = jwt.encode(option, SECRECT_KEY, algorithm='HS256')
    return encoded2   #字节码


def regist(request):
   import time
   from datetime import datetime
   if request.method=="POST":
       data = json.loads(request.body)
       verification_code = data['v_code']
       tel = data["telephone"]
       pwd = data["password"]
       yonghuming=data["yonghuming"]
       confimpwd=data["confimpwd"]
       regist_time=datetime.now()
       res = models.UserUseryonghu.objects.filter(telephone=tel).values()
       print(len(res))
       if pwd!=confimpwd:
            print(pwd)
            return JsonResponse({"statusCode":"405"})
       elif len(res):

            return JsonResponse({"statusCode":"401"})

       else:
           user = {
               "telephone": tel,
               "password": pwd,
               "confimpwd":confimpwd,
               "pub_time": regist_time,
               "yonghuming":yonghuming
           }
           try:
               result = models.Register.objects.filter(telephone=user['telephone']).values('time', 'message_code').order_by('-time')[0]
               if result['message_code']:
                   message = result['message_code']
                   register_time = result['time']
               else:
                   return JsonResponse({"statuscode": "408"})
               if str(verification_code) == message and int(time.mktime(datetime.now().timetuple())) - int(time.mktime(register_time.timetuple())) < 60:
                    u = models.UserUseryonghu.objects.create(**user)
                    code = {
                        "statusCode": "202"
                    }
                    resp = HttpResponse(json.dumps(code), status=202, charset='utf-8',
                                        content_type='application/json')
                    resp['token'] = createtoken(tel)
                    resp["Access-Control-Expose-Headers"] = "token"
                    return resp

               else:
                    return JsonResponse({"statusCode": "408"})
           except Exception as ex:
               print(ex)
               return JsonResponse({"statusCode": "402"})


def add(request):
  if request.method=='POST':
      # 添加数据，查询添加
      with open("zhishu10.json", "r+", encoding="utf-8")as fp:
          s = json.load(fp)
          for i in s :
              xuan={
                "href":i["trademark"],
                 "name": i["name"],
                  "price":i["price"],
                  "photo":i["photo"],

              }
              user=models.UserUsersousuo.objects.create(**xuan)
              # print(i)
          return HttpResponse(json.dumps(list(s), ensure_ascii=False))


def getuserbyid(request,myid):
    uu=models.UserUserinfo.objects.filter(id=myid).values()
    user=list(uu)[0]
    return JsonResponse(user)

#表连接查询：一对多
def query(request):
   if request.method=='POST':
       ul = models.UserUseraddress.objects.filter(id=1,dizhi__leixing__name='vip5',dizhi__leixing__id=14).values("address", "add_time", "dizhi__password", "dizhi__telephone", "dizhi__leixing__name","dizhi__leixing__id")
       # print(ul)

       u=models.UserUseraddress.objects.filter(id=1).first()
       # print(u.dizhi_id)
       uu=models.UserUserinfo.objects.filter(id=u.dizhi_id).first()
       # print(uu.leixing_id)
       uuu=models.UserType.objects.filter(id=uu.leixing_id).first()
       # print(uuu.name)
       return HttpResponse("yes")



    



def getjobs(request,index,con):
    try:
        pagesize=20
        index=int(index)
        if con:
            jobs = models.UserUserlala.objects.filter(title__regex=con)[pagesize * (index - 1):pagesize * index]. \
                values("job_id", "title", "com_name", "date", "salary")
            # print(jobs)
            return HttpResponse(json.dumps(list(jobs), ensure_ascii=False))
        else:
            jobs=models.UserUserlala.objects.all().values()[pagesize * (index - 1):pagesize * index]. \
                values("job_id", "title", "com_name", "date", "salary")
            return HttpResponse(json.dumps(list(jobs), ensure_ascii=False))

    except Exception as ex:
        print(ex)
        return HttpResponse("no")

def acount(request,con):
    try:
        if not con:
            len=models.UserUserlala.objects.all().count()
        else:
            len=models.UserUserlala.objects.filter(title__regex=con).count()

        return JsonResponse({"acount":len})
    except Exception as ex:
        return JsonResponse({"code":"cuo"})








def getsearch(request,index,con):
    try:
        pagesize=20
        index=int(index)
        if con:
            jobs = models.UserUsersousuo.objects.filter(name__regex=con)[pagesize * (index - 1):pagesize * index]. \
                values('id',"href", "name", "price", "photo")
            # print(jobs)
            return HttpResponse(json.dumps(list(jobs), ensure_ascii=False))
        else:
            jobs=models.UserUsersousuo.objects.all().values()[pagesize * (index - 1):pagesize * index]. \
                values('id',"href", "name", "price", "photo")
            return HttpResponse(json.dumps(list(jobs), ensure_ascii=False))

    except Exception as ex:
        print(ex)
        return HttpResponse("no")

# 分页总数
def acount1(request,con):
    try:
        if not con:
            len=models.UserUsersousuo.objects.all().count()
        else:
            len=models.UserUsersousuo.objects.filter(name__regex=con).count()

        return JsonResponse({"acount1":len})
    except Exception as ex:
        return JsonResponse({"code":"cuo"})


#通过搜索传来的id:con 跳转并获得详情页
def getdeail(request,con):
    try:
        jobs=models.UserUsersousuo.objects.filter(href=con).values("href", "name", "price", "photo")
        # print(jobs)
        return HttpResponse(json.dumps(list(jobs)))

    except Exception as ex:
        print(ex)
        return HttpResponse("no")


# 添加购物车
def Cart(request):
    from datetime import datetime
    if request.method == 'POST':
        user=json.loads(request.body)
        user1=models.UserUsersousuo.objects.filter(href=user["href"]).first()
        user1.telephone=user["telephone"]
       # "pub_time": datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        xuan={
            "telephone":user["telephone"],
            "name":user1.name[0:14],
            "price":user1.price[1:2],
            "pub_time": datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
            "href":user1.href,
            "photo":user1.photo,
            "num":1
        }
        xx=models.UserUsercart.objects.create(**xuan)

        return JsonResponse({"statusCode": "202"})

# 得到数据库里所有购物车信息
def getCart(request):
    user=models.UserUsercart.objects.all().values()
    xuan=list(user)


    return HttpResponse(json.dumps(list(user), ensure_ascii=False))

# 通过详情书的id删除购物车
def deleteCart(request):
   if request.method=='POST':
       user=json.loads(request.body)["href"]
       models.UserUsercart.objects.filter(href=user).delete()
       user=models.UserUsercart.objects.all().values()
       return HttpResponse(json.dumps(list(user), ensure_ascii=False))

# 修改数据库购物车数量
def updataCart(request):
   if request.method=='POST':
       users=json.loads(request.body)
       for user in users:
           # print(user)
           xuan=models.UserUsercart.objects.filter(href=user["href"]).update(num=user["num"])

       return HttpResponse("666")

# 添加订单
def tijiaodingdan(request):
    from datetime import datetime
    good_list = json.loads(request.body)
    print(good_list)
    # print(good_list)
    result = ''
    for good in good_list:
        time_a=  {"pub_time": datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
        models.AppOrder.objects.create(name=good['name'],telephone=good['telephone'],price=good['price'],number=good['number'],href=good['href'],pub_time=time_a["pub_time"],photo=good['photo'],address_id=good['addressid'],status='未付款')
    else:
        result = 'ok'
    if result == "ok":
        models.UserUsercart.objects.all().delete()
        return JsonResponse({"code":202})
    else:
        return JsonResponse({"code": 408})

def pullOrder(request):
   user=models.AppOrder.objects.all().values('id','address__adress','telephone','price','number','href','pub_time','photo','address_id','name',"status")
   # print(user)
   return HttpResponse(json.dumps(list(user),ensure_ascii=False))

# 创建收藏
def Shucang(request):
    from datetime import datetime
    if request.method == 'POST':
        user = json.loads(request.body)
        user1 = models.UserUsersousuo.objects.filter(href=user["href"]).first()
        user1.telephone = user["telephone"]
        # "pub_time": datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        xuan = {
            "telephone": user["telephone"],
            "name": user1.name[0:14],
            "price": user1.price[1:2],
            "pub_time": datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
            "href": user1.href,
            "photo": user1.photo,

        }
        xx = models.AppShoucang.objects.create(**xuan)

        return JsonResponse({"statusCode": "202"})

# 拿到所有收藏
def getShoucang(request):
    user=models.AppShoucang.objects.all().values()
    xuan=list(user)
    return HttpResponse(json.dumps(list(user), ensure_ascii=False))

# 删除收藏
def deleteShoucang(request):
   if request.method=='POST':
       user=json.loads(request.body)["href"]
       user1=json.loads(request.body)["telephone"]
       models.AppShoucang.objects.filter(href=user,telephone=user1).delete()
       user3=models.AppShoucang.objects.all().values()
       return HttpResponse(json.dumps(list(user3), ensure_ascii=False))

# 查看是否收藏
def Searchshoucang(request):
    if request.method=='POST':
       user=json.loads(request.body)["href"]
       user1=json.loads(request.body)["telephone"]
       res = models.AppShoucang.objects.filter(href=user,telephone=user1)
       if len(res)>0:
           return JsonResponse({"statusCode":"202"})
       else:
           return JsonResponse({"statusCode": "402"})

# 取消收藏
def quxiaoshoucang(request):
   if request.method=='POST':
       user=json.loads(request.body)["href"]
       user1=json.loads(request.body)["telephone"]
       models.AppShoucang.objects.filter(href=user,telephone=user1).delete()

       return JsonResponse({"code":"202"})
# 通过用户手机号得到所有地址
def getAllAddressByUserId(request):
    if request.method == "POST":
        user_telephone = json.loads(request.body)["telephone"]
        result = models.AppUseradress.objects.filter(telephone=user_telephone).values()
        return HttpResponse(json.dumps(list(result)))

# 删除订单
def shanchudingdan(request):
    user=json.loads(request.body)["href"]
    models.AppOrder.objects.filter(href=user).delete()

    user = models.AppOrder.objects.all().values('address__adress', 'telephone', 'price', 'number', 'href', 'pub_time', 'photo', 'address_id', 'name',"status")
    # print(user)
    return HttpResponse(json.dumps(list(user), ensure_ascii=False))


def jiesuan(request):
    from datetime import datetime
    if request.method == 'POST':
        user = json.loads(request.body)
        xuan=list(user)
        models.AppOrder.objects.all().delete()
        for i in xuan:
            xuan1={
                "telephone": i["telephone"],
                "name": i["name"],
                "price": i["price"],
                "pub_time": i["pub_time"],
                "href": i["href"],
                "photo": i["photo"],
                "number":i["number"],
                "address_id":i["address_id"],
                "status":i["status"]
            }
            models.AppOrder.objects.create(**xuan1)
        return HttpResponse("666")


def getadress(request):
    user=models.AppJiesaun.objects.all().values()

    return HttpResponse(json.dumps(list(user),ensure_ascii=False))

def querenfukuan(request):
    book_list = json.loads(request.body)
    print(book_list)
    order_list = models.AppOrder.objects.all()
    for book in book_list:
        for order in order_list:
            if book['id'] == order.id:
                models.AppOrder.objects.filter(id=book['id']).update(status='已付款')
    return HttpResponse('ok')

def deltealldingdan(request):
    user=models.AppOrder.objects.all().delete()
    return HttpResponse(json.dumps(list(user),ensure_ascii=False))

# 删除地址
def deladdress(request):
    if request.method == "POST":
        address_id = json.loads(request.body)
        result = models.AppUseradress.objects.filter(id=address_id['address_id']).delete()
        return HttpResponse("202")
    else:
        return HttpResponse("404")
# 添加地址
def addaddress(request):
    if request.method == "POST":
        address_info = json.loads(request.body)
        models.AppUseradress.objects.create(name=address_info['name'],adress=address_info['address'],telephone=address_info['telephone'])
        return HttpResponse("202")
    else:
        return HttpResponse("404")
# 修改密码
def ChangePwd(request):
   if request.method=='POST':
       user = json.loads(request.body)
       telephone=user["telephone"]
       new_pwd=user["new_pwd"]
       old_pwd=user["old_pwd"]
       user=models.UserUseryonghu.objects.filter(telephone=telephone,password=old_pwd).values()
       if user:
           models.UserUseryonghu.objects.filter(telephone=telephone).update(password=new_pwd)

           return JsonResponse({"code":"202"})



# 发送短信接口
def sendMessage(request, user_telephone):
    import random
    import http.client
    from datetime import datetime
    import urllib.parse, hashlib
    if request.method == "GET":
        # 匹配用户
        get_user = models.UserUseryonghu.objects.filter(telephone=user_telephone)
        if len(get_user) == 0:
            # 定义账号和密码，开户之后可以从用户中心得到这两个值
            accountSid = '9fd8913dd5524d03be84ef0c2d3b875b'
            acctKey = '0b409444d653414cb776328606bca801'

            # 定义地址，端口等
            serverHost = "api.miaodiyun.com"
            serverPort = 443
            industryUrl = "/20150822/industrySMS/sendSMS"

            # 生成随机数
            message = random.randrange(1000, 9999)
            # 格式化时间戳，并计算签名
            timeStamp = datetime.strftime(datetime.now(), '%Y%m%d%H%M%S')
            rawsig = accountSid + acctKey + timeStamp
            m = hashlib.md5()
            m.update(str(rawsig).encode('utf-8'))
            sig = m.hexdigest()
            params = urllib.parse.urlencode({'accountSid': accountSid,
                                             'smsContent': "【知书网】您的注册验证码为：{0}，如非本人操作，请忽略此短信。".format(message),
                                             'to': user_telephone,
                                             'timestamp': timeStamp,
                                             'sig': sig})
            # 定义header
            headers = {"Content-Type": "application/x-www-form-urlencoded", "Accept": "application/json"}

            # 与构建https连接
            conn = http.client.HTTPSConnection(serverHost, serverPort)

            # Post数据
            conn.request(method="POST", url=industryUrl, body=params, headers=headers)
            # 返回处理后的数据
            response = conn.getresponse()
            models.Register.objects.create(telephone=user_telephone, message_code=message, time=datetime.now())
            # 读取返回数据
            jsondata = response.read().decode('utf-8')
            # 解析json，获取特定的几个字段
            jsonObj = json.loads(jsondata)
            respCode = jsonObj['respCode']
            print("错误码:", respCode)
            respDesc = jsonObj['respDesc']
            print("错误描述:", respDesc)
            # 关闭连接
            conn.close()

            return JsonResponse({'statuscode': "202"})
        else:
            return JsonResponse({'statuscode': "408"})
    else:
        return JsonResponse({'statuscode': "409"})

def getCommentByArticleId(request, articleid, userid):
    if request.method == 'GET':
        try:
            result = models.Comment.objects.filter(article_id=articleid).values('id', 'content', 'user_id', 'time', 'user__user_name',
                                                                                'user__img').order_by('-time')
            final_result = []
            for re in result:
                # 全是数据的格式化
                sixsixsix = models.Sixsixsix.objects.filter(comment_id=re["id"]).count()
                comment = {"comment_id": re["id"], "comment_content": re["content"], "user_id": re["user_id"],
                           "time": re['time'].strftime("%Y-%m-%d %H:%M:%S"), "user_name": re["user__user_name"], "user_img": re["user__img"],
                           "likes": sixsixsix}
                replys = models.Reply.objects.filter(comment_id=comment['comment_id']).values('user_id', 'content', 'time', 'user__user_name',
                                                                                              'user__img').order_by('-time')
                new_replys = []
                for rep in replys:
                    reply = {"user_id": rep["user_id"], "content": rep["content"], "time": rep["time"].strftime("%Y-%m-%d %H:%M:%S"),
                             "user_name": rep["user__user_name"], "user_img": rep["user__img"]}
                    new_replys.append(reply)
                comment['replys'] = new_replys
                if userid:
                    likes = models.Sixsixsix.objects.filter(user_id=userid, comment_id=re['id']).count()
                    if likes > 0:
                        comment['islike'] = False
                    else:
                        comment['islike'] = True
                else:
                    comment['islike'] = True  # 是否点赞
                    comment['showreply'] = True  # 是否显示回复
                    comment['btnvalue'] = "收起回复"  # 收起/显示回复按钮的value
                    comment['showreplyinput'] = False  # 是否显示回复框
                    comment['showreplysize'] = 3  # 每次加载回复数量
                final_result.append(comment)
            return HttpResponse(json.dumps(final_result, ensure_ascii=False))
        except Exception as err:
            print(err)
            return JsonResponse({"statuscode": "409"})
    else:
        return JsonResponse({"statuscode": "409"})

#
# # 回复评论
# def replyComment(request):
#     if request.method == 'POST':
#         userid = json.loads(request.body)['user_id']
#         articleid = json.loads(request.body)['article_id']
#         replycontent = json.loads(request.body)['content']
#         commentid = json.loads(request.body)['comment_id']
#         getRequestToken = request.META.get('HTTP_TOKEN')
#         # 如果请求头中含有token
#         if getRequestToken:
#             # 解析到的token
#             token = getToken(getRequestToken)
#         else:
#             return JsonResponse({"statuscode": "408"})
#         # 如果解析出的id和请求的用户id一样
#         if userid and token['user_id'] == int(userid):
#             try:
#                 result = models.Article.objects.filter(id = articleid)
#                 if len(result) == 1:
#                     update_result = models.Reply.objects.create(user_id=userid,content=replycontent,comment_id=commentid,time=datetime.now())
#                     if update_result:
#                         return JsonResponse({"statuscode": "202"})
#                     else:
#                         return JsonResponse({"statuscode": "409"})
#             except Exception as err:
#                 print(err)
#                 return JsonResponse({"statuscode": "409"})
#         else:
#             return JsonResponse({"statuscode": "409"})
#     else:
#         return JsonResponse({"statuscode": "409"})
#
#
# # 文章评论
# def commentArticle(request):
#     if request.method == 'POST':
#         userid = json.loads(request.body)['user_id']
#         articleid = json.loads(request.body)['article_id']
#         commentcontent = json.loads(request.body)['content']
#         getRequestToken = request.META.get('HTTP_TOKEN')
#         # 如果请求头中含有token
#         if getRequestToken:
#             # 解析到的token
#             token = getToken(getRequestToken)
#         else:
#             return JsonResponse({"statuscode": "408"})
#         # 如果解析出的id和请求的用户id一样
#         if userid and token['user_id'] == int(userid):
#             try:
#                 result = models.Article.objects.filter(id = articleid)
#                 if len(result) == 1:
#                     update_result = models.Comment.objects.create(article_id=articleid,user_id=userid,content=commentcontent,time=datetime.now())
#                     if update_result:
#                         return JsonResponse({"statuscode": "202"})
#                     else:
#                         return JsonResponse({"statuscode": "409"})
#             except Exception as err:
#                 print(err)
#                 return JsonResponse({"statuscode": "409"})
#         else:
#             return JsonResponse({"statuscode": "409"})
#     else:
#         return JsonResponse({"statuscode": "409"})


# 上传用户头像
def upLoadIcon(request):
    if request.method == 'POST':
        try:
            # 此处可以接收文件和字符串
            user_icon = request.FILES['user_icon']
            user_id = int(request.POST.get('usericonid'))
            print(user_id)
            # 设置保存的文件名
            fname = str(settings.MEDIA_ROOT[0]) + '/icon/' + str(uuid.uuid4()) + '.' + user_icon.name.split('.')[1]
            # 由于文件是二进制流的方式，所有要用chunks()
            with open(fname, 'wb+') as pic:
                for c in user_icon.chunks():
                    pic.write(c)
            models.UserUseryonghu.objects.filter(id=user_id).update(img='http://127.0.0.1:8000/{0}'.format(fname.split('Project\\')[1]))
            return JsonResponse({"code": "202"})
        except Exception as ex:
            print(ex)
            return JsonResponse({"code": "408"})
    else:
        return JsonResponse({"code": "408"})


# 将用户名拿到发送给个人资料
def getyonghumings(request):
    telephone=json.loads(request.body)["telephone"]
    user=models.UserUseryonghu.objects.filter(telephone=telephone).values("yonghuming",'img')
    print(user)

    return HttpResponse(json.dumps(list(user),ensure_ascii=False))


# 修改用户信息
def updata_user(request):
    if request.method=='POST':
        telephone=json.loads(request.body)["telephone"]
        yonghuming=json.loads(request.body)["yonghuming"]
        user=models.UserUseryonghu.objects.filter(telephone=telephone).update(yonghuming=yonghuming)

        return JsonResponse({"code":"666"})

#这个接口是我自己用来测试用的
def found(request):
    if request.method=='POST':
        href=json.loads(request.body)["href"]
        print(href)
        user=models.UserUsersousuo.objects.filter(href=href).values()
        print(user)        
        return HttpResponse("successful")





