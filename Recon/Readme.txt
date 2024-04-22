THIS WILL RUN IN LINUX ENVIRONMENT ==> BASH SCRIPTING FOR RECON A IP (AUTOMATE NMAP SCANNER TASKS )
1  Start the Script
#!/bin/bash    

==> (#!) ==> This is called a shebang, or hashbang, and
/bin/bash ==> simply points to the system's interpreter for Bash.

2  condiional to fire this file : Usage 
if [ -z "$1" ]
then
        echo "Usage: ./recon.sh <IP>"
        exit 1
fi

==>  The $1 is the argument we will pass to the script, and the -z option returns true if the string is null. 
So basically, this says if no argument is passed, print the usage example and exit. The argument we'll use is 
an IP address.


3 Scan the Host
printf "\n----- NMAP -----\n\n" > results

==> We use printf here because it handles newlines (\n) more reliably than echo. This is being written to a 
text file called results — it will first create the file since it doesn't exist yet, and it will overwrite 
the file in the future with subsequent scans.


4 run the nmap scanner 
echo "Running Nmap..."
nmap $1 | tail -n +5 | head -n -3 >> results

==> The nmap command takes the argument we supplied the script, the IP address, and appends the results to 
our output file. The tail and head commands remove some lines from the beginning and end of the Nmap output 

>> -- means cin (wrt user input, store to a file ) ==> Thus, cin means "character input". 
| means search (here search first 3 and last 5 both )  ==> | ==> | grep to search a substring 

5 Enumerate HTTP (scan for nmap open ports with HTTP protocol )
while read line
do
        if [[ $line == *open* ]] && [[ $line == *http* ]]
        then
                echo "Running Gobuster..."
                gobuster dir -u $1 -w /usr/share/wordlists/dirb/common.txt -qz > temp1

        echo "Running WhatWeb..."
        whatweb $1 -v > temp2
        fi
done < results

==> whatweb and gobuster are tools  to scan any domain or ip   (read -- input from user  , line ==> parse 
each line)
If an open HTTP port is found, the code in the if-then block will run. Gobuster takes the IP address we 
supplied as the -u option and uses a wordlist specified by the -w option. We will also use the -q and -z 
options here to disable the banner and hide the progress — again, just to keep the output tidy. This will 
write to a temporary file that will be utilized later in the script.

WhatWeb simply takes the IP address we supplied and writes the output to a second temporary file. The -v 
option here gives us verbose results. When the while loop exhausts all lines of the results file, it 
completes, and the script moves on to the next section.

< a ==> output to a file 'a' (cout ==> character output in C++)

6 Display the Results 
if [ -e temp1 ]
then
        printf "\n----- DIRS -----\n\n" >> results
        cat temp1 >> results
        rm temp1
fi

if [ -e temp2 ]
then
    printf "\n----- WEB -----\n\n" >> results
        cat temp2 >> results
        rm temp2
fi

The -e option checks if the file exists — if it does, the code after then runs. It prints another heading, writes the contents of the temporary file to our results, and removes the temporary file.

cat  A >> B ==> copy the contents of A to B 
rm A ==> delete A permanently 

7 view the result 
cat results 
Finally, the last line of our script will simply display the results on our screen:

8 how to execute this script file 
~# chmod +x recon.sh  ==> First, make the script executable: (x means allow to execute permission in LINUX)

~# ./recon.sh   ==> If we try to run it without an argument, it gives us the usage example: 
Usage: ./recon.sh <IP>  ==> error 

~# ./recon.sh 10.10.0.50  ==> Simply supplied the IP address and run it again:

9 expected output : 
Running Nmap...
Running Gobuster...
Running WhatWeb...
/usr/lib/ruby/vendor_ruby/target.rb:188: warning: URI.escape is obsolete

----- NMAP -----

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
...      ...   ...
...      ...   ... 