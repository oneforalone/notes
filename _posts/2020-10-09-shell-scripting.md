---
title: Command Line and Shell Scripting Bible
categories: [Linux, Shell]
---

## Section One: Command Line

### command grouping

- (cmd1; cmd2; ...; cmdn): start a sub-shell executing the commands

- {cmd1; cmd2; ...; cmdn}: executing commands in current shell

#### background (what's `coproc`)

   notation `&`, `jobs` to show background jobs

   `coproc`: do two things at the same time, execute the command after genarated a sub-shell in the background


#### built-in commands V.S external commands

- built-in: part of shell

- external: not part of shell, usually in =/bin, /usr/bin, /sbin or /usr/sbin

There would be a sub-process forking while executing external commands.

Built-in commands works more effectively than external.

### Environment Variable

- global variable

  take efforts to all shell session and sub-shell. To get global variable, using `env` or `printenv`.

- local variable

  only efforts in the defined process. **set** will display all variable, including local variable, global variable and user defined variable.

- define a variable

  * `VAR_NAME=value`: not spaces or tabs allowed beside `=`.
  * `export VAR_NAME`: make variable global.
  * `echo $VAR_NAME`: get value by variable name.
  * `unset VAR_NAME`: remove value of the variable.

- shell configure file order

  1. /etc/profile (global configure file)
  2. $HOME/.bash_profile
  3. $HOME/.bashrc
  4. $HOME/.bash_login
  5. $HOME/.profile

- arrays

  ```sh
  $ $my_arr=(one two three four five)
  $ echo $mytest
  one
  $ echo ${my_arr[2]}
  three
  $ echo ${my_arr[*]}
  one two three four five
  ```

### File Permission

#### File type

- -: regular file
- d: directory
- l: link file
- c: character device
- b: block device
- n: network device

#### Permission

- r: readable (4)
- w: writeable (2)
- x: executable (1)
- -: no permission (0)

777: stands for readable, writeable and executable
umask: the true file permission is: 777 - umask

**chmod**: changing file permission
  chmod [ugoa...][[-+=][perms...]...], where perms is either zero or more letters from the set **rwxXst**, or a single letter from the set **ugo**.

  + u: the user who owns it
  + g: the file's group
  + o: other users not in the file's group
  + a: all users

advanced **perms**: file has owner and group, and so did a process from an executable file. Usually use for file sharing:

  + S(suid,4): Set User ID;
  + G(sgid,2): Set Group ID;
  + T(sticky,1): sticky;

more details: `man chmod`

## Section Two: Shell Scripting

### structured command

#### `if-then` clause

syntax:
```sh
if ~command~
then
  ~commands~
fi
```
or:
```sh
if ~command~; then
  ~commands~
fi
```

#### `if-then-else` clause

syntax:
```sh
if ~command~
then
  ~command~
else
  ~commands~
fi
```

#### nested `if`

syntax:
```sh
if ~command~
then
  ~command set 1~
elif ~command2~
then
  ~command set 2~
elif ~command 3~
then
  ~command set 3~
elif ~commnad 4~
then
  ~command set 4~
fi
  ```

#### `test` clause

syntax:
  `test ~condition~`
looks like:
  ```sh
  if test ~condition~
  then
    ~commands~
  fi
  ```

##### condition type:

- number compare

  | compare   |     | description  |
  | :---      | --- | :---         |
  | n1 -eq n2 | | equal            |
  | n1 -ge n2 | | greater or equal |
  | n1 -gt n2 | | greater than     |
  | n1 -le n2 | | less or equal    |
  | n1 -lt n2 | | less than        |
  | n1 -ne n2 | | not equal        |

- string compare

  | compare      |     | description                    |
  | :---         | --- | :---                           |
  | str1 = str2  | | equal                              |
  | str1 != str2 | | not equal                          |
  | str1 < str2  | | less than (ASCII based)            |
  | str1 > str2  | | greater than (ASCII based)         |
  | -n str1      | | not empty (i.e length not equal 0) |
  | -z str1      | | empty (i.e length equals 0)        |

  **Caution**: symbol `<` `>` should distincted with redirection, so, should add backslash(`\`) to translate: `\<` `\>`

- file compare

  | compare         |     | description                           |
  | :---            | --- | :---                                  |
  | -d file         | | is directory                              |
  | -e file         | | is exist                                  |
  | -f file         | | is exist and a file                       |
  | -r file         | | is readable                               |
  | -s file         | | is exist and not empty                    |
  | -w file         | | is writeable                              |
  | -x file         | | is executable                             |
  | -O file         | | is exist and belongs to current user      |
  | -G file         | | is exist and same as current user's group |
  | file1 -nt file2 | | is file1 newer than file2                 |
  | file1 -ot file2 | | is file1 older than file2                 |

- advanced feature of `if-then`

  * (( expression ))

    | notations    |     | description   |
    | :---         | --- | :---          |
    | val++        | | val = val + 1     |
    | val--        | | val = val - 1     |
    | ++val        | | val + 1; val      |
    | --val        | | val - 1; val      |
    | *!*          | | not (logical)     |
    | *~*          | | not (bit)         |
    | **           | | power operation   |
    | <<           | | left shift (bit)  |
    | >>           | | right shift (bit) |
    | &            | | and (bool)        |
    | (shift + \)  | | or (bool)         |
    | &&           | | and (logical)     |
    | (shift + \\) | | or (logical       |

  * double bracket(`[[ expression ]]`): pattern matching

#### `case` clause

syntax:
```sh
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

#### `for` clause

syntax:
```sh
for var in list
do
  commands
done
```

C style for:
```C
for (( variable assignment ; condition ; iteration process ))
do
  commands
done
```

#### `while` clause

syntax:
```sh
while test command
do
  other commands
done
```

#### `until` clause

syntax:
```sh
until test commands
do
  other commands
done
```

oppsite with **while**

#### set the delimiter (IFS variable)

the default delimiter of bash shell:

* space
* tabulator
* newline

e.g. adding multiple account

  ```sh
  $ cat add_mul_users.sh
  #!/bin/bash
  # process new user accounts

  input="users.csv"
  while IFS=',' read -r userid name
  do
    echo "adding $userid"
    useradd -c "$name" -m $userid
  done < "$input"

  $ cat users.csv
  rich,Richard Blum
  christine,Christine Bresnahan
  barbara,Barbara Blumtim
  tim,Timothy Bresnahan
  $
  ```

### User Input

#### command line arguments
```
$0: program name
$1: the first argument
$2: the second argument
...
$9: the ninth argument
$#: amount of arguments

$*: store all arguments as a single word
$@: store all arguments as mutiple independent words
```
  e.g

```sh
$ cat test.sh
#!/bin/bash
# testing $* and $@
#
echo
count=1
#
for param in "$*"
do
  echo "\$* parameter #$count = $param"
  count=$[ $count + 1 ]
done
#
echo
count=1
#
for param in "$@"
do
  echo "\$@ parameter #$count = $param"
  count=$[ $count + 1 ]
done
$
$ ./test.sh rich barbara katie jessica

$* Parameter #1 = rich barbara katie jessica

$@ Parameter #1 rich
$@ Parameter #2 barbara
$@ Parameter #3 katie
$@ Parameter #4 jessica
$
```

#### moving arguments (`shift`)

when the number of arguments unknown, it's better to use `shift` to go through the arguments list

```sh
$ cat test.sh
#!/bin/bash# demonstrating the shift command
echo
count=1
while [ -n "$1" ]
do
  echo "Parameter #$count = $1"
  count=$[ $count + 1 ]
  shift
done
$
$ ./test.sh rich barbara katie jessica

Parameter #1 = rich
Parameter #2 = barbara
Parameter #3 = katie
Parameter #4 = jessica
$
```

#### deal with options

- normal one

  ```sh
  $ cat test.sh
  #!/bin/bash
  # extracting command line options as parameters
  #
  echo
  while [ -n "$1" ]
  do
    case "$1" in
      -a) echo "Found the -a option" ;;
      -b) echo "Found the -b option" ;;
      -c) echo "Found the -c option" ;;
      *) echo "$1 is not an option" ;;
    esac
    shift
  done
  $
  $ ./test.sh -a -b -c -d

  Found the -a option
  Found the -b option
  Found the -c option
  -d is not an option
  $
  ```

- split arguments and option ( `--` )

  ```sh
  $ cat test.sh
  #!/bin/bash
  # extracting options and parameters
  echo
  while [ -n "$1" ]
  do
    case "$1" in
      -a) echo "Found the -a option" ;;
      -b) echo "Found the -b option" ;;
      -c) echo "Found the -c option" ;;
      --) shift
          break ;;
      *) echo "$1 is not an option" ;;
    esac
    shift
  done
  #
  count=1
  for param in $@
  do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
  done
  $
  $ ./test.sh -c -a -b test1 test2 test3

  Found the -c option
  Found the -a option
  Found the -b option
  test1 is not an option
  test2 is not an option
  test3 is not an option
  $
  $ ./test.sh -c -a -b -- test1 test2 test3

  Found the -c option
  Found the -a option
  Found the -b option
  Parameter #1: test1
  Parameter #2: test2
  Parameter #3: test3
  $
  ```

- options with value

  ```sh
  $ cat test.sh
  #!/bin/bash
  # extracting command line options and values
  echo
  while [ -n "$1" ]
  do
    case "$1" in
      -a) echo "Found the -a option" ;;
      -b) param="$2"
          echo "Found the -b option, with parameter value $param"
          shift ;;
      -c) echo "Found the -c option" ;;
      --) shift
          break;
      *) echo "$1 is not an option" ;;
    esac
    shift
  done
  #
  count=1
  for param in "$@"
  do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
  done
  $
  $ ./test.sh -a -b test1 -d

  Found the -a option
  Found the -b option, with parameter value test1
  -d is not an option
  $
  ```

- `getopt` V.S `getopts`

  syntax:
    `getopt optstring parameters`
  to avoid error, using `-q` option.
  If an option need take a value, add a colon after it like this:

  ```sh
  $ getopt ab:cd -a -b test1 -cd test2 test3
  -a -b test1 -c -d -- test2 test3
  $
  $ cat test.sh
  #!/bin/bash
  # Extract command line options & values with getopt
  #
  set -- $(getopt -q ab:cd "$@")
  #
  echo
  while [ -n "$1" ]
  do
    case "$1" in
      -a) echo "Found the -a option" ;;
      -b) param="$2"
          echo "Found the -b option, with parameter value $param"
          shift ;;
      -c) echo "Found the -c option" ;;
      --) shift
          break;
      *) echo "$1 is not an option" ;;
    esac
    shift
  done
  #
  count=1
  for param in "$@"
  do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
  done
  #
  $ ./test.sh -ac

  Found the -a option
  Found the -c option
  $ ./test.sh -a -b test1 -cd test2 test3 test4

  Found the -a option
  Found the -b option, with parameter value 'test1'
  Found the -c option
  Parameter #1: 'test2'
  Parameter #2: 'test3'
  Parameter #3: 'test4'
  $ ./test.sh -a -b test1 -cd "test2 test3" test4 # can not deal space in quota
  Found the -a option
  Found the -b option, with parameter value 'test1'
  Found the -c option
  Parameter #1: 'test2
  Parameter #2: test3'
  Parameter #3: 'test4'
  $
  $ cat test_getopts.sh
  #!/bin/bash
  # simple demonstrating of the getopts command
  #
  echo
  while getops :ab:c opt
  do
    case "$opt" in
      a) echo "Found the -a option" ;;
      b) echo "Found the -b option, with value $OPTARG" ;;
      c) echo "Found the -c option" ;;
      *) echo "Unknown option: $opt" ;;
    esac
  done
  $
  $ ./test_getopts.sh -ab test1 -c

  Found the -a option
  Found the -b option, with value test1
  Found the -c option
  $
  $ ./test_getopts.sh -b "test1 test2" -a

  Found the -b option, with value test1 test2
  Found the -a option
  $ ./test_getopts.sh -abtest1

  Found the -a option
  Found the -b option, with value test1
  $ ./test_getopts.sh -d

  Unknown option: ?
  $
  $ ./test_getopts.sh -acde

  Found the -a option
  Found the -c option
  Unknown option: ?
  Unknown option: ?
  $
  ```

  **OPTIND** varaible store the amount of arguments, after dealing an option,
  it with increse itself.

- Standard Options

  | options |     | description                 |
  | :---    | --- | :---                        |
  | -a      | | show all objects                |
  | -c      | | genarate a counter              |
  | -d      | | specify the directory           |
  | -e      | | extend an object                |
  | -f      | | specify the input file          |
  | -h      | | display help info               |
  | -i      | | ignore case-sensetive           |
  | -l      | | display long info               |
  | -n      | | non-interactive (batch script)  |
  | -o      | | specify output file             |
  | -q      | | quiet mode                      |
  | -r      | | recursive                       |
  | -s      | | slient mode ( familiar with -q) |
  | -v      | | show details                    |
  | -x      | | exclude the object              |
  | -y      | | yes for all questions           |


- obtain user input: `read`

  options:
  -p: promote messages
  -t: timeout

  read from file.

  ```sh
  cat filename | while read line
  do
    ...
  ```



#### Input and Output

- 0: STDIN, standard input in shell
- 1: STDOUT, standard output in shell
- 2: STDERR, standard error in shell

file descriptor limited to 9 in shell, redirect all: `exec 2> info.err`, close file
descriptor: `exec 3>&-`

read and write to a file.

```sh
$
$ cat test.sh
#!/bin/bash
# testing input/output file descriptor

exec 3<> testfile
read line <&3
echo "Read: $line"
echo "This is a test line" >&3
$ cat testfile
This is the first line.
This is the second line.
This is the third line.
$ ./test.sh
Read: This is the first line.
$ cat testfile
This is the first line.
This is a test line
ine.
This is the third line.
$
```

`mktemp`: making local temporate file.

```sh
$ mktemp testing.XXXXXX
testing.1qwUjj
$ ls -al testing*
-rw------- 1 yuqi yuqi 0 Jul 24 20:11 testing.1qwUjj
$
```

`tee`: send the output to the display as well as logfile.

```sh
$ date | tee testfile
Fri 24 Jul 2020 08:14:51 PM CST
$ cat testfile
Fri 24 Jul 2020 08:14:51 PM CST
$ date | tee -a testfile # append the output to testfile
Fri 24 Jul 2020 08:16:51 PM CST
```

#### Shell Control

- `trap`: capture the signal. syntax: `trap commands signals`

- `nohup`: block all `SIGHUP` to process, so that process will keep running when console is closed

- `nice`: change priority, -n to specify the priority, run with command

- `renice`: change process priority

- `at`: Delayed job execution and batch processing. syntax: `at [-f filename] time`

- `atq`: at job queue

- `atrm`: remove job from job queue

- `cron`: daemon to execute scheduled commands

## Section Three: Advance Shell Scripting

### Function

syntax:
```sh
function name {
  commands
}
```

or

```sh
name() {
commands
}
```

- `local`: declare a local variable
- `source (dot operator)`: load other shell script file

### GUI

```sh
#!/bin/bash
# simple script menu

function diskspace {
  clear
  df -k
}

function whoseon {
  clear
  who
}

function memusage {
  clear
  cat /proc/meminfo
}

function menu {
  clear
  echo
  echo -e "\t\t\tSys Admin Menu\n"
  echo -e "\t1. Display disk space"
  echo -e "\t2. Display logged on users"
  echo -e "\t3. Display memory usage"
  echo -e "\t0. Exit program\n\n"
  echo -en "\t\tEnter option: "
  read -n 1 option
}

while [ 1 ]
do
  menu
  case $option in
  0)
    break ;;
  1)
    diskspace ;;
  2)
    whoseon ;;
  3)
    memusage ;;
  *)
    clear
    echo "Sorry, wrong selection" ;;
  esac
  echo -en "\n\n\t\t\tHit any key to continue"
  read -n 1 line
done
clear
```

`select` example:

```sh
#!/bin/bash
# using select in the menu

function diskspace {
  clear
  df -k
}

function whoseon {
  clear
  who
}

function memusage {
  clear
  cat /proc/meminfo
}

PS3="Enter option: "
select option in "Display disk space" "Display logged on users"\
                 "Display memory usage" "Exit program"
do
  case $option in
  "Exit program")
    break ;;
  "Display disk space")
    diskspace ;;
  "Display logged on users")
    whoseon ;;
  "Display memory usage")
    memusage ;;
  *)
    clear
    echo "Sorry, wrong option" ;;
  esac
done
clear
```

#### `dialog` package

syntax: `dialog --widget parameters`

- `msgbox`

  syntax: `dialog --msgbox text height width`

- `yesno`

  ```sh
  $ dialog --title "Please answer" --yesno "Is this thing on?" 10 20
  $ echo $? # yes for 0, no for 1
  0
  $
  ```

GNOME: `gdialog`, `zenity`

### Text process command: `sed` and `awk`

#### `sed`

syntax: `sed options script file`

`sed '/pattern/command' filename`

e.g.

```sh
$ grep yuqi /etc/passwd
yuqi:x:1000:1000:Yuqi Liu,,,:/home/yuqi:/bin/bash
$ sed '/yuqi/s/bash/csh/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
[...]
yuqi:x:1000:1000:Yuqi Liu,,,:/home/yuqi:/bin/csh
$
```

#### `awk`

syntax: `awk options program file`
