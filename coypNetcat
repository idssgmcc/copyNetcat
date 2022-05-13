import argparse #创建带命令行界面的程序
import socket
import shlex
import subprocess
import sys
import textwrap
import threading
socket.setdefaulttimeout(10)
def execute(cmd):       #创建函数execute\这个函数将会接受一条命令并执行、并返回字符串。
    cmd = cmd.strip()
    if not cmd:
        return
    output = subprocess.check_output(shlex.split(cmd),      #subprocess库提供进程创建接口，可通过多种方式调用其他程序，subprocess.check_output运行命令并返回输出
                                     stderr=subprocess.STDOUT)
    return output.decode()
#创建main代码块，用来解析命令参数并调用其他函数

#客户端代码
class NetCat:
    def __init__(self, args, buffer=None):  #main代码块传进来的命令行参数和缓冲区数据，初始化一个Netcat对象
        self.args = args
        self.buffer = buffer
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM) #创建一个socker对象
        self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    def run(self):
        if self.args.listen:
            self.listen()   #run函数作为netcat对象的执行口、逻辑为直接把后续的执行交给其他两个函数，如果netcat是接收方就run listen函数、如果是发送方就run send函数
        else:
            self.socket.connect((self.args.target, self.args.port)) #链接到ip和端口、如缓冲区有数据，先发数据
            
            if self.buffer:
                self.send(self.buffer)
            while True: 
                buffer = ''
                buffer = input('noob > ').strip()
                print(buffer.encode())
                self.send(buffer.encode())

    def send(self , buffer , timeout = 2):
        if buffer:
            self.socket.send(buffer)

        try:    #创建try/catch块，可以使用ctrl+c关闭连接
            # while True:     #创建大循环一轮轮接受target返回的数据，创建小循环来读取socket返回本轮数据
            recv_len = 1
            response = ''
            while recv_len:
                data = self.socket.recv(4096)
                recv_len = len(data)
                response += data.decode()
                if recv_len < 4096:
                    break   #如果socket里面读完了，就推出小循环

            if response:
                print(response)
                # buffer = input('noob > ')+ "\n"
                # # buffer += '\n'
                # self.socket.send(buffer.encode())   #检查是否还有新类容、输出并暂停，等待用户输入新内容、再发给target


        except KeyboardInterrupt:   #ctrl+c KeyboardInterrupt来中断循环，同时关闭socker对象
            print('User terminated.')
            self.socket.close()
            sys.exit()

    def listen(self):
        self.socket.bind((self.args.target, self.args.port))    #listen函数把socket对象绑定ip和端口
        self.socket.listen(5)
        while True:     #用循环监听新连接，把已连接的socket对象传递给handle函数
            client_scoket, _ = self.socket.accept()
            client_thread = threading.Thread(
                target=self.handle, args=(client_scoket,)
            )
            client_thread.start()

    def handle(self, client_socker):
        if self.args.execute:   #handle函数命令传递给execute，然后输出结果通过socket发送
            output = execute(self.args.execute)
            client_socker.send(output.encode())

        elif self.args.upload:  #上传文件建一个循环来接受socket传来的文件内容、再将收到的全部数据写道参数指定文件里
            file_buffer = b''
            while True:
                data = client_socker.recv(4096)
                if data:
                    file_buffer += data
                else:
                    break

            with open(self.args.upload, 'wb') as f:
                f.write(file_buffer)
            message = f'Saved file {self.args.upload}'
            client_socker.send(message.encode())

        elif self.args.command: #创建shell，循环发送提示符、发送命令、收到使用execute函数执行，返回结果
            cmd_buffer = b''
            while True:
                try:
                    client_socker.send(b'BHP: #> ')
                    while '\n' not in cmd_buffer.decode():
                        cmd_buffer += client_socker.recv(64)
                    response = execute(cmd_buffer.decode())
                    if response:
                        client_socker.send(response.encode())
                    cmd_buffer = b''
                except Exception as e:
                    print(f'server killed {e}')
                    self.socket.close()
                    sys.exit()
#接收端代码
if __name__ == '__main__':
    parser = argparse.ArgumentParser(
        description='BHP Net Tool',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog=textwrap.dedent('''Example:  #帮助信息
            netcat.py -t 192.168.3.8 -p 5555 -l -c # command shell
            netcat.py -t 192.168.3.8 -p 5555 -l -u=mytest.txt # upload to file
            netcat.py -t 192.168.3.8 -p 5555 -l -e=\"cat /etc/passwd\" # execute command
            echo 'ABC' | ./netcat.py -t 192.168.3.8 -p 135 # echo text to server port 135
            netcat.py -t 192.168.3.8 -p 5555 # connect to server   
        '''))
    parser.add_argument('-c', '--command', action='store_true', help='command shell')   #控制程序
    parser.add_argument('-e', '--execute', help='execute specified command')
    parser.add_argument('-l', '--listen', action='store_true', help='listen')
    parser.add_argument('-p', '--port', type=int, default=5555, help='command shell')
    parser.add_argument('-t', '--target', default='192.168.3.8', help='specified IP')
    parser.add_argument('-u', '--upload', help='upload file')
    args = parser.parse_args()
    if args.listen:
        buffer = ''
    else:
        buffer = None
    # elif len(sys.stdin):
    #     buffer = sys.stdin.read()

    if buffer :
        nc = NetCat(args, buffer.encode())
    else: nc  = NetCat(args)
    nc.run()
