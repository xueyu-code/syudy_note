# RTSP学习
全称是(Real Time Streaming Protocol),RFC2326,*实时流传输协议*，是TCP/IP协议中的一个应用层协议(**串流协议**)。该协议定义了一个**一对多应用程序如何有效地通过IP网络传送多媒体数据**
RTSP在体系结构上位于RTP与RTCP之上，它使用TCP或者UDP来完成数据传输
*RTSP是基于文本的协议，采用ISO10646字符集，使用UTF-8编码方案*
RTSP通常来说不负责发送流数据，一般只做控制，发送交给RTP或者RTCP。**RTSP没有规定底层传输层协议，用UDP/TCP/HTTP都可以**
## 与HTTP的不同
- HTTP请求由客户机发出，服务器做出相应
- RTSP中客户机与服务器都可以发出请求，是双向协议
- HTTP传送HTML，RTSP传输多媒体数据
## 实时流媒体会话协议
- SDP（会话描述协议）Session Description Protocol
- RTP(实时传输协议)Real Time Protocol 只能传输多媒体数据，不能控制
*RTSP有控制功能*



