# tcpdump抓包命令

监听网卡 eth0 和 host为x1或x2的tcp连接，并记录到request.cap中

request.cap可以通过wireshark打开

tcpdump -i eth0 tcp and host \(10.16.4.25 or 10.16.4.76\) -w request.cap

