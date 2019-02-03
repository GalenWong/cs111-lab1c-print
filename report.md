* Name: Galen Wong
* UID: 104939937 
* Email: wonggalen1999@gmail.com

# Experimental Report on CS111 Lab 1c 

I wrote 3 benchmarks to compare the performance between `simpsh`, `bash`, and `dash`.

## Benchmark 1

```bash
cat < input1.txt | tr -cs 'A-Za-z' '[\n*]' | sort -u | wc -l  > output1.txt
```
This command output "input1.txt" and tokenize the words.
The tokens are sorted uniquely and number of words is counted.
Essentially, we count how many unique words there are in input1.txt.

I ran this command with "input1.txt" being 5.9M.

Equivalent `simpsh` command:
```bash
./simpsh \
    --rdonly input1.txt \
    --creat --trunc --wronly output1.txt \
    --creat --trunc --wronly err1.txt \
    --pipe \
    --pipe \
    --pipe \
    --command 0 4 2 cat \
    --command 3 6 2 tr -cs 'A-Za-z' '[\n*]' \
    --command 5 8 2 sort -u \
    --command 7 1 2 wc -l \
    --close 4 --close 6 --close 8\
    --profile
    --wait
```

Results:

| Shell     | user CPU time (s) | system CPU time (s)   |
| --------  |-------------      | ------                |
| `simpsh`  | 1.1306 ÷ 3 = 0.3769 | 0.058482 ÷ 3 = 0.01949 |
| `bash`    | 1.136 ÷ 3 = 0.3787 | 0.073 ÷ 3 = 0.02343 |
| `dash`    | 1.13 ÷ 3 = **0.3767** | 0.05 ÷ 3 = **0.01667** |

* Commands are executed 3 times
* All time shown is in average
* Times are rounded to 4 significant figures
* Shortest time is highlighted in **bold**


## Benchmark 2

```bash
ps aux | grep vim | wc -l > 2v.txt; \
ps aux | grep emacs | wc -l > 2e.txt; \
printf "vim\n" > output2.txt; cat 2v.txt > output2.txt; \
printf "vs\nemacs\n" > output3.txt; cat 2e.txt > output2.txt
```
This command reads all the current active process.
Then, it selects process with `vim` and `emacs` in them and count the number of processes.
Essentially, it is a battle between vim and emacs.
Eventually, a conclusion will be drawn about the hierarcgy of the 2 editors.

Equivalent `simpsh` command:
```bash
./simpsh \
    --creat --rdwr dummy.txt \
    --creat --trunc --wronly 2v.txt \
    --creat --trunc --wronly 2e.txt \
    --creat --trunc --wronly err3.txt \
    --pipe \
    --pipe \
    --command 0 5 3 ps aux \
    --command 4 7 3 grep vim \
    --command 6 1 3 wc -l \
    --close 5 --close 7 \
    --wait \
    --pipe \
    --pipe \
    --command 0 9 3 ps aux \
    --command 8 11 3 grep emacs \
    --command 10 2 3 wc -l \
    --close 9 --close 11 \
    --wait \
    --creat --trunc --wronly output2.txt \
    --command 0 12 3 printf vim\n \
    --wait \
    --command 0 12 3 cat 2v.txt \
    --wait \
    --command 0 12 3 printf vs\nemacs\n\
    --wait \
    --command 0 12 3 cat 2e.txt \
    --profile
    --wait
```

Results:

| Shell     | user CPU time (s) | system CPU time (s)   |
| --------  |-------------      | ------                |
| `simpsh`  | 0.128708 ÷ 3 = **0.04290** |  0.436929 ÷ 3 = 0.145643 |
| `bash`    | 0.132 ÷ 3 = 0.0440 | 0.434 ÷ 3 = 0.1447 |
| `dash`    | 0.13 ÷ 3 = 0.0433 | 0.410 ÷ 3 = **0.1367** |



* Commands are executed 3 times
* All time shown is in average
* Times are rounded to 4 significant figures
* Shortest time is highlighted in **bold**


## Benchmark 3
```bash
dd bs=1024 if=/dev/urandom count=100000 | \
    tr -cd "A-Za-z0-9\!_()" | fold -w 20 | \
    sed -e "/^.{,19}$/d" >> output3.txt
```
This command reads 100000 * 1024 bytes from `/dev/urandom`.
And extract only a small subset of the ascii character set with `tr`.
Then, it generates random password of length 20, removing lines less than 20 characters.
Essentially, a very inefficient password generator.

Equivalent `simpsh` command:
```bash
./simpsh \
    --verbose \
    --creat --rdonly dummy.txt \
    --creat --trunc --wronly err3.txt \
    --pipe \
    --pipe \
    --pipe \
    --creat --append --wronly output3.txt \
    --command 0 3 1 dd bs=1024 if=/dev/urandom count=100000 \
    --command 2 5 1 tr -cd 'A-Za-z0-9!_()' \
    --command 4 7 1 fold -w 20 \
    --command 6 8 1 sed -e /^.{,19}$/d \
    --close 3 --close 5 --close 7 \
    --profile \
    --wait
```

Results:


| Shell     | user CPU time (s) | system CPU time (s)   |
| --------  |-------------      | ------                |
| `simpsh`  | 3.851608 ÷ 3 = 1.284 |  3.883207 ÷ 3 = 1.2944 |
| `bash`    | 3.934 ÷ 3 = 1.311 | 3.948 ÷ 3 = 1.316 |
| `dash`    | 3.840 ÷ 3 = **1.280** | 3.86 ÷ 3 = **1.287** |

* Commands are executed 3 times
* All time shown is in average
* Times are rounded to 4 significant figures
* Shortest time is highlighted in **bold**


## Conlusions

In general, `dash` performs the best in terms of both user CPU time and system CPU time.
Altough there is an exception in benchmark 2, the performance depends on the number of currently running process and other factors as well. 
