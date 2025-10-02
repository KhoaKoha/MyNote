# Connect

## **TTY Shell**

#### Basic way

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
(inside the nc session) CTRL+Z
stty size
stty raw -echo; fg
reset
export TERM=xterm
export SHELL=/bin/bash
```

#### Alternative way: [Full-TTYs](https://hacktricks.boitatech.com.br/shells/shells/full-ttys)
