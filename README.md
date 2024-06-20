import re

# 你的配置文本
text = """
name: xxx
type: xxx
platform: xxx
host: xxx
query: xxx
name: xxx2
type: xxx2
platform: xxx2
host: xxx2
query: xxx2
name: xxx3
type: xxx3
platform: xxx3
host: xxx3
query: xxx3
...
name: xxxn
type: xxxn
platform: xxxn
host: xxxn
query: xxxn
"""

# 匹配每个配置块
pattern = re.compile(r'name:\s*(\S+)\s*type:\s*(\S+)\s*platform:\s*(\S+)\s*host:\s*(\S+)\s*query:\s*(\S+)')
matches = pattern.findall(text)

# 生成SQL语句
sql_statements = []
for match in matches:
    name, type_, platform, host, query = match
    sql = f"INSERT INTO jenkins_config (name, type, platform, host, query) VALUES ('{name}', '{type_}', '{platform}', '{host}', '{query}');"
    sql_statements.append(sql)

# 输出结果
for sql in sql_statements:
    print(sql)
