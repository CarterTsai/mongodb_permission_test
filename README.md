Mongodb
===========

### 使用 Admin 登入 Mongodb

#### 使用 `mongosh` 連線到 MongoDB

1. 確保 MongoDB 容器已啟動：
   ```bash
   docker-compose up -d
   ```

2. 使用 `mongosh` 連線：
   ```bash
   mongosh "mongodb://root:example@localhost:27017"
   ```

3. 常用指令：
   ```bash
   show dbs
   use test
   show collections
   db.collectionName.insertOne({ key: "value" })
   db.collectionName.find()
   ```

#### 插入測試資料

以下範例將資料插入到 `testData` 集合中：
```javascript
db.testData.insertOne({ name: "John Doe", age: 30, email: "johndoe@example.com" })
db.testData.insertOne({ name: "Jane Smith", age: 25, email: "janesmith@example.com" })
db.testData.insertOne({ name: "Alice Brown", age: 28, email: "alicebrown@example.com" })

db.testData.find()
```

#### 新增帳號並設定權限

以下範例新增一組帳號，並設定權限：

```javascript
// 1. 只讀 testDataView
db.createRole({
  role: "readViewOnly",
  privileges: [
    {
      resource: { db: "test", collection: "testDataView" },
      actions: ["find"]
    }
  ],
  roles: []
});

// 2. 只對 testData 做 insert & remove
db.createRole({
  role: "writeDeleteOnly1",
  privileges: [
    {
      resource: { db: "test", collection: "testData" },
      actions: ["insert", "remove"]
    }
  ],
  roles: []
});

// 新增帳號
db.createUser({
   user: "testUser",
   pwd: "testPassword",
   roles: [
      { role: "readViewOnly", db: "test" }, // 對 testDataView 具有讀取權限
      { role: "writeDeleteOnly1", db: "test" } // 對 testData 具有管理權限（包含刪除）
   ]
})

// 驗證帳號
db.auth("testUser", "testPassword")
```

#### 使用 `testUser` 登入

以下範例展示如何使用 `testUser` 登入 MongoDB：

```bash
mongosh "mongodb://testUser:testPassword@localhost:27017/test"
```

#### 建立 View

以下範例建立一個名為 `testDataView` 的 View，來源為 `testData` 集合：
```javascript
db.createView(
   "testDataView", // View 的名稱
   "testData",     // 來源集合名稱
   [
      {
         $project: {
            _id: 0,          // 隱藏 `_id` 欄位
            name: 1,         // 顯示 `name` 欄位
            age: 1,          // 顯示 `age` 欄位
            email: 1         // 顯示 `email` 欄位
         }
      }
   ]
)

db.testDataView.find()
```

#### 驗證權限：嘗試刪除 `testData` 的資料

以下範例展示如何嘗試刪除 `testData` 的資料，並驗證權限：

```javascript
// 嘗試刪除 testData 的所有資料
try {
   db.testData.insertOne({ name: "New User", age: 40, email: "newuser@example.com" });
   console.log("寫入資料到 testData 的資料");
   db.testData.deleteMany({});
   console.log("成功刪除 testData 的資料");
   db.testData.find(); // 應該會顯示權限錯誤
} catch (error) {
   console.error("權限不足，讀取 testData 的資料:", error.message);
}
```

#### 驗證權限：嘗試刪除 `testDataView` 的資料

以下範例展示如何嘗試刪除 `testDataView` 的資料，並驗證權限：

```javascript
// 嘗試刪除 testDataView 的所有資料
try {
   db.testDataView.find()
   console.log("可以查詢資料到 testData 的資料");
   db.testDataView.deleteMany({});
} catch (error) {
   console.error("權限不足，無法刪除 testDataView 的資料:", error.message);
}
```

#### 刪除使用者 

1. 刪除使用者
db.dropUser("testUser")

2. 驗證刪除
db.getUsers()