jdk7 引入了 Path 和 Paths 类
	● Path 用来表示文件路径
	● Paths 是工具类，用来获取 Path 实例
```
Path source = Paths.get("1.txt"); // 相对路径 使用 user.dir 环境变量来定位 1.txt
Path source = Paths.get("d:\\1.txt"); // 绝对路径 代表了 d:\1.txt
Path source = Paths.get("d:/1.txt"); // 绝对路径 同样代表了 d:\1.txt
Path projects = Paths.get("d:\\data", "projects"); // 代表了 d:\data\projects
```
	● . 代表了当前路径
	● .. 代表了上一级路径
	