## 欢迎来到狗蛋的主页

我即将介绍一个极具生产力的工具，也是由我设计及编写的。虽然目前并不完善，但是其想法非常值得分享。

### easytestapi

easytestapi是我给这个工具起的名字，意思是简单测试api。是一款接口测试工具。
你可能不觉得有什么稀奇，毕竟市面上的接口测试工具可太多了，例如：postman、jmeter、apifox、soapUI等等。这些工具都有其特色及定位，也非常的优秀。
但是从我的实际工作中这些工具也仅仅作为调试工具来使用，虽然有些也有团队协作、性能或者自动化之类的功能。但是在我看来还不够，还是麻烦。
简单来说，我编辑接口后，还需要我额外一点点的添加额外的东西来满足自动化测试，在我看来还不够自动化。

### 为什么能实现高度自动化

http接口的结构可以说是相当清楚统一了，既然如此，我们为何不能定义好这种统一的数据结构来进行高度自动化呢？

### 统一定义接口的数据结构

下面是一个简单的例子，登录接口定义
```json
{
    "desc": "登录接口",
    "code": "tp_login_api",
    "versions": ["v1.0", "v2.0"],
    "route": "/user/login",
    "method": "POST",
    "setup": [
    ],
    "headers": {
        "Content-Type": "application/json"
    },
    "fields": [
        {
            "name": "account",
            "data_type": {
                "default": "str"
            },
            "length": {
                "default": 30
            },
            "required": {
                "default": true,
                "case": {
                    "value": "eval:notfill",
                    "flag": false,
                    "expect_result": {
                        "$.code": "40001",
                        "$.msg": "账号必填"
                    },
                    "desc": "字段不填"
                }
            },
            "nullable": {
                "default": false,
                "case": {
                    "value": null,
                    "flag": false,
                    "expect_result": {
                        "$.code": "40001",
                        "$.msg": "账号必填"
                    },
                    "desc": "字段填null"
                }
            },
            "location": 1,
            "prefetch": {
                "default": {
                    "value": "eval:consts['account']",
                    "desc": "默认账户"
                },
                "case": {
                    "value": "eval:consts['account']",
                    "desc": "默认账户"
                }
            },
            "desc": "账户"
        },
        {
            "name": "password",
            "data_type": "str",
            "length": {
                "default": 30
            },
            "require": {
                "default": true,
                "case": {
                    "value": "eval:notfill",
                    "flag": false,
                    "expect_result": {
                        "$.code": "40002",
                        "$.msg": "密码必填"
                    },
                    "desc": "字段不填"
                }
            },
            "nullable": {
                "default": null,
                "case": {
                    "value": null,
                    "flag": false,
                    "expect_result": {
                        "$.code": "40002",
                        "$.msg": "密码必填"
                    },
                    "desc": "字段填null"
                }
            },
            "location": 1,
            "prefetch": {
                "default": {
                    "value": "eval:consts['password']",
                    "desc": "默认密码"
                },
                "case": {
                    "value": "eval:consts['password']",
                    "desc": "默认密码"
                }
            },
            "desc": "密码"
        }
    ],
    "teardown": [],
    "additional": [],
    "response": {
        "$.code": "200",
        "$.data.token": "\\w+"
    }
}
```

### 生成用例
![image](https://user-images.githubusercontent.com/25731134/168555481-f33ede0e-32cf-499e-97f4-cf578745fcc8.png)
- 正向的登录接口用例

```json
{
    "id": "1",
    "flag": true,
    "desc": "登录接口",
    "versions": [
        "v1.0",
        "v2.0"
    ],
    "route": "/user/login",
    "route_params": {},
    "params": {},
    "method": "POST",
    "setup": [],
    "headers": {
        "Content-Type": "application/json"
    },
    "body": {
        "account": "eval:consts['account']",
        "password": "eval:consts['password']"
    },
    "teardown": [],
    "response": {
        "$.code": "200",
        "$.data.token": "\\w+"
    }
}
```
- 反向的登录接口用例(取一个展示)

```json
{
    "id": "2",
    "flag": false,
    "desc": "登录接口_account参数不填",
    "versions": [
        "v1.0",
        "v2.0"
    ],
    "route": "/user/login",
    "route_params": {},
    "params": {},
    "method": "POST",
    "setup": [],
    "headers": {
        "Content-Type": "application/json"
    },
    "body": {
        "password": "eval:consts['password']"
    },
    "teardown": [],
    "response": {
        "$.code": "40001",
        "$.msg": "账号必填"
    }
}
```
### 执行用例
下面为执行日志展示
```
2022-05-16 16:55:15 root[line:57] INFO [Eval]: admin
2022-05-16 16:55:15 root[line:57] INFO [Eval]: 123456
2022-05-16 16:55:15 urllib3.connectionpool[line:228] DEBUG Starting new HTTP connection (1): 127.0.0.1:8077
2022-05-16 16:55:15 urllib3.connectionpool[line:456] DEBUG http://127.0.0.1:8077 "POST http://192.168.255.10:5000/user/login HTTP/1.1" 200 801
2022-05-16 16:55:16 root[line:471] DEBUG [Headers]: {'Content-Type': 'application/json'}
2022-05-16 16:55:16 root[line:472] DEBUG [Request-Entries]: {} {} {'account': 'admin', 'password': '123456'}
2022-05-16 16:55:16 root[line:473] DEBUG [Response]: {'code': 200, 'data': {'current_role_id': 2, 'user_info': {'id': 2, 'username': '管理员', 'user_code': 'admin', 'email': 'admin@qq.com', 'phone': None, 'create_time': '2022-01-21 18:25:30', 'update_time': '2022-01-21 18:25:30', 'locked': '1', 'locked_time': None, 'unlocked_time': None, 'remark': None}, 'user_roles': {'id': 2, 'roles': [{'id': 2, 'role_name': '普通管理员角色', 'role_level': 2, 'introduction': '企业用户管理员角色使用', 'create_user_id': 1}, {'id': 3, 'role_name': '普通用户角色', 'role_level': 3, 'introduction': '企业下普通用户使用', 'create_user_id': 1}]}, 'token': '2-4061d0a37a4647ff50db3e6cd28a4030f98b9e6a50c52ded3ea59128b864092a'}}
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.code==200 200
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.data.token==\w+ 2-4061d0a37a4647ff50db3e6cd28a4030f98b9e6a50c52ded3ea59128b864092a
2022-05-16 16:55:16 root[line:486] INFO [AssertRight] ！o(￣▽￣)ｄ！
2022-05-16 16:55:16 root[line:57] INFO [Eval]: 123456
2022-05-16 16:55:16 urllib3.connectionpool[line:228] DEBUG Starting new HTTP connection (1): 127.0.0.1:8077
2022-05-16 16:55:16 urllib3.connectionpool[line:456] DEBUG http://127.0.0.1:8077 "POST http://192.168.255.10:5000/user/login HTTP/1.1" 200 50
2022-05-16 16:55:16 root[line:471] DEBUG [Headers]: {'Content-Type': 'application/json'}
2022-05-16 16:55:16 root[line:472] DEBUG [Request-Entries]: {} {} {'password': '123456'}
2022-05-16 16:55:16 root[line:473] DEBUG [Response]: {'code': 40001, 'msg': '账号必填'}
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.code==40001 40001
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.msg==账号必填 账号必填
2022-05-16 16:55:16 root[line:486] INFO [AssertRight] ！o(￣▽￣)ｄ！
2022-05-16 16:55:16 root[line:57] INFO [Eval]: 123456
2022-05-16 16:55:16 urllib3.connectionpool[line:228] DEBUG Starting new HTTP connection (1): 127.0.0.1:8077
2022-05-16 16:55:16 urllib3.connectionpool[line:456] DEBUG http://127.0.0.1:8077 "POST http://192.168.255.10:5000/user/login HTTP/1.1" 200 50
2022-05-16 16:55:16 root[line:471] DEBUG [Headers]: {'Content-Type': 'application/json'}
2022-05-16 16:55:16 root[line:472] DEBUG [Request-Entries]: {} {} {'account': None, 'password': '123456'}
2022-05-16 16:55:16 root[line:473] DEBUG [Response]: {'code': 40001, 'msg': '账号必填'}
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.code==40001 40001
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.msg==账号必填 账号必填
2022-05-16 16:55:16 root[line:486] INFO [AssertRight] ！o(￣▽￣)ｄ！
2022-05-16 16:55:16 root[line:57] INFO [Eval]: 123456
2022-05-16 16:55:16 root[line:57] INFO [Eval]: admin
2022-05-16 16:55:16 urllib3.connectionpool[line:228] DEBUG Starting new HTTP connection (1): 127.0.0.1:8077
2022-05-16 16:55:16 urllib3.connectionpool[line:456] DEBUG http://127.0.0.1:8077 "POST http://192.168.255.10:5000/user/login HTTP/1.1" 200 801
2022-05-16 16:55:16 root[line:471] DEBUG [Headers]: {'Content-Type': 'application/json'}
2022-05-16 16:55:16 root[line:472] DEBUG [Request-Entries]: {} {} {'password': '123456', 'account': 'admin'}
2022-05-16 16:55:16 root[line:473] DEBUG [Response]: {'code': 200, 'data': {'current_role_id': 2, 'user_info': {'id': 2, 'username': '管理员', 'user_code': 'admin', 'email': 'admin@qq.com', 'phone': None, 'create_time': '2022-01-21 18:25:30', 'update_time': '2022-01-21 18:25:30', 'locked': '1', 'locked_time': None, 'unlocked_time': None, 'remark': None}, 'user_roles': {'id': 2, 'roles': [{'id': 2, 'role_name': '普通管理员角色', 'role_level': 2, 'introduction': '企业用户管理员角色使用', 'create_user_id': 1}, {'id': 3, 'role_name': '普通用户角色', 'role_level': 3, 'introduction': '企业下普通用户使用', 'create_user_id': 1}]}, 'token': '2-05b245a3487bde0abc8aaa3d16b9d1215752c4d0e7defec149aebdbc756b784d'}}
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.code==200 200
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.data.token==\w+ 2-05b245a3487bde0abc8aaa3d16b9d1215752c4d0e7defec149aebdbc756b784d
2022-05-16 16:55:16 root[line:486] INFO [AssertRight] ！o(￣▽￣)ｄ！
2022-05-16 16:55:16 root[line:57] INFO [Eval]: admin
2022-05-16 16:55:16 urllib3.connectionpool[line:228] DEBUG Starting new HTTP connection (1): 127.0.0.1:8077
2022-05-16 16:55:16 urllib3.connectionpool[line:456] DEBUG http://127.0.0.1:8077 "POST http://192.168.255.10:5000/user/login HTTP/1.1" 200 50
2022-05-16 16:55:16 root[line:471] DEBUG [Headers]: {'Content-Type': 'application/json'}
2022-05-16 16:55:16 root[line:472] DEBUG [Request-Entries]: {} {} {'account': 'admin'}
2022-05-16 16:55:16 root[line:473] DEBUG [Response]: {'code': 40002, 'msg': '密码必填'}
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.code==40002 40002
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.msg==密码必填 密码必填
2022-05-16 16:55:16 root[line:486] INFO [AssertRight] ！o(￣▽￣)ｄ！
2022-05-16 16:55:16 root[line:57] INFO [Eval]: admin
2022-05-16 16:55:16 urllib3.connectionpool[line:228] DEBUG Starting new HTTP connection (1): 127.0.0.1:8077
2022-05-16 16:55:16 urllib3.connectionpool[line:456] DEBUG http://127.0.0.1:8077 "POST http://192.168.255.10:5000/user/login HTTP/1.1" 200 50
2022-05-16 16:55:16 root[line:471] DEBUG [Headers]: {'Content-Type': 'application/json'}
2022-05-16 16:55:16 root[line:472] DEBUG [Request-Entries]: {} {} {'password': None, 'account': 'admin'}
2022-05-16 16:55:16 root[line:473] DEBUG [Response]: {'code': 40002, 'msg': '密码必填'}
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.code==40002 40002
2022-05-16 16:55:16 root[line:32] INFO [Assert]: $.msg==密码必填 密码必填
2022-05-16 16:55:16 root[line:486] INFO [AssertRight] ！o(￣▽￣)ｄ！
```
### 现在仅仅是自动化用例生成测试，后面还有更多的想像空间

如果有兴趣的，联系我或加入我。孤独使人堕落（心有余而力不足）！
