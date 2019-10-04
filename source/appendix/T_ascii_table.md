# 附录 T. ASCII 表

传统上，这类书的附录会有一个 ASCII 表。但本书不会。取而代之的是这些生成一张完整 ASCII 表的简短脚本。

例 T-1. 一个生成 ASCII 表的脚本

```bash
#!/bin/bash
# ascii.sh
# ver. 0.2, reldate 26 Aug 2008
# Patched by ABS Guide author.

# Original script by Sebastian Arming.
# Used with permission (thanks!).

exec >ASCII.txt         #  Save stdout to file,
                        #+ as in the example scripts
                        #+ reassign-stdout.sh and upperconv.sh.

MAXNUM=256
COLUMNS=5
OCT=8
OCTSQU=64
LITTLESPACE=-3
BIGSPACE=-5

i=1 # Decimal counter
o=1 # Octal counter

while [ "$i" -lt "$MAXNUM" ]; do  # We don't have to count past 400 octal.
        paddi="    $i"
        echo -n "${paddi: $BIGSPACE}  "       # Column spacing.
        paddo="00$o"
#       echo -ne "\\${paddo: $LITTLESPACE}"   # Original.
        echo -ne "\\0${paddo: $LITTLESPACE}"  # Fixup.
#                   ^
        echo -n "     "
        if (( i % $COLUMNS == 0)); then       # New line.
           echo
        fi
        ((i++, o++))
        # The octal notation for 8 is 10, and 64 decimal is 100 octal.
        (( i % $OCT == 0))    && ((o+=2))
        (( i % $OCTSQU == 0)) && ((o+=20))
done

exit $?

# Compare this script with the "pr-asc.sh" example.
# This one handles "unprintable" characters.

# Exercise:
# Rewrite this script to use decimal numbers, rather than octal.
```

例 T-2. 另一个 ASCII 表脚本

```bash
#!/bin/bash
# Script author: Joseph Steinhauser
# Lightly edited by ABS Guide author, but not commented.
# Used in ABS Guide with permission.

#-------------------------------------------------------------------------
#-- File:  ascii.sh    Print ASCII chart, base 10/8/16         (JETS-2012)
#-------------------------------------------------------------------------
#-- Usage: ascii [oct|dec|hex|help|8|10|16]
#--
#-- This script prints out a summary of ASCII char codes from Zero to 127.
#-- Numeric values may be printed in Base10, Octal, or Hex.
#--
#-- Format Based on: /usr/share/lib/pub/ascii with base-10 as default.
#-- For more detail, man ascii . . .
#-------------------------------------------------------------------------

[ -n "$BASH_VERSION" ] && shopt -s extglob

case "$1" in
   oct|[Oo]?([Cc][Tt])|8)       Obase=Octal;  Numy=3o;;
   hex|[Hh]?([Ee][Xx])|16|[Xx]) Obase=Hex;    Numy=2X;;
   help|?(-)[h?])        sed -n '2,/^[ ]*$/p' $0;exit;;
   code|[Cc][Oo][Dd][Ee])sed -n '/case/,$p'   $0;exit;;
   *) Obase=Decimal
esac # CODE is actually shorter than the chart!

printf "\t\t## $Obase ASCII Chart ##\n\n"; FM1="|%0${Numy:-3d}"; LD=-1

AB="nul soh stx etx eot enq ack bel bs tab nl vt np cr so si dle"
AD="dc1 dc2 dc3 dc4 nak syn etb can em sub esc fs gs rs us sp"

for TOK in $AB $AD; do ABR[$((LD+=1))]=$TOK; done;
ABR[127]=del

IDX=0
while [ $IDX -le 127 ] && CHR="${ABR[$IDX]}"
   do ((${#CHR}))&& FM2='%-3s'|| FM2=`printf '\\\\%o  ' $IDX`
      printf "$FM1 $FM2" "$IDX" $CHR; (( (IDX+=1)%8))||echo '|'
   done

exit $?
```
例 T-3. 第三个 ASCII 表脚本，使用 *awk*

```bash
#!/bin/bash
# ASCII table script, using awk.
# Author: Joseph Steinhauser
# Used in ABS Guide with permission.


#-------------------------------------------------------------------------
#-- File:  ascii     Print ASCII chart, base 10/8/16         (JETS-2010)
#-------------------------------------------------------------------------
#-- Usage: ascii [oct|dec|hex|help|8|10|16]
#--
#-- This script prints a summary of ASCII char codes from Zero to 127.
#-- Numeric values may be printed in Base10, Octal, or Hex (Base16).
#--
#-- Format Based on: /usr/share/lib/pub/ascii with base-10 as default.
#-- For more detail, man ascii
#-------------------------------------------------------------------------

[ -n "$BASH_VERSION" ] && shopt -s extglob

case "$1" in
   oct|[Oo]?([Cc][Tt])|8)       Obase=Octal;  Numy=3o;;
   hex|[Hh]?([Ee][Xx])|16|[Xx]) Obase=Hex;    Numy=2X;;
   help|?(-)[h?])        sed -n '2,/^[ ]*$/p' $0;exit;;
   code|[Cc][Oo][Dd][Ee])sed -n '/case/,$p'   $0;exit;;
   *) Obase=Decimal
esac
export Obase   # CODE is actually shorter than the chart!

awk 'BEGIN{print "\n\t\t## "ENVIRON["Obase"]" ASCII Chart ##\n"
           ab="soh,stx,etx,eot,enq,ack,bel,bs,tab,nl,vt,np,cr,so,si,dle,"
           ad="dc1,dc2,dc3,dc4,nak,syn,etb,can,em,sub,esc,fs,gs,rs,us,sp"
           split(ab ad,abr,",");abr[0]="nul";abr[127]="del";
           fm1="|%0'"${Numy:- 4d}"' %-3s"
           for(idx=0;idx<128;idx++){fmt=fm1 (++colz%8?"":"|\n")
           printf(fmt,idx,(idx in abr)?abr[idx]:sprintf("%c",idx))} }'

exit $?
```
