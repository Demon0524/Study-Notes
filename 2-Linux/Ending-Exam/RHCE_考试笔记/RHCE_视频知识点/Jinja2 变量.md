#Linux #Ending_Exam #RHCE #视频笔记
## 第六章---在被创建的
双大括号--调用变量
setup---查询变量信息
* 查询与 host 有关的变量
`setup -a -fittle “* host *” `
src---表示路劲下的模板拷贝到目标目录
1.  jinja---模板文件
*  定义---通过不同主机已有的变量生成相应的文件


* 使用 for 语句逐一运行 users 变量中的所有值，将 myuser 替换为各个值
```

```
* 变量过滤器
	* 
* 变量测试
* jinja 模板(python)
```json
	from jinjia2 import Template,Environment,FileSystemLoader
	dict_item=[
	{'name':'tom','age':20},
	{'name':'jerry','age':18},
	{'name':'andy','age':19},
	]
	s="{% for key,value in dict_item.items() %}"\ /*表示 FOR 循环 */
		key1:{{key}} \ 
		value:{{value}} \
		{% endfor %}"
	template=Template(s)
	print(template)
	//定义了一个新的字典
	dict={'A:'tom',B:'jerry''}
	items=template.render(dict_item=dict)
	print(items)
```
* ansible---
```yml
---
-	hosts:webone
	vars:
		my_name:tom
	tasks:
		-	name:print message
			debug:
				msg:
				-	"This is my name {{my_name}}"
				-	"This is my name {{my_name | lower}}"//使用 jinja 过滤器
				-	"This is my name {{my_name | upper}}"
				-	"This is my name {{my_name | capitalize}}"
				-	"This is my name {{my_name | title}}"
				-	"This is my name {{my_name}}{{ real_name | default//默认创建 ('Andy')}}"
```

vim jinja2_for.yml
```yml
---
-	hosts:webone
	vars:
		listvars:['data1','data2','data3',4]
	tasks:
		-	name:print message
			debug:
				msg: |
					{%for item in listvars: -%}
						{{ item }}
					{% endfor %}
				
```

vim jinja2_for_if.yml
```yml
---
-	hosts:webone
	vars:
		persons:
			user1:
				name:tom
				age:18
			user2:
				name:jerRy
				age:20
	tasks:
		-	name:print message1
			debug:
				msg: |
					{%for item in persons: -%}
						{{ item }}
					{% endfor %}
		-	name:print message2
			debug:
				msg: |
					{%for item in persons: -%}
						{% if itemvalue.age | int > 20 %}
						my name id {{ itemvalue.name }}, my age is {{ itemvalue.age}}
						{% endif %}
					{% endfor %}
					
		-	name:print message3
			debug:
				msg:
					"{{ ansible_flect['fqdn']}}"
```
