---
title: Electron 17.1.2使用BrowserView实现静默打印
date: 2025-02-24 17:31:09
tags: electron
---
最近在实现静默打印功能，搜索了一下教程看到的都是老版本的使用webview元素实现的，目前最新的17.1.2不再推荐，官方推荐使用BrowserView

#### 第一步：获取到当前设备的打印机列表

```javascript
 //在主线程下，通过ipcMain对象监听渲染线程传过来的getPrinterList事件
 ipcMain.on("getPrinterList", (event) => {
    //主线程获取打印机列表
    const list = mainWindow.webContents.getPrinters();
    //通过webContents发送事件到渲染线程，同时将打印机列表也传过去
    mainWindow.webContents.send("getPrinterList", list);
  });
```

#### 第二步：主线程内监听loadPrinterView（自定义名称）
```javascript
ipcMain.on("loadPrinterView", (event, obj) => {
    //定义BrowserView
    const view = new BrowserView({
      webPreferences: {
        nodeIntegration: true,
        contextIsolation: false,
        nodeIntegrationInWorker: true,
        enableRemoteModule: true,
        devTools: true,
      },
    });
    // mainWindow是主BrowserWindow
    mainWindow.addBrowserView(view);
    // x和y设置负数，可以让打印页面在屏幕上不显示
    view.setBounds({ x: -1000, y: -1000, width: 180, height: 160 });
    // prntURL是打印的html路径
    view.webContents.loadURL(prntURL);
    view.webContents.once("dom-ready", () => {
      // view.webContents.openDevTools(true);
      // 向print.htm发送打印数据
      view.webContents.send("webview-print-render", obj);
    });
	// 监听print.htm页面渲染完成后发送的消息
    view.webContents.on("ipc-message", (event, channel) => {
      // 调用打印方法
      view.webContents.print({
        silent: true,
        printBackground: true,
        deviceName: obj.printObj.name, // 打印机对象的name
      },
      data => {
		// 打印成功后，移除BrowserView
        mainWindow.removeBrowserView(view);
      });
      
      
    });
  });
```

#### 第三步：封装一个公共的打印方法，使用是引入调用即可
```javascript
import { ipcRenderer } from "electron";
/**
 * @param {打印机需要的参数} obj 
 * @param {打印完成的回调} fn 
 */
export function printFn(obj, fn) {
  ipcRenderer.send("getPrinterList");
  //监听主线程获取到打印机列表后的回调
  ipcRenderer.once("getPrinterList", (event, data) => {
    //data就是打印机列表
    if (data.length <= 0) {
        alert("未找到打印机!");
    } else {
      let oIndex = data.findIndex((item) => {
          return item.isDefault === true;
      });
      let printObj = {};
      if(oIndex > -1) {
        // 打印机对象isDefault是默认打印机
        printObj = data[oIndex];
      } else {
        printObj = data[0];
      }
      // loadPrinterView要和第二步监听的字段一致
      ipcRenderer.send("loadPrinterView", {
        name: '大法官-张三'
      });
      fn();
    }
  });
}
```

#### 第四步：定义一个打印页面print.html
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>document</title>
  </head>
  <body>
    <h2 class="name">张三</h2>
    
    <script>
      //引入ipcRenderer对象
      const { ipcRenderer } = require("electron");
      //监听渲染线程传过来的webview-print-render事件
      ipcRenderer.on("webview-print-render", (event, deviceInfo) => {
        // deviceInfo是接受到的打印数据
        document.querySelector(".name").innerHTML = deviceInfo.name;
        //通过ipcRenderer对象的send方法和渲染线程通讯，第二步会监听，然后开始打印
        ipcRenderer.send("webview-print-do");
      });
    </script>
  </body>
</html>

```