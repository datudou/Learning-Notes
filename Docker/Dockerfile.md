#编写Dockerfile

### `COPY`指令和`ADD`指令
`COPY`指令用来将本地（Dockerfile所在位置）的文件或文件夹复制到编译环境的指定路径下。Dockerfile还提供了另外一个类似的指令：`ADD`。在复制文件方面`ADD`指令和`COPY`指令的格式和效果是完全一样的。这两个指令的区别主要由两点：

1. `ADD`指令可以从一个URL地址下载内容复制到容器的文件系统中;
2. `ADD`指令会将压缩打包格式的文件解开后复制到指定位置，而`COPY`指令只做复制操作。

[引用自:https://github.com/zhangpeihao/LearningDocker/edit/master/manuscript/04-WriteDockerfile.md](https://github.com/zhangpeihao/LearningDocker/edit/master/manuscript/04-WriteDockerfile.md)
