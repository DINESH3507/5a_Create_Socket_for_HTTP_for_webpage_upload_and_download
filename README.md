# 5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
## AIM :
To write a PYTHON program for socket for HTTP for web page upload and download
## Algorithm

1.Start the program.
<BR>
2.Get the frame size from the user
<BR>
3.To create the frame based on the user request.
<BR>
4.To send frames to server from the client side.
<BR>
5.If your frames reach the server it will send ACK signal to client otherwise it will send NACK signal to client.
<BR>
6.Stop the program
<BR>
## Program 
Client.py
```
import socket
import os

def send_request(host, port, request_headers, body=b''):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))
        s.sendall(request_headers.encode() + body)
        response = b""
        while True:
            data = s.recv(4096)
            if not data:
                break
            response += data
    return response

def upload_file(host, port, filename):
    if not os.path.exists(filename):
        print(f"File '{filename}' not found.")
        return

    with open(filename, 'rb') as file:
        file_data = file.read()

    content_length = len(file_data)
    request_headers = (
        f"POST /upload HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Content-Length: {content_length}\r\n"
        f"Content-Type: application/octet-stream\r\n"
        f"Connection: close\r\n\r\n"
    )
    response = send_request(host, port, request_headers, body=file_data)
    return response.decode(errors='ignore')

def download_file(host, port, filename):
    request_headers = (
        f"GET /{filename} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Connection: close\r\n\r\n"
    )
    response = send_request(host, port, request_headers)
    if b"\r\n\r\n" not in response:
        print("Invalid response received.")
        return
    headers, body = response.split(b"\r\n\r\n", 1)
    with open(filename, 'wb') as file:
        file.write(body)
    print(f"File '{filename}' downloaded successfully.")

if __name__ == "__main__":
    host = 'localhost'
    port = 8080

    print("Uploading file...")
    upload_response = upload_file(host, port, 'example.txt')
    print("Upload response:\n", upload_response[:200], "\n")

    print("Downloading file...")
    download_file(host, port, 'example.txt')
```
Server.py
```
from flask import Flask, request, send_from_directory
import os

app = Flask(__name__)
UPLOAD_FOLDER = "./uploads"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route("/upload", methods=["POST"])
def upload():
    data = request.data
    filename = "example.txt"
    with open(os.path.join(UPLOAD_FOLDER, filename), "wb") as f:
        f.write(data)
    return "File uploaded successfully."

@app.route("/<path:filename>", methods=["GET"])
def download(filename):
    return send_from_directory(UPLOAD_FOLDER, filename)

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080)
```
## OUTPUT
<img width="1847" height="1070" alt="image" src="https://github.com/user-attachments/assets/cdcd7f3f-30c0-48b1-89f8-31462268a0c9" />

## Result
Thus the socket for HTTP for web page upload and download created and Executed
