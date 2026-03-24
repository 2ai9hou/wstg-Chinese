# 测试浏览器存储

|ID          |
|------------|
|WSTG-CLNT-12|

## 概述

浏览器为开发人员提供以下客户端存储机制以存储和检索数据：

- LocalStorage
- SessionStorage
- IndexedDB
- Web SQL（已弃用）
- Cookies

这些存储机制可以使用浏览器的开发者工具（如[Google Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/storage/localstorage)或[Firefox的存储检查器](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector)）查看和编辑。

注意：虽然缓存也是存储的一种形式，但它在[单独的部分](../04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md)中介绍，涵盖其自身的特性和关注点。

## 测试目标

- 确定站点是否在客户端存储中存储敏感数据。
- 应检查处理存储对象的代码是否有注入攻击的可能性，如利用未验证的输入或易受攻击的库。

## 如何测试

### LocalStorage

`window.localStorage`是实现[Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)的全局属性，并在浏览器中提供**持久化**键值存储。

键和值都只能是字符串，因此任何非字符串值必须首先转换为字符串才能存储，通常通过[JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)完成。

`localStorage`条目的持久化甚至在浏览器窗口关闭后仍然存在，私人/隐身模式窗口除外。

`localStorage`的最大存储容量因浏览器而异。

#### 列出所有键值条目

```javascript
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### SessionStorage

`window.sessionStorage`是实现[Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)的全局属性，并在浏览器中提供**临时性**键值存储。

键和值都只能是字符串，因此任何非字符串值必须首先转换为字符串才能存储，通常通过[JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)完成。

`sessionStorage`条目是临时性的，因为当浏览器选项卡/窗口关闭时会清除。

`sessionStorage`的最大存储容量因浏览器而异。

#### 列出所有键值条目

```javascript
for (let i = 0; i < sessionStorage.length; i++) {
  const key = sessionStorage.key(i);
  const value = sessionStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### IndexedDB

IndexedDB是一个面向对象的数据库，旨在处理结构化数据。IndexedDB数据库可以具有多个对象存储，每个对象存储可以具有多个对象。

与localStorage和sessionStorage相比，IndexedDB可以存储比字符串更多的内容。任何受[结构化克隆算法](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)支持的对象都可以存储在IndexedDB中。

可以存储在IndexedDB中但不能存储在localStorage/sessionStorage中的复杂JavaScript对象的一个示例是[CryptoKeys](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey)。

关于[Web Crypto API](https://www.w3.org/TR/WebCryptoAPI/)的W3C建议[推荐](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage)需要保存在浏览器中的CryptoKeys存储在IndexedDB中。测试网页时，在IndexedDB中查找任何CryptoKeys，并检查当它们应该设置为`extractable: false`时是否设置为`extractable: true`（即确保底层私钥材料在加密操作期间永不暴露）。

#### 打印IndexedDB的全部内容

```javascript
const dumpIndexedDB = dbName => {
  const DB_VERSION = 1;
  const req = indexedDB.open(dbName, DB_VERSION);
  req.onsuccess = function() {
    const db = req.result;
    const objectStoreNames = db.objectStoreNames || [];

    console.log(`[*] Database: ${dbName}`);

    Array.from(objectStoreNames).forEach(storeName => {
      const txn = db.transaction(storeName, 'readonly');
      const objectStore = txn.objectStore(storeName);

      console.log(`\t[+] ObjectStore: ${storeName}`);

      // 打印对象存储中名为`storeName`的所有条目
      objectStore.getAll().onsuccess = event => {
        const items = event.target.result || [];
        items.forEach(item => console.log(`\t\t[-] `, item));
      };
    });
  };
};

indexedDB.databases().then(dbs => dbs.forEach(db => dumpIndexedDB(db.name)));
```

### Web SQL

Web SQL自2010年11月18日起已弃用，建议Web开发人员不要使用它。

### Cookies

Cookies是一种主要用于会话管理但Web开发人员仍可用于存储任意字符串数据的主要键值存储机制。

Cookies在[测试Cookies属性](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)场景中有广泛介绍。

#### 列出所有Cookies

```javascript
console.log(window.document.cookie);
```

### 全局Window对象

有时Web开发人员通过向全局`window`对象分配自定义属性来初始化和维护仅在页面运行生命周期中可用的全局状态。例如：

```javascript
window.MY_STATE = {
  counter: 0,
  flag: false,
};
```

`window`对象上附加的任何数据在页面刷新或关闭时将丢失。

#### 列出Window对象上的所有条目

```javascript
(() => {
  // 创建一个iframe并追加到body以加载干净的window对象
  const iframe = document.createElement('iframe');
  iframe.style.display = 'none';
  document.body.appendChild(iframe);

  // 获取window上当前属性列表
  const currentWindow = Object.getOwnPropertyNames(window);

  // 根据干净window中存在的属性过滤列表
  const results = currentWindow.filter(
    prop => !iframe.contentWindow.hasOwnProperty(prop)
  );

  // 移除iframe
  document.body.removeChild(iframe);

  // 记录不同的键值条目
  results.forEach(key => console.log(`${key}: ${window[key]}`));
})();
```

*（修改自此[片段](https://stackoverflow.com/a/17246535/3099132)）*

### 安全影响

当审查浏览器存储机制时，测试人员应评估敏感数据是否不必要地在客户端暴露。现代应用程序，特别是SPA，经常在浏览器存储中存储认证令牌或应用程序状态，这可能引入安全风险。

常见问题包括：

- 存储在`localStorage`或`sessionStorage`中的认证令牌（如JWT），可通过JavaScript访问并可能通过XSS暴露。
- 登出后会话标识符或令牌持续存在。
- 在IndexedDB或localStorage中存储敏感业务数据，没有明确要求。
- 密码学材料在应为受保护时被设置为可提取。

不当的客户端存储可能增加客户端攻击（如基于DOM的XSS）的影响。

### 一般测试指导

除了枚举存储条目，测试人员还应：

- 使用开发者工具（Application/Storage选项卡）检查浏览器存储。
- 识别认证令牌、会话标识符或敏感业务数据。
- 尝试通过JavaScript控制台访问存储的值。
- 验证登出或会话到期后是否清除存储条目。
- 评估存储数据是否可用于客户端攻击链。

### 攻击链

在识别上述任何攻击向量后，可以与不同类型的客户端攻击（如[基于DOM的XSS](01-Testing_for_DOM-based_Cross_Site_Scripting.md)攻击）形成攻击链。

## 修复

应用程序应在服务器端存储敏感数据，而不是在客户端，以安全方式遵循最佳实践。

## 参考资料

- [LocalStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [SessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Web Crypto API: 密钥存储](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage)
- [Web SQL](https://www.w3.org/TR/webdatabase/)
- [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

有关HTML5 Web Storage API的更多OWASP资源，请参阅[会话管理速查表](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#html5-web-storage-api)。
