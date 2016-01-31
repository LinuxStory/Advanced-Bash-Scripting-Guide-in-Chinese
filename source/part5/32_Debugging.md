# 32 调试
<blockquote class="blockquote-center">
调试代码要比写代码困难两倍。因此，你写代码时越多的使用奇技淫巧（自做聪明），顾名思义，你越难以调试它。    --Brian Kernighan
</blockquote>

Bash shell中不包含内置的debug工具，甚至没有调试专用的命令和结构。当调试非功能脚本，产生语法错误或者有错别字时，往往是无用的错误提示消息。

例子 32-1. 一个错误脚本

```
#!/bin/bash
# ex74.sh

# 这是一个错误脚本，但是它错在哪？

a=37

if [$a -gt 27 ]
then
  echo $a
fi  

exit $?   # 0! 为什么?
```
脚本的输出:

```
./ex74.sh: [37: command not found
```
上边的脚本究竟哪错了(提示: 注意if的后边)

例子 32-2. 缺少关键字

```
#!/bin/bash
# missing-keyword.sh
# 这个脚本会提示什么错误信息？

for a in 1 2 3
do
  echo "$a"
# done     #所需关键字'done'在第8行被注释掉.

exit 0     # 将不会在这退出!

#在命令行执行完此脚本后
输入：echo $?    
输出：2
```

脚本的输出:

```
missing-keyword.sh: line 10: syntax error: unexpected end of file
```
注意, 其实不必参考错误信息中指出的错误行号. 这行只不过是Bash解释器最终认定错误的地方.
出错信息在报告产生语法错误的行号时, 可能会忽略脚本的注释行.
如果脚本可以执行, 但并不如你所期望的那样工作, 怎么办? 通常情况下, 这都是由常见的逻辑错误所
产生的.

例子 32-3.

```
#!/bin/bash

#  这个脚本应该删除在当前目录下所有文件名中含有空格的文件
#  它不能正常运行，为什么？

badname=`ls | grep ' '`

# Try this:
# echo "$badname"

rm "$badname"

exit 0
```

可以通过把echo "$badname"行的注释符去掉，找出例子 29-3中的错误， 看一下echo出来的信息，是否按你期望的方式运行.

在这种特殊的情况下，rm "$badname"不能得到预期的结果，因为$badname不应该加双引号。加上双引号会让rm只有一个参数(这就只能匹配一个文件名).一种不完善的解决办法是去掉$badname外 面的引号, 并且重新设置$IFS, 让$IFS只包含一个换行符, IFS=$'\n'. 但是, 下面这个方法更简单.

```
# 删除包含空格的文件的正确方法.
rm *\ *
rm *" "*
rm *' '*
# 感谢. S.C.
```

总结一下这个问题脚本的症状:
>1. 由于"syntax error"(语法错误)使得脚本停止运行,2. 或者脚本能够运行, 但是并不是按照我们所期望的那样运行(逻辑错误). 
3. 脚本能够按照我们所期望的那样运行, 但是有烦人的副作用(逻辑炸弹).


如果想调试脚本, 可以用以下方式:

1. echo语句可以放在脚本中存在疑问的位置上, 观察变量的值, 来了解脚本运行时的情况.

	```
	### debecho (debug-echo), by Stefano Falsetto ###
	### Will echo passed parameters only if DEBUG is set to a value. ###
	debecho () {
  		if [ ! -z "$DEBUG" ]; then
     		echo "$1" >&2
     		# ^^^ to stderr
  		fi
	}

	DEBUG=on
	Whatever=whatnot
	debecho $Whatever   # whatnot
	
	DEBUG=
	Whatever=notwhat
	debecho $Whatever   # (Will not echo.)
	```


2. 使用过滤器tee来检查临界点上的进程或数据流.
3. 设置选项-n -v -x

	sh -n scriptname不会运行脚本, 只会检查脚本的语法错误. 这等价于把set -n或set -o noexec插入脚本中. 注意, 某些类型的语法错误不会被这种方式检查出来.
		sh -v scriptname将会在运行脚本之前, 打印出每一个命令. 这等价于把set -v或set -o verbose插入到脚本中.
		选项-n和-v可以同时使用. sh -nv scriptname将会给出详细的语法检查.
		sh -x scriptname会打印出每个命令执行的结果, 但只使用缩写形式. 这等价于在脚本中插入set
-x或set -o xtrace.

	把set -u或set -o nounset插入到脚本中, 并运行它, 就会在每个试图使用未声明变量的地方给出一个unbound variable错误信息.

	```
	set -u   # Or   set -o nounset
	
	# Setting a variable to null will not trigger the error/abort.
	# unset_var=
	
	echo $unset_var   # Unset (and undeclared) variable.
	echo "Should not echo!"
	
	#sh t2.sh
	#t2.sh: line 6: unset_var: unbound variable
	```
4. 使用“断言”功能在脚本的关键点进行测试的变量或条件。 （这是从C借来的一个想法）

	Example 32-4. Testing a condition with an assert
	
	```
	#!/bin/bash
	# assert.sh
	
	#######################################################################
	assert ()                 #  If condition false,
	{                         #+ exit from script
	                          #+ with appropriate error message.
	  E_PARAM_ERR=98
	  E_ASSERT_FAILED=99
	
	
	  if [ -z "$2" ]          #  Not enough parameters passed
	  then                    #+ to assert() function.
	    return $E_PARAM_ERR   #  No damage done.
	  fi
	
	  lineno=$2
	
	  if [ ! $1 ] 
	  then
	    echo "Assertion failed:  \"$1\""
	    echo "File \"$0\", line $lineno"    # Give name of file and line number.
	    exit $E_ASSERT_FAILED
	  # else
	  #   return
	  #   and continue executing the script.
	  fi  
	} # Insert a similar assert() function into a script you need to debug.    
	#######################################################################
	
	
	a=5
	b=4
	condition="$a -lt $b"     #  Error message and exit from script.
	                          #  Try setting "condition" to something else
	                          #+ and see what happens.
	
	assert "$condition" $LINENO
	# The remainder of the script executes only if the "assert" does not fail.
	
	
	# Some commands.
	# Some more commands . . .
	echo "This statement echoes only if the \"assert\" does not fail."
	# . . .
	# More commands . . .
	
	exit $?
	```

5. 使用变量$LINENO和内建命令caller.

6. 捕获exit返回值.

	The exit command in a script triggers a signal 0, terminating the process, 	that is, the script itself. [1] It is often useful to trap the exit, forcing 	a "printout" of variables, for example. The trap must be the first command 	in the script.

捕获信号
	
trap
Specifies an action on receipt of a signal; also useful for debugging.
	
	
A signal is a message sent to a process, either by the kernel or another 	process, telling it to take some specified action (usually to terminate). 	For example, hitting a Control-C sends a user interrupt, an INT signal, to a 	running program.
	
A simple instance:
	
```
trap '' 2
# Ignore interrupt 2 (Control-C), with no action specified. 
	
trap 'echo "Control-C disabled."' 2
# Message when Control-C pressed.
```

Example 32-5. Trapping at exit

```
#!/bin/bash
# Hunting variables with a trap.

trap 'echo Variable Listing --- a = $a  b = $b' EXIT
#  EXIT is the name of the signal generated upon exit from a script.
#
#  The command specified by the "trap" doesn't execute until
#+ the appropriate signal is sent.

echo "This prints before the \"trap\" --"
echo "even though the script sees the \"trap\" first."
echo

a=39
b=36

exit 0


#  Note that commenting out the 'exit' command makes no difference,
#+ since the script exits in any case after running out of commands.
```

Example 32-6. Cleaning up after Control-C


```
#!/bin/bash
# logon.sh: A quick 'n dirty script to check whether you are on-line yet.

umask 177  # Make sure temp files are not world readable.


TRUE=1
LOGFILE=/var/log/messages
#  Note that $LOGFILE must be readable
#+ (as root, chmod 644 /var/log/messages).
TEMPFILE=temp.$$
#  Create a "unique" temp file name, using process id of the script.
#     Using 'mktemp' is an alternative.
#     For example:
#     TEMPFILE=`mktemp temp.XXXXXX`
KEYWORD=address
#  At logon, the line "remote IP address xxx.xxx.xxx.xxx"
#                      appended to /var/log/messages.
ONLINE=22
USER_INTERRUPT=13
CHECK_LINES=100
#  How many lines in log file to check.

trap 'rm -f $TEMPFILE; exit $USER_INTERRUPT' TERM INT
#  Cleans up the temp file if script interrupted by control-c.

echo

while [ $TRUE ]  #Endless loop.
do
  tail -n $CHECK_LINES $LOGFILE> $TEMPFILE
  #  Saves last 100 lines of system log file as temp file.
  #  Necessary, since newer kernels generate many log messages at log on.
  search=`grep $KEYWORD $TEMPFILE`
  #  Checks for presence of the "IP address" phrase,
  #+ indicating a successful logon.

  if [ ! -z "$search" ] #  Quotes necessary because of possible spaces.
  then
     echo "On-line"
     rm -f $TEMPFILE    #  Clean up temp file.
     exit $ONLINE
  else
     echo -n "."        #  The -n option to echo suppresses newline,
                        #+ so you get continuous rows of dots.
  fi

  sleep 1  
done  


#  Note: if you change the KEYWORD variable to "Exit",
#+ this script can be used while on-line
#+ to check for an unexpected logoff.

# Exercise: Change the script, per the above note,
#           and prettify it.

exit 0


# Nick Drage suggests an alternate method:

while true
  do ifconfig ppp0 | grep UP 1> /dev/null && echo "connected" && exit 0
  echo -n "."   # Prints dots (.....) until connected.
  sleep 2
done

# Problem: Hitting Control-C to terminate this process may be insufficient.
#+         (Dots may keep on echoing.)
# Exercise: Fix this.



# Stephane Chazelas has yet another alternative:

CHECK_INTERVAL=1

while ! tail -n 1 "$LOGFILE" | grep -q "$KEYWORD"
do echo -n .
   sleep $CHECK_INTERVAL
done
echo "On-line"

# Exercise: Discuss the relative strengths and weaknesses
#           of each of these various approaches.
Example 32-7. A Simple Implementation of a Progress Bar

#! /bin/bash
# progress-bar2.sh
# Author: Graham Ewart (with reformatting by ABS Guide author).
# Used in ABS Guide with permission (thanks!).

# Invoke this script with bash. It doesn't work with sh.

interval=1
long_interval=10

{
     trap "exit" SIGUSR1
     sleep $interval; sleep $interval
     while true
     do
       echo -n '.'     # Use dots.
       sleep $interval
     done; } &         # Start a progress bar as a background process.

pid=$!
trap "echo !; kill -USR1 $pid; wait $pid"  EXIT        # To handle ^C.

echo -n 'Long-running process '
sleep $long_interval
echo ' Finished!'

kill -USR1 $pid
wait $pid              # Stop the progress bar.
trap EXIT

exit $?

```
Note	
The DEBUG argument to trap causes a specified action to execute after every command in a script. This permits tracing variables, for example.

Example 32-8. Tracing a variable

```

#!/bin/bash

trap 'echo "VARIABLE-TRACE> \$variable = \"$variable\""' DEBUG
# Echoes the value of $variable after every command.

variable=29; line=$LINENO

echo "  Just initialized \$variable to $variable in line number $line."

let "variable *= 3"; line=$LINENO
echo "  Just multiplied \$variable by 3 in line number $line."

exit 0

#  The "trap 'command1 . . . command2 . . .' DEBUG" construct is
#+ more appropriate in the context of a complex script,
#+ where inserting multiple "echo $variable" statements might be
#+ awkward and time-consuming.

# Thanks, Stephane Chazelas for the pointer.
```

Output of script:

VARIABLE-TRACE> $variable = ""
VARIABLE-TRACE> $variable = "29"
  Just initialized $variable to 29.
VARIABLE-TRACE> $variable = "29"
VARIABLE-TRACE> $variable = "87"
  Just multiplied $variable by 3.
VARIABLE-TRACE> $variable = "87"
Of course, the trap command has other uses aside from debugging, such as disabling certain keystrokes within a script (see Example A-43).

Example 32-9. Running multiple processes (on an SMP box)

```

#!/bin/bash
# parent.sh
# Running multiple processes on an SMP box.
# Author: Tedman Eng

#  This is the first of two scripts,
#+ both of which must be present in the current working directory.




LIMIT=$1         # Total number of process to start
NUMPROC=4        # Number of concurrent threads (forks?)
PROCID=1         # Starting Process ID
echo "My PID is $$"

function start_thread() {
        if [ $PROCID -le $LIMIT ] ; then
                ./child.sh $PROCID&
                let "PROCID++"
        else
           echo "Limit reached."
           wait
           exit
        fi
}

while [ "$NUMPROC" -gt 0 ]; do
        start_thread;
        let "NUMPROC--"
done


while true
do

trap "start_thread" SIGRTMIN

done

exit 0



# ======== Second script follows ========


#!/bin/bash
# child.sh
# Running multiple processes on an SMP box.
# This script is called by parent.sh.
# Author: Tedman Eng

temp=$RANDOM
index=$1
shift
let "temp %= 5"
let "temp += 4"
echo "Starting $index  Time:$temp" "$@"
sleep ${temp}
echo "Ending $index"
kill -s SIGRTMIN $PPID

exit 0


# ======================= SCRIPT AUTHOR'S NOTES ======================= #
#  It's not completely bug free.
#  I ran it with limit = 500 and after the first few hundred iterations,
#+ one of the concurrent threads disappeared!
#  Not sure if this is collisions from trap signals or something else.
#  Once the trap is received, there's a brief moment while executing the
#+ trap handler but before the next trap is set.  During this time, it may
#+ be possible to miss a trap signal, thus miss spawning a child process.

#  No doubt someone may spot the bug and will be writing 
#+ . . . in the future.



# ===================================================================== #



# ----------------------------------------------------------------------#



#################################################################
# The following is the original script written by Vernia Damiano.
# Unfortunately, it doesn't work properly.
#################################################################

#!/bin/bash

#  Must call script with at least one integer parameter
#+ (number of concurrent processes).
#  All other parameters are passed through to the processes started.


INDICE=8        # Total number of process to start
TEMPO=5         # Maximum sleep time per process
E_BADARGS=65    # No arg(s) passed to script.

if [ $# -eq 0 ] # Check for at least one argument passed to script.
then
  echo "Usage: `basename $0` number_of_processes [passed params]"
  exit $E_BADARGS
fi

NUMPROC=$1              # Number of concurrent process
shift
PARAMETRI=( "$@" )      # Parameters of each process

function avvia() {
         local temp
         local index
         temp=$RANDOM
         index=$1
         shift
         let "temp %= $TEMPO"
         let "temp += 1"
         echo "Starting $index Time:$temp" "$@"
         sleep ${temp}
         echo "Ending $index"
         kill -s SIGRTMIN $$
}

function parti() {
         if [ $INDICE -gt 0 ] ; then
              avvia $INDICE "${PARAMETRI[@]}" &
                let "INDICE--"
         else
                trap : SIGRTMIN
         fi
}

trap parti SIGRTMIN

while [ "$NUMPROC" -gt 0 ]; do
         parti;
         let "NUMPROC--"
done

wait
trap - SIGRTMIN

exit $?

: <<SCRIPT_AUTHOR_COMMENTS
I had the need to run a program, with specified options, on a number of
different files, using a SMP machine. So I thought [I'd] keep running
a specified number of processes and start a new one each time . . . one
of these terminates.

The "wait" instruction does not help, since it waits for a given process
or *all* process started in background. So I wrote [this] bash script
that can do the job, using the "trap" instruction.
  --Vernia Damiano
SCRIPT_AUTHOR_COMMENTS

```

Note	
trap '' SIGNAL (two adjacent apostrophes) disables SIGNAL for the remainder of the script. trap SIGNAL restores the functioning of SIGNAL once more. This is useful to protect a critical portion of a script from an undesirable interrupt.

```
trap '' 2  # Signal 2 is Control-C, now disabled.
command
command
command
trap 2     # Reenables Control-C
```
	