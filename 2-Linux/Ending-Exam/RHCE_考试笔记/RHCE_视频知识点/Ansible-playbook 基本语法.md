#Linux #Ending_Exam #RHCE #视频笔记
[TOC]

# Ansible-playbook 基本语法

## 实施Playbook

> 1. 编写基本的 Ansible Playbook ，再使用 ansible-playbook 命令运行它
> 2. 编写一个使用多个 play 和各 play 特权升级的 palybook
> 3. 有效使用 ansible-doc 学习如何使用新模块来实施 play 任务

### Ansible Playbook 和临时命令

-  **Ad-hoc 临时命令** 可以作为一次性命令对一组目标主机运行一项简单的任务。不过，若要真正 **发挥 Ansible 的力量** ，需要了解如何使用 palybook 以轻松重复的方式对一组目标主机执行多项复杂的任务。
- Ansible 使用 ==YAML 语法描述配置文件== ，YAML 语法以简洁明了、结构清晰著称。
- Ansible 的 ==任务配置文件== 被称为 ==Playbook== ，我们可以称之为 “剧本”。每个剧本（Palybook）中都 **包含一系列的任务** ，这每个任务再 Ansible 中又被称之为 “play” 。一个 **Playbook** 中可以 **包含多个 Play** 。



### YAML基本格式

- 对象

  - 对象表示键值对（key:value）形式出现的数据，使用“冒号+空格”来分开键与值

    > animal: cat

  - Yaml 也允许另一种写法，将所有键值对写成一个行内对象。

    > hash: { name: Steve, foo: bar }

- 数组（列表）

  - 一组连词线开头的行，构成一个数组

    ```yaml
    - Cat
    - Dog
    - Goldfish
    ```

  - YAML 中的数组表示方法为在元素前加上 "-" （和 Markdown 很像）；键值对的表示方法和 JSON 差不多，不过 YAML 中的 key 和字符串不用加引号。

  - YAML 中的数组和对象也可以嵌套，其逻辑上与 JSON 是一致的

> 对象支持多层嵌套（用缩进表示层级关系）

```yaml
# YAML
key:
  child-key1: value1
  child-key2: value2
  child-key3: value3
```

```json
// JSON
"key":{
  "child-key1": "value1",
  "child-key2": "value2",
}
```

> 支持流式风格（Flow style）的语法（用花括号包裹，用逗号加空格分隔，类似 JSON）

```yaml
# YAML
key: { child-key1: value1, child-key2: value2 }
```

```JSON
// JSON
"key":{"child-key1": "value1", "child-key2": "vlaue2"}
```

> 一组以区块格式（Block Format）（即“破折号+空格”）开头的数据组成的一数组

```yaml
# YAML
values:
 - value1
 - value2
 - value3
```

```JSON
// JSON
"values": ["value1","value2", "value3"]
```

> 数据结构的子成员是一个数组，则可以再该项下面缩进一个空格

```shell
-
- Cat
- Dog
- Goldfish
```

- 数组也可以采用行内表示法

```shell
animal:[Cat, Dog]
```

- 值可以使用 | 或 > 跨多行。使用“文字块标量”跨越多行 | 将包括换行符和任何末尾空格。使用“折叠块标量” > 将折换行到空格；它是用来是一个很长的行更容易阅读和编辑

  ```yaml
  include_newlines: |
          exactly as you see
          will appear these three
          lines of poetry
  fold_newlines: >
          this is reslly a 
          single line of text
      despite appearances
  ```

  


#### 单、双引号的区别在于，双引号中可以使用转义

- foo: "a \t TAB and \n NEWWLINE"

### 格式化 Ansible Playbook



### TAB缩进

-  **只有空格字符可用于缩进** ；==不允许使用制表符（tab键）== 。
- 使用 vi 编辑器时，可以应用一些设置，以便轻松地编辑 playbook 。例如在  **￥HOME/.vimrc** 文件中添加 `autocmd FileType yaml setlocal ai ts=2 sw=2 et`
- Playbook 开头的一行由三个破折号(---) 组成，这是文档 ==开始标记== 

