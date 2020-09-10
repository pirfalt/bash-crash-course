# Bash Crash Course

TLDR:

- [devhints.io/bash]('https://devhints.io/bash') (my prefered cheat sheet)
- [shell check extention for vs code]('https://marketplace.visualstudio.com/items?itemName=timonwong.shellcheck')
- [shellcheck]('https://www.shellcheck.net/')
- [shellharden]('https://github.com/anordal/shellharden')

## What is an OS

- Communicate with hardware.
- Run programs.

Programs are the real work horse, they can do _anything_.

## What is `bash`?

- Text based OS interface.
  - With heavy focus on the filesystem for hardware communication.
- Start programs.
- Read and write files.
- REPL (Read Evaluate Print Loop).

## Why do we use it?

Your server/CICD has it. 🤖

Programs can compose in interesting ways. 💥

## Starting programs

Write the name of the program.

```bash
# Print your current Working Directory
pwd

# List directory contents
ls

# Print environment
env

# Manual
man

# Read as much as you can see from a file
less
```

Optionally pass some arguments, almost like a function call in a programming language.

```bash
ls -l
man ls
less --help
```

### The `PATH` and other "magic" places

Where programs are found.

```bash
# Show you mine
echo $PATH

# So, where is my program? _which_ program?
man which

which pwd
which ls
which env
/usr/bin/env
```

You are special. 🏠

```bash
echo $HOME
ls $HOME
ls ~
```

Others just require special help, from you. 🤷‍♂️

```bash
echo $JAVA_HOME
echo $GOPATH
```

## Pipes - input, output and "errors"

Inputs and outputs, all mixed up.

```bash
cat
```

Files, files and more files. From every (re)direction.

```bash
cat < readme.md
cat > output_cat.txt

cat readme.md
```

Markdown

```bash
npm install -g marked
which marked
marked -h
marked
```

```
# H1
And a paragraph
```

Common convention, word "flags" have double `--`, short "flags" (single letter) have one `-`.
(Common, but anything goes)

```bash
marked --input readme.md
marked --input readme.md --output output_space.html
marked --input=readme.md --output=output_equal.html
marked -i readme.md -o output_short.html

marked < readme.md
marked   < readme.md   > output_redirected.html

cat readme.md | marked > output_redirected2.html
```

Wait, it was supposed to be files, not the internet?

```bash
# Download from github
curl https://api.github.com/users/pirfalt

# Save to file
curl https://api.github.com/users/pirfalt > users_pirfalt.json

# Was that an error?
curl https://api.github.com/users/pirfalt 2> users_pirfalt_error.json

# Yes, you can multi redirect
curl https://api.github.com/users/pirfalt > users_pirfalt.json 2> users_pirfalt_error.json

# And long lines can be broken
curl https://api.github.com/users/pirfalt \
  > users_pirfalt.json \
  2> users_pirfalt_error.json
```

## The ~~appstore~~ programstore:s

Get the programs you don't have.

### Mac

```bash
brew
brew list
```

### Linux (ubuntu)

```bash
docker run -it --rm ubuntu:20.04
echo $SHELL
env
curl

apt --help
apt update
apt install curl
curl

exit
```

Also on "linux" `yum` (redhat, centos), `apk` (alpine) ...

### Windows

- https://docs.microsoft.com/en-us/windows/wsl/ 👌
  - Just like linux.
- https://chocolatey.org/
- `¯\_(ツ)\_/¯`

## Programs running other programs

Bash starting more bash

```bash
bash -c 'cd ~ ; ls'
#             ^   Thats new
```

Python

```bash
python -c 'import os; print(os.environ)'
```

NodeJS

```bash
node -e 'console.log(process.env)'
```

jq

```bash
cat users_pirfalt.json | jq '.'

cat users_pirfalt.json | jq '. | { login, location, company }'

cat users_pirfalt.json | jq '.repos_url'

man jq
cat users_pirfalt.json | jq -r '.repos_url'
curl $() # ??
```

```bash
cat readme.md | grep 'echo'
```

## Syntax

I still have to look it up, very often. [devhints.io/bash]('https://devhints.io/bash').

### Variables

```bash
variable_declaration_and_assignment="first"
echo "$variable_declaration_and_assignment"

single_quotes='includes: $variable_declaration_and_assignment'
echo "$single_quotes"

double_quotes="includes: $variable_declaration_and_assignment"
echo "$double_quotes"

no_quotes=includes:\ $variable_declaration_and_assignment
echo "$no_quotes"

# My own personal rule of thumb:
#  Always use single quotes. Until you need double quotes.
#  No quotes are problems waiting to happen.

# And use shellcheck/shellharden.
```

### Functions

```bash
curl -H 'Accept: application/vnd.github.v3+json' https://api.github.com/users/pirfalt

# Call github with the correct headers
call_github() {
  local args=$*
  curl -H 'Accept: application/vnd.github.v3+json' $args
}

call_github https://api.github.com/users/pirfalt

# Start crawling
find_repos_for_user() {
  local user="$1"

  local user_url="https://api.github.com/users/$user"
  local repo_url="$(call_github "$user_url" | jq -r '.repos_url')"
  call_github "$repo_url" | jq '.[] | [ .name, .html_url ]'
}

find_repos_for_user pirfalt

# Late input
find_repos_for_user_v2() {
  echo -n 'Please enter a username: '
  read user
  local user_url="https://api.github.com/users/$user"
  local repo_url="$(call_github "$user_url" | jq -r '.repos_url')"
  call_github "$repo_url" | jq '.[] | [ .name, .html_url ]'
}

find_repos_for_user_v2
```

### Branches

Please don't. `bash` _can_ do `if-then-else` logic. But it's messy. 🤷‍♂️

### Loops

```bash
ls | while read file_name; do echo "Found: $file_name in this directory"; done

for f in $(ls); do echo "Found: $f in this directory"; done

for c in 1 2 3 4; do echo "space separated $c"; done

while true; do ls; sleep 2; done
```
