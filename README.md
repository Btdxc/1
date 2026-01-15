package main

import (
	"fmt"
	"io"
	"log"
	"net"
)

const host = "node4.anna.nssctf.cn:25939"

func SendRequest(conn net.Conn, host string) error {
	request := fmt.Sprintf("GET / HTTP/1.1\r\nHOST: %s\r\n\r\n", host)
	_, err := conn.Write([]byte(request))
	if err != nil {
		return fmt.Errorf("发送请求失败%v", err)
	}
	fmt.Println("请求已发送")
	fmt.Println(request)
	return nil
}
func GetRespond(conn net.Conn) ([]byte, error) {
	buffer := make([]byte, 4096)
	var response []byte
	for {
		// 读取数据
		n, err := conn.Read(buffer)
		if n > 0 {
			response = append(response, buffer[:n]...)
		}
		if err == io.EOF {
			break
		}
		if err != nil {
			return nil, fmt.Errorf("读取响应失败: %v", err)
		}
	}
	return response, nil
}

func GetAnswer(r []byte) string {
	for k, _ := range r {
		if r[k] == 'f' && r[k+1] == 'l' && r[k+2] == 'a' && r[k+3] == 'g' {
			k += 4
			ans := "NSSCTF"
			for r[k] != '}' {
				ans = fmt.Sprintf("%s%c", ans, r[k])
				k++
			}
			ans = fmt.Sprintf("%s}", ans)
			return ans
		}
	}
	return ""
}

func main() {
	conn, err := net.Dial("tcp", host)
	if err != nil {
		fmt.Println("无法连接:", err)
		return
	}
	defer conn.Close()
	err = SendRequest(conn, host)
	if err != nil {
		log.Fatalf("发送请求出错: %v", err)
	}
	request, err := GetRespond(conn)
	if err != nil {
		log.Fatalf("读取时相应出错 %v", err)
	}
	flag := GetAnswer(request)
	fmt.Println(flag)
}
