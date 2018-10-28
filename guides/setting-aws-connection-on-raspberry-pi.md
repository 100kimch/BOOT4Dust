# Setting AWS connection on Raspberry Pi

## Install AWS-CLI

See: [AWS Command Line Interface 설치](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/installing.html) / [명령줄 경로에 AWS CLI 실행 파일 추가](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/awscli-install-linux.html#awscli-install-linux-path)

Logs:

```bash
$ pip install awscli --upgrade --user
$ which python
/usr/bin/python
$ ls -al /usr/bin/python
lrwxrwxrwx 1 root root 9 Jan 24  2017 /usr/bin/python -> python2.7
$ nano ~/.profile
```

```text
export PATH=~/.local/bin:$PATH
```

```bash
$ source ~/.profile
```

{% hint style="info" %}
$ means a writable line in Bash\(Terminal\). type the commands just after $ signs.

Likewise, \# means a writable line in "bluetooth" program. type the commands just after \* signs.
{% endhint %}

## AWS-CLI Configuration

See: [AWS CLI 구성](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-getting-started.html) 

























