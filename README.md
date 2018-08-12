# dexieJsdemo
a demo abour fetch database's version with dexie.js
主要介绍初始化后如何获得indexedDB的version
1.如果浏览器中不存在数据库时，直接创建数据库

var db = new Dexie(dbName); 
window.db = db; 
//无数据时，默认version为1，须是db.close();的时候设置 
db.version(1).stores();

2.浏览器中存在数据库时，需先找到其version的，再根据version来修改或者更新数据库，获取当前浏览器的数据库的version:

 db = new Dexie(dbName);
          db.open().then(result => {
            //打开成功后，将version存下来
            version = db.verno;
            resolve(db);
          })
1
2
3
4
5
6
列表内容

3.对Dexie.js中exists的理解 
官网说明:
You normally won’t need this method! Dexie will automatically detect if database creation or upgrade is needed. The following checks are made withing db.open():

If database does not exists, Dexie will create it automatically.
If database exists but on a previous version, Dexie will upgrade it for you.
If database exists on the declared version, Dexie will just open it for you.

和db.open()配合使用，new Dexie（“dataName”）：database不存在时，自动创建，database存在但是先前的版本就升级，database存在且定义当前的版本则打开它 
4.具体可见如下简单的code


