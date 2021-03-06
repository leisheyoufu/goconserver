## goconserver

`goconserver` is written in golang which intend to replace the `conserver`
which used in [xcat2](https://github.com/xcat2/xcat-core). The microservice
based design makes it easy to integrate with other tool which hope to log the
terminal sessions. It can also work independently through command line or rest
api interface.

![preview](/goconserver.gif)

## Key Features

- Manage the lifecycle of seesion hosts via REST or BULK REST interface.
- Interface based desgin, support multiple types of terminal, storage and
  output plugin. Multiple output plugins could work together.
- Multiple client could share one host session.

### Terminal plugins

- ssh: SSH driver start ssh session within goruntine, As no external process,
       goconserver could support a large number of OpenBMC
       [openbmc](https://github.com/openbmc) consoles with high performance.

- cmd: A general driver to help redirect the command input and output. Any
       shell based terminal type could be supported.

### Output plugins:

- file: Store the terminal session in files for different hosts.
- tcp:  Send the console line (splitted) in json format to the remote target
        with tcp method.
- udp:  Send the console line (splitted) in json format to the remote target
        with udp method.

### Storage plugins

- file: Store the host information in a json file.
- etcd: Support goconserver cluster [experimental].

### Design Structure
`goconserver` can be divided into two parts:
- daemon part: `goconserver`, expose rest api interface to define and control
  the session node.

- client part: `congo`, a command line tool to define session or connect to the
  session. Multiple client sessions could be shared.

## Setup goconserver from release

### Setup goconserver from binary
Download the binary tarball for release from
[goconserver](https://github.com/chenglch/goconserver/releases)
```
tar xvfz goconserver_linux_amd64.tar.gz
cd goconserver_linux_amd64
./setup.sh
```

Modify the congiguration file `/etc/goconserver/server.conf` based on your
environment, then run `goconserver` to start the daemon service. To support a
large amount of sessions, please use `ulimit -n <number>` command to set the
number of open files.
```
goconserver [--congi-file <file>]
```

Modify the the environment variables in `/etc/profile.d/congo.sh` based on your
environment, then try the `congo` command.
```
source /etc/profile.d/congo.sh
congo list
```
### Setup goconserver from rpm or deb

```
tar xvfz <tarball for rpm or deb>
yum install <rpm>
dpkg -i <deb>
```

## Development

### Requirement

Please setup golang SDK(1.9 or higher), GOPATH environment variable and
[glide](https://github.com/Masterminds/glide) tool at first.

### Build and install

```
git clone https://github.com/chenglch/goconserver.git
cd goconserver
make deps
make install
```

### Setup SSL (optional)

Please refer to [ssl](/scripts/ssl/)

## Command Example

### Start service
```
goconserver &                     # for debug
service goconserver start         # only support systemd system
```
### Define testnode node session
congo is the client command. Use congo help to see the detail.
```
congo create testnode driver=ssh ondemand=false --params user=root,host=10.5.102.73,port=22,password=<password>
```
or with ssh private key
```
congo create testnode driver=ssh ondemand=false --params user=root,host=10.5.102.73,port=22,private_key=<priavte_key_path>
```
or general command driver
```
congo create testnode driver=cmd ondemand=false --params cmd="ssh -l root -p 22 10.5.102.73"
```

### List or show detail
```
congo list
congo show testnode
```

### Connect to the testnode session
```
congo console testnode
```

## Rest API

Rest api support bulk interface to manage the console sessions.
Please refer to [rest api](/api/) for detail.
