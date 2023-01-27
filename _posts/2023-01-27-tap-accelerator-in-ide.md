---
layout: post
title: When you faced error message while retrieving app accelerator list in VS Code.

categories: [troubleshooting, tap]
---

This is about configurating app accelerator in VS Code when you cannot retrieve App accelerator list in VS Code.

When you click tanzu icon for retrieving accelerator list in VS Code, you see the error message in OUTPUT Panel as below.

Error Message: 
```
connect ECONNREFUSED ::1:80
Error: Error getting accelerators from , connect ECONNREFUSED ::1:80
	at Ar (/Users/hshin/.vscode/extensions/vmware.tanzu-app-accelerator-0.1.5/dist/extension.js:24:2988)
	at process.processTicksAndRejections (node:internal/process/task_queues:96:5)
	at async /Users/hshin/.vscode/extensions/vmware.tanzu-app-accelerator-0.1.5/dist/extension.js:24:7796
```

And you can see the screenshot in app accelerator plugin.
![app-accelerator 1](https://raw.githubusercontent.com/haewons-tanzu/haewons-tanzu.github.io/master/static/img/_posts/2023-01-27-tap-accelerator-in-ide/1.png)

This is because you misconfiured TAP GUI backend URL in Tanzu App Accelerator in extension setting.
If you click the blue "Go to settings" button in the screenshot above, you can see 2 extensions list by searching "tanzu" included.
There are 2 tab such as "User" and "Workspace". You must set the TAP GUI backend URL in Workspace. If you configure it in User, it won't work.

If youy configure here, then you can see the app accelerator list as below!!
![app-accelerator 2](https://raw.githubusercontent.com/haewons-tanzu/haewons-tanzu.github.io/master/static/img/_posts/2023-01-27-tap-accelerator-in-ide/2.png)
