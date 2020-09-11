---
layout: post
title: Starting out as a Gopher - Implementing ls command in  golang
categories: Golang
tags:
- Golang, Tutorial
---


<table><tbody><tr><td>
<img src="../assets/images/me_gopher.png" /></td>
<td>
Although it appears that I'm late to the <a href="https://blog.golang.org/gopher"> Gophers </a> party, I have finally managed to embrace Idiomatic Go in my programming arsenal.

<p />
I now realized that better late than never. Golang is truely a <b>highly productive language</b>

<p />
In this post, I'm gonna talk about my new project <b> <a href="https://github.com/kspviswa/lsgo"> lsgo </a> - ls command in golang for fun & learning </b>. 
</td></tr></tbody></table>

<hr>

After watching various [Gophercon videos](https://www.gophercon.com/) , Rob Pike's ["Concurrency is not Parallelism" ](https://www.youtube.com/watch?v=cN_DpYBzKso) & Liz's ["Containers from Scratch"](https://www.youtube.com/watch?v=Utf-A4rODH8) , I was super charged in getting my hands dirty with `golang`.

Rather than starting out in traditional hello-world world, I decided to take a spin and wanted to try with real-world program.

Well, who doesn't know about `ls` command?? . Simple, yet powerful. So what about, implementing `ls` command from scratch using `golang` ? and how much time does a newbie like me, would typically need in converting an idea to implementation, using `golang` ??

As it turns out, it only took me a weekend to learn `golang` basic & it's [powerful idioms](https://pocketgophers.com/idiomatic-go/), there by implementing a subset of mighty `ls` command purely in `golang`.

The power of **golang** is such that, what started out as something very trivial as  below

{% highlight go %}

func serveDir(dir string) {
	f, err := os.OpenFile(dir, os.O_RDONLY, 0666)
	checkerr(err)
	files, err := f.Readdirnames(0)
	checkerr(err)
	for _, file := range files {
		fmt.Println(file)
	}
}

{% endhighlight %}

ramped up into a complete tool within short span of time ( read productivity ). No wonder why every start-up is thriving with `golang` .

Needless to say, the experience was so much fun & absolutely addictive. I'm sure I'm gonna use `golang` for all my future coding endeavours.

## Intrigued to learn more about my `lsgo` project?? 

`lsgo` is available in [Github](https://github.com/kspviswa/lsgo). While not all features of `ls` isn't available, this tool serves `PWD` & any directory when provided as an input.

Along with normal file listing, this tool also supports *long, size in human readable format, file-only, dir-only* listing. Furthermore, it also supports **tree** lookup *(recursive directory lookup)* .

 The goal is not to replace `ls`, rather than to mimic `ls` in golang to the extent possible using native `golang` constructs  - basically for fun & learning golang.
  
### Usage
 
{% highlight text %}
 ./ls --help

NAME:
   ls ( implemented in golang ) - ls [flags] [command][args]

USAGE:
   ls [global options] command [command options] [arguments...]

VERSION:
   0.1

AUTHOR:
   Viswanath Kumar Skand Priya <kspviswa.github@gmail.com>

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   -l, --long             include extended information
   -d, --dronly           include only directories
   -f, --fileonly         include only regular files
   --hr, --humanfriendly  view size in humar readable format
   -t, --tree             view tree structure ( recursive lookup )
   --help, -h             show help
   --version, -v          print the version

COPYRIGHT:
   MIT Licensed
 
{% endhighlight %}
 

### Sample output

{% highlight text %}
./ls -hr
+-----------+--------+------------+---------------------+-------+
|   NAME    |  SIZE  |   PERMS    |         AT          |  DIR  |
+-----------+--------+------------+---------------------+-------+
| .git      | 4K     | drwxrwxrwx | 2017-10-22 01:15:14 | true  |
| demo.json | 116.5K | -rwxrwxrwx | 2017-10-22 01:02:45 | false |
| LICENSE   | 1.1K   | -rwxrwxrwx | 2017-10-21 19:34:57 | false |
| ls        | 3.5M   | -rwxrwxrwx | 2017-10-22 00:31:26 | false |
| ls.go     | 3.2K   | -rwxrwxrwx | 2017-10-22 00:31:23 | false |
| README.md | 166B   | -rwxrwxrwx | 2017-10-22 01:11:07 | false |
+-----------+--------+------------+---------------------+-------+

{% endhighlight %}
 
## Features demo - webcast
 
 <script type="text/javascript" src="https://asciinema.org/a/66oqlLrbMehCGcgYxh06lYJs6.js" id="asciicast-66oqlLrbMehCGcgYxh06lYJs6" async></script>
 
 <hr>
 
 If you happen to like my project, feel free to let me know your thoughts and I would love to hear them. Cya in my next blogpost..
 
 - A learning Gopher...
