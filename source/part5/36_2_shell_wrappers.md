# 36.2 shell wrappers

wrapper是一个包含系统命令和工具的脚本，脚本会把一些参数传递给这些（脚本内的）命令。将一个复杂的命令封装成一个wrapper是为了调用它时比较简单好记，特别在使用sed和awk命令时会这么做。

sed或awk脚本通常在命令行下调用时是sed -e '命令'或者awk '命令'。在Bash脚本中嵌入这些命令会让它们在调用时很简单，并且能够被重用。使用这种方法可以将sed和awk的优势统一起来，比如将sed命令处理的结果通过管道传递给awk继续处理。将这些保存成为一个可执行文件，你可以重复调用它的原始版本或者修改版本，而不用在命令行里反复敲冗长的命令。

Example 36-1. shell wrapper

```
#!/bin/bash

# 这个脚本功能是去除文件中的空白行
# 没有做参数检查
#
# 也许你想添加下面的内容：
#
# E_NOARGS=85
# if [ -z "$1" ]
# then
#  echo "Usage: `basename $0` target-file"
#  exit $E_NOARGS
# fi

sed -e /^$/d "$1"
# 就像这个命令
#    sed -e '/^$/d' filename
# 通过命令行调用

#  The '-e' means an "editing" command follows (optional here).
#  '^' indicates the beginning of line, '$' the end.
#  This matches lines with nothing between the beginning and the end --
#+ blank lines.
#  The 'd' is the delete command.

#  Quoting the command-line arg permits
#+ whitespace and special characters in the filename.

#  Note that this script doesn't actually change the target file.
#  If you need to do that, redirect its output.

exit
```

Example 36-2. A slightly more complex shell wrapper

```
#!/bin/bash

#  subst.sh: a script that substitutes one pattern for
#+ another in a file,
#+ i.e., "sh subst.sh Smith Jones letter.txt".
#                     Jones replaces Smith.

ARGS=3         # Script requires 3 arguments.
E_BADARGS=85   # Wrong number of arguments passed to script.

if [ $# -ne "$ARGS" ]
    then
      echo "Usage: `basename $0` old-pattern new-pattern filename"
        exit $E_BADARGS
        fi

        old_pattern=$1
        new_pattern=$2

        if [ -f "$3" ]
            then
                file_name=$3
                else
                        echo "File \"$3\" does not exist."
                            exit $E_BADARGS
                            fi


# -----------------------------------------------
#  Here is where the heavy work gets done.
sed -e "s/$old_pattern/$new_pattern/g" $file_name
# -----------------------------------------------

#  's' is, of course, the substitute command in sed,
#+ and /pattern/ invokes address matching.
#  The 'g,' or global flag causes substitution for EVERY
#+ occurence of $old_pattern on each line, not just the first.
#  Read the 'sed' docs for an in-depth explanation.

exit $?  # Redirect the output of this script to write to a file.
```

Example 36-3. A generic shell wrapper that writes to a logfile

```
#!/bin/bash
#  logging-wrapper.sh
#  Generic shell wrapper that performs an operation
#+ and logs it.

DEFAULT_LOGFILE=logfile.txt

# Set the following two variables.
OPERATION=
#         Can be a complex chain of commands,
#+        for example an awk script or a pipe . . .

LOGFILE=
if [ -z "$LOGFILE" ]
    then     # If not set, default to ...
      LOGFILE="$DEFAULT_LOGFILE"
      fi

#         Command-line arguments, if any, for the operation.
OPTIONS="$@"


# Log it.
echo "`date` + `whoami` + $OPERATION "$@"" >> $LOGFILE
# Now, do it.
exec $OPERATION "$@"

# It's necessary to do the logging before the operation.
# Why?
```

Example 36-4. A shell wrapper around an awk script

```
#!/bin/bash
# pr-ascii.sh: Prints a table of ASCII characters.

START=33   # Range of printable ASCII characters (decimal).
END=127    # Will not work for unprintable characters (> 127).

echo " Decimal   Hex     Character"   # Header.
echo " -------   ---     ---------"

for ((i=START; i<=END; i++))
    do
          echo $i | awk '{printf("  %3d       %2x         %c\n", $1, $1, $1)}'
# The Bash printf builtin will not work in this context:
#     printf "%c" "$i"
          done

          exit 0


#  Decimal   Hex     Character
#  -------   ---     ---------
#    33       21         !
#    34       22         "
#    35       23         #
#    36       24         $
#
#    . . .
#
#   122       7a         z
#   123       7b         {
#   124       7c         |
#   125       7d         }


#  Redirect the output of this script to a file
#+ or pipe it to "more":  sh pr-asc.sh | more
```

Example 36-5. A shell wrapper around another awk script

```
#!/bin/bash

# Adds up a specified column (of numbers) in the target file.
# Floating-point (decimal) numbers okay, because awk can handle them.

ARGS=2
E_WRONGARGS=85

if [ $# -ne "$ARGS" ] # Check for proper number of command-line args.
    then
       echo "Usage: `basename $0` filename column-number"
          exit $E_WRONGARGS
          fi

          filename=$1
          column_number=$2

#  Passing shell variables to the awk part of the script is a bit tricky.
#  One method is to strong-quote the Bash-script variable
#+ within the awk script.
#     $'$BASH_SCRIPT_VAR'
#      ^                ^
#  This is done in the embedded awk script below.
#  See the awk documentation for more details.

# A multi-line awk script is here invoked by
#   awk '
#   ...
#   ...
#   ...
#   '


# Begin awk script.
# -----------------------------
awk '

{ total += $'"${column_number}"'
}
END {
         print total
}     

' "$filename"
# -----------------------------
# End awk script.


#   It may not be safe to pass shell variables to an embedded awk script,
#+  so Stephane Chazelas proposes the following alternative:
#   ---------------------------------------
#   awk -v column_number="$column_number" '
#   { total += $column_number
#   }
#   END {
#       print total
#   }' "$filename"
#   ---------------------------------------


exit 0
```

For those scripts needing a single do-it-all tool, a Swiss army knife, there is Perl. Perl combines the capabilities of sed and awk, and throws in a large subset of C, to boot. It is modular and contains support for everything ranging from object-oriented programming up to and including the kitchen sink. Short Perl scripts lend themselves to embedding within shell scripts, and there may be some substance to the claim that Perl can totally replace shell scripting (though the author of the ABS
Guide remains skeptical).

Example 36-6. Perl embedded in a Bash script

```
#!/bin/bash

# Shell commands may precede the Perl script.
echo "This precedes the embedded Perl script within \"$0\"."
echo "==============================================================="

perl -e 'print "This line prints from an embedded Perl script.\n";'
# Like sed, Perl also uses the "-e" option.

echo "==============================================================="
echo "However, the script may also contain shell and system commands."

exit 0
```

It is even possible to combine a Bash script and Perl script within the same file. Depending on how the script is invoked, either the Bash part or the Perl part will execute.

Example 36-7. Bash and Perl scripts combined

```
#!/bin/bash
# bashandperl.sh

echo "Greetings from the Bash part of the script, $0."
# More Bash commands may follow here.

exit
# End of Bash part of the script.

# =======================================================

#!/usr/bin/perl
# This part of the script must be invoked with
#    perl -x bashandperl.sh

print "Greetings from the Perl part of the script, $0.\n";
#      Perl doesn't seem to like "echo" ...
# More Perl commands may follow here.

# End of Perl part of the script.

bash$ bash bashandperl.sh
Greetings from the Bash part of the script.


bash$ perl -x bashandperl.sh
Greetings from the Perl part of the script.
          

          It is, of course, possible to embed even more exotic scripting languages within shell wrappers. Python, for example ...

          Example 36-8. Python embedded in a Bash script

#!/bin/bash
# ex56py.sh

# Shell commands may precede the Python script.
echo "This precedes the embedded Python script within \"$0.\""
echo "==============================================================="

python -c 'print "This line prints from an embedded Python script.\n";'
# Unlike sed and perl, Python uses the "-c" option.
python -c 'k = raw_input( "Hit a key to exit to outer script. " )'

echo "==============================================================="
echo "However, the script may also contain shell and system commands."

exit 0

Wrapping a script around mplayer and the Google's translation server, you can create something that talks back to you.

Example 36-9. A script that speaks

#!/bin/bash
#   Courtesy of:
#   http://elinux.org/RPi_Text_to_Speech_(Speech_Synthesis)

#  You must be on-line for this script to work,
#+ so you can access the Google translation server.
#  Of course, mplayer must be present on your computer.

speak()
  {
        local IFS=+
          # Invoke mplayer, then connect to Google translation server.
            /usr/bin/mplayer -ao alsa -really-quiet -noconsolecontrols \
             "http://translate.google.com/translate_tts?tl=en&q="$*""
               # Google translates, but can also speak.
                 }

                 LINES=4

                 spk=$(tail -$LINES $0) # Tail end of same script!
                 speak "$spk"
                 exit
# Browns. Nice talking to you.

                 One interesting example of a complex shell wrapper is Martin Matusiak's undvd script, which provides an easy-to-use command-line interface to the complex mencoder utility. Another example is Itzchak Rehberg's Ext3Undel, a set of scripts to recover deleted file on an ext3 filesystem.
                 Notes
                 [1]    

                 Quite a number of Linux utilities are, in fact, shell wrappers. Some examples are /usr/bin/pdf2ps, /usr/bin/batch, and /usr/bin/xmkmf.
