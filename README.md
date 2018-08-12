# dexieJsdemo
a demo abour fetch database's version with dexie.js
主要介绍初始化后如何获得indexedDB的version
如果浏览器中不存在数据库时，直接创建数据库

var db = new Dexie(dbName); 
window.db = db; 
//无数据时，默认version为1，须是db.close();的时候设置 
db.version(1).stores();

浏览器中存在数据库时，需先找到其version的，再根据version来修改或者更新数据库，获取当前浏览器的数据库的version:

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

对Dexie.js中exists的理解 
官网说明:
You normally won’t need this method! Dexie will automatically detect if database creation or upgrade is needed. The following checks are made withing db.open():

If database does not exists, Dexie will create it automatically.
If database exists but on a previous version, Dexie will upgrade it for you.
If database exists on the declared version, Dexie will just open it for you.
1
2
3
4
5
和db.open()配合使用，new Dexie（“dataName”）：database不存在时，自动创建，database存在但是先前的版本就升级，database存在且定义当前的版本则打开它 
4.具体可见如下简单的demo

<html>
<head>
  <meta charset="utf-8">
  <script src="https://unpkg.com/dexie/dist/dexie.js"></script>
  <script>
    //当前的indexedDB的version
    var version = 1;
    var dataBase;
    createDB().then(db => {
      dataBase = db;
      version = db.verno;
      alert("创建数据库成功,当前版本：" + db.verno);
    });

    //获取当前的version
    function createDB(dbName) {
      dbName = dbName || 'zq-dataBase';
      //調用創建数据库的方法，如果dbName名的不存在则会创建，若已有了，则会将现有的数据库赋值上
      var db = new Dexie(dbName);
      window.db = db;
      //无数据时，默认version为1，须是db.close();的时候设置
      db.version(version).stores();
      //promise函数
      return new Promise(((resolve, reject) => {
        //打开数据库时，会判断当前version值是否大于已经存在的version值，若大于则会upgrade即升到最高版本
        db.open().then(result => {
          //打开成功后
          version = db.verno;
          resolve(db);
        }).catch('VersionError', e => {
          //若所设version小于当前的version,则会报versionError,此时
          //关闭后
          db.close();
          //重新创建数据库，此时不设置db.version(version).stores();,会将当前的version设置为当前的
          db = new Dexie(dbName);
          db.open().then(result => {
            //打开成功后，将version存下来
            version = db.verno;
            resolve(db);
          }).catch(e => {
            reject("failure")
            console.error(e);
          })
        }).catch(e => {
          reject("failure")
          console.error(e);
        });
      }));
    }

    //给数据库创表和修改version
    function addTables() {
      //设置表名
      dataBase.close();
      Dexie.delete('zq-dataBase');
      version = (version * 10 + 1)/10;
      dataBase = new Dexie('zq-dataBase');
      dataBase.version(version).stores({
        friends: "++id,name,age,isCloseFriend",
        car: "++id,band,color,seats"
      });
      dataBase.open().then(result =>{
        alert("修改版本成功，当前的version："+dataBase.verno);
      });
    }


    //删除数据库表
    function deleteTables() {
      Dexie.delete('zq-dataBase');
      version = 1;
    }

    //获得表数据
    function getData() {
       var list = document.getElementById("list");
       var dataList = dataBase.tables;
       var ulnode = document.createElement("ul");
       console.log(dataList,"----------");
       for(var i = 0;i < dataList.length;i++){
         var li = document.createElement("li");
         li.innerHTML = dataList[i].name;
         ulnode.appendChild(li);
       }
       list.appendChild(ulnode);
    }

  </script>
  <style>
    .content{
      display: flex;
      display: -webkit-flex;
      flex-direction: column;
      width: 400px;
      height: auto;
      background-color: lightgrey;
    }
    .content .item{
      width: 200px;
      height: 100px;
      margin: 10px;
    }
    .content .item button{
      width: 200px;
      height: 100px;
    }
    .content .left{
      background-color: rebeccapurple;
    }
    .content .middle{
      background-color: aqua;
    }
    .content .right{
      background-color: burlywood;
    }
  </style>
</head>
<body>
<div class="content">
  <div class="item left">
    <button onclick="addTables()">改变数据库</button>
  </div>
  <div class="item middle">
    <button onclick="deleteTables()">删除数据库</button>

  </div>
  <div class="item right">
    <button onclick="getData()">获取数据库表</button>

  </div>
</div>

<div id="list"></div>
</body>
</html>
