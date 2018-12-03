
HttpRunner 在命令行中启动测试时，通过指定参数，可实现丰富的测试特性控制。

```text
$ hrun -h
usage: main-debug.py [-h] [-V] [--log-level LOG_LEVEL] [--log-file LOG_FILE]
                     [--dot-env-path DOT_ENV_PATH]
                     [--report-template REPORT_TEMPLATE]
                     [--report-dir REPORT_DIR] [--failfast] [--save-tests]
                     [--startproject STARTPROJECT]
                     [--validate [VALIDATE [VALIDATE ...]]]
                     [--prettify [PRETTIFY [PRETTIFY ...]]]
                     [testcase_paths [testcase_paths ...]]

One-stop solution for HTTP(S) testing.

positional arguments:
  testcase_paths        testcase file path

optional arguments:
  -h, --help            show this help message and exit
  -V, --version         show version
  --log-level LOG_LEVEL
                        Specify logging level, default is INFO.
  --log-file LOG_FILE   Write logs to specified file path.
  --dot-env-path DOT_ENV_PATH
                        Specify .env file path, which is useful for keeping
                        sensitive data.
  --report-template REPORT_TEMPLATE
                        specify report template path.
  --report-dir REPORT_DIR
                        specify report save directory.
  --failfast            Stop the test run on the first error or failure.
  --save-tests          Save loaded tests and parsed tests to JSON file.
  --startproject STARTPROJECT
                        Specify new project name.
  --validate [VALIDATE [VALIDATE ...]]
                        Validate JSON testcase format.
  --prettify [PRETTIFY [PRETTIFY ...]]
                        Prettify JSON testcase format.
```

## 指定测试用例路径

使用 HttpRunner 指定测试用例路径时，支持多种方式。

使用 hrun 命令外加单个测试用例文件的路径，运行单个测试用例，并生成一个测试报告文件：

```text
$ hrun filepath/testcase.yml
```

将多个测试用例文件放置到文件夹中，指定文件夹路径可将文件夹下所有测试用例作为测试用例集进行运行，并生成一个测试报告文件：

```text
$ hrun testcases_folder_path
```

使用 hrun 命令外加多个测试用例文件（文件夹）的路径，即可依次运行多个测试用例（用例集），并分别生成对应的测试报告文件：

```text
$ hrun filepath1/testcase1.yml filefolder2/
```

## failfast

默认情况下，HttpRunner 会运行指定用例集中的所有测试用例，并统计测试结果。

> 对于某些依赖于执行顺序的测试用例，例如需要先登录成功才能执行后续接口请求的场景，当前面的测试用例执行失败后，后续的测试用例也都必将失败，因此没有继续执行的必要了。

若希望测试用例在运行过程中，遇到失败时不再继续运行后续用例，则可通过在命令中添加`--failfast`实现。

```text
$ hrun filepath/testcase.yml --failfast
```

## 日志级别

默认情况下，HttpRunner 运行时的日志级别为`INFO`，只会包含最基本的信息，包括用例名称、请求的URL和Method、响应结果的状态码、耗时和内容大小。

```text
$ hrun tests/httpbin/hooks.yml
INFO     Loading environment variables from tests/.env
INFO     Start to run testcase: basic test with httpbin
headers
INFO     GET http://httpbin.org:80/headers
INFO     status_code: 200, response_time(ms): 491.48 ms, response_length: 185 bytes
.
alter response
INFO     GET http://httpbin.org:80/headers
INFO     status_code: 200, response_time(ms): 248.41 ms, response_length: 185 bytes
.

----------------------------------------------------------------------
Ran 2 tests in 0.845s

OK
INFO     Start to render Html report ...
INFO     Generated Html report: reports/1543827143.html
```

若需要查看到更详尽的信息，例如请求的参数和响应的详细内容，可以将日志级别设置为`DEBUG`，即在命令中添加`--log-level debug`。

```
$ hrun tests/httpbin/hooks.yml --log-level debug
INFO     Loading environment variables from tests/.env
DEBUG    Set OS environment variable: UserName
DEBUG    Set OS environment variable: Password
DEBUG    Set OS environment variable: PROJECT_KEY
INFO     Start to run testcase: basic test with httpbin
headers
DEBUG    call setup hook actions.
DEBUG    call hook function: ${setup_hook_prepare_kwargs($request)}
DEBUG    call hook function: ${setup_hook_add_kwargs($request)}
DEBUG    call hook function: ${setup_hook_remove_kwargs($request)}
INFO     GET http://httpbin.org:80/headers
DEBUG    request kwargs(raw): {'verify': True}
DEBUG    processed request:
> GET http://httpbin.org:80/headers
> kwargs: {'verify': True, 'timeout': 120}
DEBUG
================== request details ==================
url              : 'http://httpbin.org:80/headers'
method           : 'GET'
headers          : {'User-Agent': 'python-requests/2.20.0', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'Connection': 'keep-alive'}
start_timestamp  : 1543827400.7875352
verify           : True

DEBUG
================== response details ==================
status_code      : 200
headers          : {'Connection': 'keep-alive', 'Server': 'gunicorn/19.9.0', 'Date': 'Mon, 03 Dec 2018 08:56:41 GMT', 'Content-Type': 'application/json', 'Content-Length': '185', 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Credentials': 'true', 'Via': '1.1 vegur'}
content_size     : 185
response_time_ms : 528.63
elapsed_ms       : 520.948
encoding         : None
content_type     : 'application/json'
ok               : True
url              : 'http://httpbin.org:80/headers'
reason           : 'OK'
cookies          : {}
json             : {'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Connection': 'close', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.20.0'}}

INFO     status_code: 200, response_time(ms): 528.63 ms, response_length: 185 bytes

DEBUG    call teardown hook actions.
DEBUG    call hook function: ${teardown_hook_sleep_N_secs($response, 1)}
DEBUG    start to validate.
DEBUG    extract: status_code   => 200
DEBUG    validate: status_code equals 200(int)  ==> pass
DEBUG    extract: content.headers.Host  => httpbin.org
DEBUG    validate: content.headers.Host contained_by http://httpbin.org:80(str) ==> pass
.
alter response
DEBUG    call setup hook actions.
DEBUG    call hook function: ${setup_hook_prepare_kwargs($request)}
INFO     GET http://httpbin.org:80/headers
DEBUG    request kwargs(raw): {'verify': True}
DEBUG    processed request:
> GET http://httpbin.org:80/headers
> kwargs: {'verify': True, 'timeout': 120}
DEBUG
================== request details ==================
url              : 'http://httpbin.org:80/headers'
method           : 'GET'
headers          : {'User-Agent': 'python-requests/2.20.0', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'Connection': 'keep-alive'}
start_timestamp  : 1543827401.4223402
verify           : True

DEBUG
================== response details ==================
status_code      : 200
headers          : {'Connection': 'keep-alive', 'Server': 'gunicorn/19.9.0', 'Date': 'Mon, 03 Dec 2018 08:56:41 GMT', 'Content-Type': 'application/json', 'Content-Length': '185', 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Credentials': 'true', 'Via': '1.1 vegur'}
content_size     : 185
response_time_ms : 269.15
elapsed_ms       : 267.673
encoding         : None
content_type     : 'application/json'
ok               : True
url              : 'http://httpbin.org:80/headers'
reason           : 'OK'
cookies          : {}
json             : {'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Connection': 'close', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.20.0'}}

INFO     status_code: 200, response_time(ms): 269.15 ms, response_length: 185 bytes

DEBUG    call teardown hook actions.
DEBUG    call hook function: ${alter_response($response)}
DEBUG    start to validate.
DEBUG    extract: status_code   => 500
DEBUG    validate: status_code equals 500(int)  ==> pass
DEBUG    extract: headers.content-type  => html/text
DEBUG    validate: headers.content-type equals html/text(str) ==> pass
DEBUG    extract: json.headers.Host => 127.0.0.1:8888
DEBUG    validate: json.headers.Host equals 127.0.0.1:8888(str) ==> pass
DEBUG    extract: content.headers.Host  => 127.0.0.1:8888
DEBUG    validate: content.headers.Host equals 127.0.0.1:8888(str)      ==> pass
DEBUG    extract: text.headers.Host => 127.0.0.1:8888
DEBUG    validate: text.headers.Host equals 127.0.0.1:8888(str) ==> pass
.

----------------------------------------------------------------------
Ran 2 tests in 0.908s

OK

DEBUG    No html report template specified, use default.
INFO     Start to render Html report ...
INFO     Generated Html report: reports/1543827400.html
```

## 保存详细过程数据

为了方便定位问题，运行测试时可指定 `--save-tests` 参数，即可将运行过程的中间数据保存为日志文件。

日志文件将保存在项目根目录的 `logs` 文件夹中，生成的文件有如下三个（XXX为测试用例名称）：

- `XXX.loaded.json`：测试用例加载后的数据结构内容，加载包括测试用例文件（YAML/JSON）、debugtalk.py、.env 等所有项目文件
- `XXX.parsed.json`：测试用例解析后的数据结构内容，解析内容包括测试用例引用（API/testcase）、变量计算和替换、base_url 拼接等
- `XXX.summary.json`：测试报告生成前的数据结构内容

```text
hrun tests/httpbin/hooks.yml --save-tests
INFO     Loading environment variables from tests/.env
dump file: tests/logs/hooks.loaded.json
dump file: tests/logs/hooks.parsed.json
INFO     Start to run testcase: basic test with httpbin
headers
INFO     GET http://httpbin.org:80/headers
INFO     status_code: 200, response_time(ms): 589.45 ms, response_length: 185 bytes
.
alter response
INFO     GET http://httpbin.org:80/headers
INFO     status_code: 200, response_time(ms): 313.6 ms, response_length: 185 bytes
.

----------------------------------------------------------------------
Ran 2 tests in 1.009s

OK
dump file: tests/logs/hooks.summary.json
INFO     Start to render Html report ...
INFO     Generated Html report: reports/1543827620.html
```