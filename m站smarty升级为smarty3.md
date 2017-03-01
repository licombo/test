#M站Smarty升级为Smarty3

---

##smarty3源码修改部分

* ###smarty.php第81行false改为false

* ###parent::Smarty()->parent::__construct(); 

* ###注册过滤器机制的改变

smarty2：

    $this->register_outputfilter(array(&$this,'htmlConcentrate')); 
    

smarty3:
 
    //前置过滤器

    $this->registerFilter('pre', array(&$this,'htmlConcentrate')); <br />
    
* ###获取数组长度的方法   

smarty2：

    {count($Arr)}
    {$Arr|@count}
    {$Arr|count}
    
在smarty3中，写在if条件中**{if $housePolicy.item |@count>0}**，会报fetal error
    
smarty3:建议使用
	
	{count($Arr)}
	{if count($housePolicy.item) > 0}
	
* ###inc.php中set_include_path添加一个目录

	\_ROOTPATH\_."/include/Smarty/sysplugins"

     在include的时候加了个判断
     else if (strpos($classname, 'Smarty') !== false && $classname != 'Smarty' && $classname != 'Smarty_Autoloader' && $classname != 'SmartyBC' && $classname != 'SmartyBC' && $classname != 'MySmarty') {
                //因为smarty3中sysplugins文件夹下类名是大写的,但是文件名有些并不是
                $classname = strtolower($classname);
                include $classname . '.php';
     }