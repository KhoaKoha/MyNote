# General

## HTTP Server <a href="#advance-http-server-supporting-upload-method" id="advance-http-server-supporting-upload-method"></a>

Host an HTTP server to serve files or allow file uploads from clients.

### **Simple HTTP Server**

Quickly serve files in the current directory over HTTP using Python.

```bash
python3 -m http.server <port>
```

### **Advanced HTTP Server**

Run a Python script to serve files and allow file uploads via HTTP PUT requests.

```python
import http.server
import socketserver

class SputHTTPRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_PUT(self):
        length = int(self.headers["Content-Length"])
        path = self.translate_path(self.path)
        with open(path, "wb") as dst:
            dst.write(self.rfile.read(length))
        # Send response
        response = b"Upload successful\n"
        self.send_response(200)
        self.send_header("Content-type", "text/plain")
        self.send_header("Content-Length", len(response))
        self.send_header("Connection", "close")
        self.end_headers()
        self.wfile.write(response)
        self.wfile.flush()

if __name__ == '__main__':
    PORT = 80 #CHANGE THIS
    try:
        with socketserver.TCPServer(("", PORT), SputHTTPRequestHandler) as httpd:
            print(f"Serving at port {PORT}")
            httpd.serve_forever()
    except KeyboardInterrupt:
        print("\nKeyboard interrupt received, exiting.")
```

### **Upload File to HTTP Server**

Upload a file to the advanced HTTP server using curl.

```bash
curl -T <file> http://<server_ip>:<port>/<filename>
```

* Example: `curl -T document.txt http://192.168.236.128:8000/document.txt`

## **SMB Server**

Host an SMB server to share files over a network, commonly used in Windows environments.

### **Host SMB Server**

Use impacket-smbserver to share a directory as an SMB share.

```bash
impacket-smbserver <share_name> <path> -smb2support
```

* Example: `impacket-smbserver SHARE . -smb2support`

### **Retrieve File from SMB Server**

Copy a file from the SMB server to the local machine.

```
copy \\<server_ip>\<share_name>\<file> <destination>
```

* Example: `copy \192.168.236.128\SHARE\file.txt .`
