# 升级php7


---

#以知识站点为例
## 1. 修改ZhishiRDHandle
首先因为php7移除了一些扩展，像mysql，mssql都已经移除

在ZhishiRDHandle中使用的mysql_escape_string函数已经不在支持，为了便于测试，我在本地测试的时候直接删除了这个过滤的方法，建议之后是用mysqli和pdo的占位符的方式。

**解决方案1**：
就是使用拥有Prepared Statement机制的PDO和MYSQLi来代替
例如：
PDO：

    $pdo = new PDO('mysql:dbname=dbtest;host=127.0.0.1;charset=utf8', 'root', '123456');
    
    $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $stmt = $pdo->prepare('SELECT * FROM user WHERE name = :name');
    $stmt->execute(array('name' => $name));
    

MYSQLi：

    $stmt = $dbConnection->prepare('SELECT * FROM user WHERE name = ?');
    $stmt->bind_param('s', $name);
    
    $stmt->execute();
    
    $result = $stmt->get_result();
    while ($row = $result->fetch_assoc()) {
        // do something with $row
    }
    
**解决方案2：**
使用addslashes代替mysql_escape_string，mysql_real_escape_string 这种还是有addslashes和mysql_escape_string，mysql_real_escape_string还是有一定区别的

* 区别1：mysql_real_escape_string 转义任何MySQL需要的转义 ，而有些addslashes是不会转义的。
如与addslashes对比，mysql_real_escape_string同时还对\r、\n和\x1a(16进制)进行转义。这些字符必须正确地告诉MySQL，否则会得到错误的查询结果。
* 区别2：在GBK里，0xbf27不是一个合法的多字符字符，但0xbf5c却是。在单字节环境里，0xbf27被视为0xbf后面跟着0x27(')，同时0xbf5c被视为0xbf后面跟着0x5c(\)。
一个用反斜杠转义的单引号，是无法有效阻止针对MySQL的SQL注入攻击的。如果你使用addslashes，那么，攻击者只要注入一些类似0xbf27，然后addslashes将它修改为0xbf5c27，一个合法的多字节字符后面接着一个单引号。换句话说，攻击者可以无视你的转义，成功地注入一个单引号。这是因为0xbf5c被当作单字节字符，而非双字节。
例如：
```php
<?php 
    /*
    CREATE TABLE `users` (
    `username` varchar(32) CHARACTER SET gbk NOT NULL DEFAULT '',
    `password` varchar(32) CHARACTER SET gbk DEFAULT NULL,
    PRIMARY KEY (`username`)
    ) 
    */
    $mysqli = new mysqli("127.0.0.1", "root", "123456", "test");
    $mysqli->query("SET NAMES 'gbk'");
    /* SQL Injection Example */
    $_POST['username'] = chr(0xbf) . chr(0x27) .' OR 1 = 1 #';
    $_POST['password'] = 'lixin';
    $username = addslashes($_POST['username']);
    $password = addslashes($_POST['password']);
    $sql = "SELECT * FROM   users  WHERE  username = '{$username}' AND password = '{$password}'";
    echo $sql.'<br/>';
    $result = $mysqli->query($sql);
    if ($result->num_rows) {
        echo $result->num_rows.'<br>';
        echo 'Success';
        /* Success */
    } else {
        /* Failure */
        echo 'Failure';
    }
```
即可成功注入攻击。

总结：建议还是使用mysqli和pdo的占位符机制更安全。
## 2. 修改Smarty_Compiler.class.php
报错是

     preg_replace(): The /e modifier is no longer supported, use preg_replace_callback instead in /home/lixin/fang/trunk/include/Smarty/Smarty_Compiler.class.php on line 269
    

在Smarty_Compiler.class.php修改264-270行

        /* replace special blocks by "{php}" */
        $source_content = preg_replace($search.'e', "'"
                                       . $this->_quote_replace($this->left_delimiter) . 'php'
                                       . "' . str_repeat(\"\n\", substr_count('\\0', \"\n\")) .'"
                                       . $this->_quote_replace($this->right_delimiter)
                                       . "'"
                                       , $source_content);
                                       
为

       /* replace special blocks by "{php}" */
       $source_content = preg_replace_callback($search, create_function ('$matches', "return '" 
                               . $this->_quote_replace($this->left_delimiter) . 'php' 
                               . "' . str_repeat(\"\n\", substr_count('\$matches[1]', \"\n\")) .'" 
                               . $this->_quote_replace($this->right_delimiter) 
                               . "';") 
                               , $source_content); 

## 3.修改MySmarty.class.php

    Warning: Declaration of MySmarty::display($templateName, $time = false) should be compatible with Smarty::display($resource_name, $cache_id = NULL, $compile_id = NULL) in /home/lixin/fang/trunk/include/MySmarty.php on line 32

这是因为php7的错误发生了变化，具体见http://php.net/manual/zh/migration70.incompatible.php。这个不改应该也能跑。
在display()方法中多加了两个参数，$cache_id，$compile_id用来兼容父类方法

    public function display($templateName, $time=false, $cache_id = NULL, $compile_id = NULL)
