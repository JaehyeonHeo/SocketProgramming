# ✅ Study_SocketProgramming ✅

```
<< TCP/IP 소캣 프로그래밍 >>
 리눅스 기반 webserver를 만들고 이미지 띄우기
```
![SocketProgramming](https://raw.githubusercontent.com/JaehyeonHeo/SocketProgramming/7c0044757a88b916aab45cfb306923a59716dc6d/image/TcpImage.jpg "실행화면")

```C
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<sys/sendfile.h>
#define TRUE 1
char webpage[] ="HTTP/1.1 200 OK\r\n"  //상태 라인
                 "Server:Linux Web Server\r\n"
                 "Content-Type: text/html; charset=UTF-8\r\n\r\n"
                 "<!DOCTYPE html>\r\n”// 웹브라우저에 HTML5로 작성
                 "<html><head><title> My Web Page </title>\r\n"
                 "<style>body {background-color: #FFFF00 }</style></head>\r\n"
                 "<body><center><h1>Hello world!!</h1><br>\r\n"
                 "<img src=\"game.jpg\"></center></body></html>\r\n";

int main(int argc, char*argv[])
 {
     int serv_sock, clnt_sock;
     struct sockaddr_in serv_addr, clnt_addr;
     socklen_t sin_len =sizeof(clnt_addr);
     char buf[2048];
     int fdimg;
     int option = TRUE;
     char img_buf[700000];

     serv_sock = socket(AF_INET, SOCK_STREAM, 0);
     // 주소 재할당
     setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, &option, sizeof(int));

     memset(&serv_addr, 0, sizeof(serv_addr));
     serv_addr.sin_family=AF_INET;
     serv_addr.sin_addr.s_addr=INADDR_ANY;
     serv_addr.sin_port=htons(atoi(argv[1]));
     if(bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr))==-1)
         perror("bind() error");
     if(listen(serv_sock, 5)==-1)
         perror("listen() error");
         
     while(1) {
         clnt_sock=accept(serv_sock, (struct sockaddr*)&clnt_addr, &sin_len);
         puts("New client connected,,,,,");
         read(clnt_sock, buf, 2047);
         printf("%s\n", buf);

         if(!strncmp(buf, "GET /game.jpg", 13))  {
             fdimg = open("game.jpg", O_RDONLY);
             sendfile(clnt_sock, fdimg, NULL, 700000);
                 //read-write 함수를 통한 구현부분
             read(fdimg, img_buf, sizeof(img_buf));
             write(clnt_sock, img_buf, sizeof(img_buf));

             close(fdimg);
         }
         else
             //send(clnt_sock, webpage, sizeof(webpage), 0);
             write(clnt_sock, webpage, sizeof(webpage)-1);
             puts("closing,,,");
     }
         close(clnt_sock);
         close(serv_sock);
     return0;
 }
```
