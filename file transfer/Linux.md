# Linux

## Temporary File Locations

* `/tmp`: Standard temporary directory (e.g., `/tmp/document.txt`).
* `/dev/shm`: RAM-based temporary storage (e.g., `/dev/shm/document.txt`).

## Wget

Download a file from a remote server.

```bash
wget http://<target_ip>:<port>/<file> -o /<local_path>/<file>
```

* Example: `wget http://192.168.1.200:80/document.txt -o /tmp/document.txt`

## cURL

Download a file from a remote server.

```bash
curl http://<target_ip>:<port>/<file> -o /<local_path>/<file>
```

* Example: `curl http://192.168.1.200:80/document.txt -o /tmp/document.txt`

## Netcat

Transfer files using a TCP connection.

### Sender

Host a file for transfer.

```bash
nc -lvp <port> < /<local_path>/<file>
```

* Example: `nc -lvp 4444 < /tmp/document.txt`

### Receiver

Connect to the sender to receive the file.

```bash
nc -nv <target_ip> <port> > /<local_path>/<file>
```

* Example: `nc -nv 192.168.1.200 4444 > /tmp/document.txt`

## SCP

Securely copy files between local and remote systems.

### Copy Local File to Remote System

Transfer a file to a remote system.

```bash
scp /<local_path>/<file> <user>@<target_ip>:/<remote_path>
```

* Example: `scp /tmp/document.txt admin@192.168.1.200:/home/admin/`

### Copy Remote File to Local System

Transfer a file from a remote system.

```bash
scp <user>@<target_ip>:/<remote_path>/<file> /<local_path>
```

* Example: `scp admin@192.168.1.200:/home/admin/document.txt /tmp/`
