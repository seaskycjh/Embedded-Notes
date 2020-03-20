1 功能

基于S3C2440开发板的http服务器，支持并发，支持cgi程序，支持上传文件，支持图片、音乐、视频等多媒体文件。基于Linux的qt客户端，用来发生http请求，主要功能有LED控制、蜂鸣器控制、上传/下载文件、摄像头控制（视频监控、拍照等）。视频监控功能通过移植mjpg-streamer实现。其他控制功能通过一个cgi程序以及一个后台控制程序实现。

2 基础知识

2.1 HTTP协议

- GET和POST的区别：

服务器需要设置的环境变量：

```html
QUERY_STRING    /* 重要，对 get 方式来说不可缺少 */
HTTP_COOKIE     /* 浏览器 cookie */
CONTENT_TYPE    /* 重要，对 cgi post 方式不可缺少 */
CONTENT_LENGTH  /* 重要，对 cgi post 方式不可缺少 */
```

- putenv 与 setenv 的区别：

**putenv**可以使用已有的变量，不给此变量所指向的内容重新分配空间。

**setenv**则是每一个环境变量都重新分配一次空间，即使参数不是一个字符串常量也会重新为其分配空间。

2.2 CGI编程

通过getenv函数即可获得环境变量（如QUERY_STRING），通过标准输入即可读取POST参数，如scanf函数，通过标准输出向客户端发送信息，如printf函数。具体如何实现见 CgiWriteToClient 函数。

2.3 QT编程

3 实现

3.1 http服务器

```c
int main(int argc, char *argv[])
{
    /* 格式信息初始化 */
    FormatMgrInit();
    /* 为服务器准备环境 */
    ReadyEnv();
    /* 创建服务器套接字 */
    CreateServerSocket();
    /* 父进程退出 */
    if(fork()) ...;
    /* 子进程监听客户端连接 */
    while(1){
        /* 创建子进程处理客户端请求 */
        if(fork() == 0){
            HandleRequest(clientSocketFd);
        }else{
            close(clientSocketFd)
        }
    }
}
int FormatMgrInit(void){
    /* cgi格式初始化 */
    CgiInit();
    /* html格式初始化 */
    HtmlInit();
    /* 任何类型文件 */
    PlainInit();
}
void *HandleRequest(void *Data){
    /* 获取客户端请求头部 */
    GetRequestHeader(iClientSocketFd, &tReqHeader);
    /* 响应请求 */
    PutResponseHeader(iClientSocketFd, &tReqHeader);
}
int GetRequestHeader(int iSockFd, struct RequestHeader *ptReqHeader){
    /* 获取请求行（包含了请求方法，url 以及 http 版本号） */
    GetLineFromSock(iSockFd, strBuf, sizeof(strBuf));
    /* 获取请求头部 */
    /* 动态分配POST参数的内存空间 */
    ptReqHeader->strPostArgs = malloc(iPostArgLen);
    /* 获取POST参数 */
    GetBytesFromSock(iSockFd, ptReqHeader->strPostArgs, iPostArgLen);
}
int PutResponseHeader(int iSockFd, struct RequestHeader *ptReqHeader){
    /* 提取URL中的参数 */
    /* 获取文件信息 */
    stat(strPath, &reqFileStat);
    /* 设置环境变量 */
    SetEnv(ptReqHeader);
    /* 执行文件 */
    RegularFileExec(iSockFd, strPath, ptReqHeader);
}
/* 为CGI程序创建环境变量 */
void SetEnv(struct RequestHeader *ptReqHeader){
    setenv("REQUEST_METHOD", ptReqHeader->strMethod, 1);
    /* 根据GET/POST请求方法设置不同环境变量 */
    if(!strcmp(ptReqHeader->strMethod, "GET")){
        setenv("QUERY_STRING", ptReqHeader->strGetArgs, 1);
    }else if(!strcmp(ptReqHeader->strMethod, "POST")){
        setenv("CONTENT_TYPE", ptReqHeader->strContType, 1);
    	setenv("CONTENT_LENGTH", strTmp, 1);
    }
}
int RegularFileExec(int iClient, char *strPath, struct RequestHeader *ptReqHeader){
    /* 打开文件并获取文件信息 */
    /* 将文件映射到虚拟内存 */
    /* 根据文件类型获取对应的支持格式信息（html/cgi/plain） */
    ptFormatMrg = GetSupportedFormatMgr(&tFileDesc);
    /* 发送文件到客户端 */
    ptFormatMrg->WriteToClient(iClient, &tFileDesc, ptReqHeader);
    /* 取消内存映射 */
    munmap(pucMemStart, tFStat.st_size);
}
/* 执行cgi程序并发送结果到客户端 */
int CgiWriteToClient(int iClient, struct FileDesc *ptFileDesc, struct RequestHeader *ptReqHeader){
    /* 创建管道 */
    pipe(iCgiOutFd);
    pipe(iCgiInFd);
    /* 创建子线程 */
    ForkPid = fork();
    /* 执行cgi程序并写入客户端 */
    if(ForkPid == 0){
        dup2(iCgiOutFd[1], STDOUT);  /* 把标准输出连接到管道输出 */
        dup2(iCgiInFd[0], STDIN);  /* 把标准输入连接到管道输入 */
        close(iCgiOutFd[0]);
        close(iCgiInFd[1]);
        execl(ptFileDesc->strFName, NULL, NULL);
    }else{  /* 主进程 */
        close(iCgiOutFd[1]);
        close(iCgiInFd[0]);
        /* 将参数通过管道传递给cgi程序的标准输入 */
        write(iCgiInFd[1], strTmp, ptReqHeader->iContLen);
        /* 通过管道从cgi程序的标准输出读取数据并发送到客户端 */
        while(read(iCgiOutFd[0], &cByte, 1) > 0){
            send(iClient, &cByte, 1, 0);
        }
    }
}


```

```c
/* mjpg-streamer */
./mjpg_streamer -o "output_http.so -w ./www" -i "input_uvc.so -y"
```

