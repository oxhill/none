# None
## 模块导入
```
from flask import Flask,request,json, jsonify,  current_app
import time
from flask_pymongo import PyMongo
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flask_sqlalchemy import SQLAlchemy
import flask_whooshalchemyplus
from jieba.analyse.analyzer import ChineseAnalyzer
app=Flask(__name__)
tasks=list()
app=Flask(__name__)
tasks=list()

``` 
## 用户功能（使用 token 实现）

```
@app.route('/find', methods=['GET'])
def generate_token(api_users):
    expiration = 3600
    s = Serializer(current_app.config['SECRET_KEY'], expires_in=expiration) #expiration是过期时间
    token = s.dumps({'id': api_users.id}).decode('ascii')
    return token, expiration
def find():

    user = request.args.get('user') #利用这个request.args.get方法可以模拟postman输入参数
    password = request.args.get('password')
    box_id = request.args.get('box_id')
    user = User.query.filter_by(user=user, password=password).first()
    result = mongo.db[box_id].find().sort([("t", -1)]).limit(1)
    list = []
    for res in result:
        list.append(str(res))
    token = generate_token(user)
    str_token = str(token)
    list.append(str_token)
    json_data = jsonify(list)
    return json_data

@app.route('/token', methods=['GET', 'POST'])
def token():
    token = request.args.get('token')
    _token = verify_auth_token(token)
    # print(_token)
    user = User.query.filter_by(id=_token.id).first()
    result = mongo.db[user.box_id].find().sort([("t", -1)]).limit(1)
    list = []
    for res in result:
        list.append(str(res))
    json_data = jsonify(list)
    return json_data
```
## 事项相关的设置（定义事项属性，对事项实现一些功能）
### 一条事项
```
class TASK():
	def __init__(self,content):#定义一条事项的属性
		self.id=TASK.max_id+1
		self.content=content
		self.status=False
		self.start_time=time.time()
		self.end_time=None
		TASK.max_id+=1
	def fanhui(self):#使用josn格式返回一条事项的数据
		return{
			"id":self.id,
			"content":self.content,
			"status":int(self.status),
			"start_time":self.start_time,
			"end_time":self.end_time
		}
	@classmethod#classmethod 修饰符对应的函数不需要实例化，不需要 self 参数，但第一个参数需要是表示自身类的 cls 参数，可以来调用类的属性，类的方法，实例化对象等。
	def fanhui_list(cls,tasks):
		return[task.fanhui()for task in tasks]#返回所有事项的数据
def make_resp(data,status=200,message="success"):#200和success是默认参数
	return {
		"status":status,
		"message":message,
		"data":data
	}
@app.route('/tasks',methods=['GET','POST'])#(可以用put吗)
def task_list():
	if request.method=="GET":
		data=TASK.fanhui_list(tasks)#查看所有事项
		response1=make_resp(data)

		result3=response1.get('data')
		print(result3)
		return response1
	if request.method=="POST":
		content=request.form['content']#获取以POST方式提交的数据
		new_task=TASK(content)
		tasks.append(new_task)#添加事项
		data=new_task.fanhui()
		response1=make_resp(data)
		return response1
@app.route("/task/<int:id>",methods=["GET","POST","DELETE"])
def task(id):
	if request.method=="GET":
		return make_resp(task[id].fanhui())#查看某一条事项
	if request.method=="POST":
		tasks[id].status=not tasks[id].status#将某一条事项设置为已完成
	if request.method=="DELETE":
		tasks.remove(tasks[id])#删除某一条事项
		return make_resp({})
```
### 所有事项
```
@app.route("/all_task",methods=["GET","POST","DELETE"])
def all_ask():
	if request.method=="GET":
		data=TASK.fanhui_list(tasks)#查看所有事项
		response1=make_resp(data)
		return response1
	if request.method=="POST":
		for id in range(0,len(tasks)):
			tasks[id].status=not tasks[id].status#将所有事项设置为已完成
		
	if request.method=="DELETE":
		tasks.remove(tasks)#删除所有事项
		return make_resp({})
```
## 查询
   
```
 #初始化搜索
flask_whooshalchemyplus.init_app(app)
@api.route('/search',methods=["GET","POST"])
def search():
    if request.method == "POST":
        q = str(request.form.get('tasks'))
        result = all_task.query.whoosh_search(q).all()
        return render_template('search_back.html',results=result)

    flask_whooshalchemyplus.index_one_model(all_task)

    return render_template('search.html')
    （还没写好www）
if __name__=='__main__':

	app.run(debug=True)
```
