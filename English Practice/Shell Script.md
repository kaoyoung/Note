This artical is based on [The Shell Scripting Tutorial](https://www.shellscript.sh/)
# Home
- As such, it has been written as **a basis for** one-on-one or group tutorials and exercises, and as a reference for **subsequent use**
> subsequent : **happening after something else**
> example : 
> Those explosions must have been subsequent to our departure, because we didn't hear anything.

- Command-line entries will be preceded by the Dollar sign ($).
>precede : **to be or go before something or someone in time or space**
>example : 
>It would be helpful if you were to precede the report with an introduction.

---
# Philosophy
- Shell script programming has a bit of a bad press amongst some Unix systems administrators.
>administrator : **someone whose job is to control the operation of a business, organization, or plan**
>example : 
>From 1969 to 1971, he was administrator of the Illinois state drug abuse program.

- You may be forgiven for thinking that with a simple script, this is not too significant a problem, but two things here are worth **bearing in mind**.
	1. First, a simple script will, more often than anticipated, grow into a large, complex one.
	2. Secondly, if nobody else can understand how it works, you will be lumbered with maintaining it yourself for the rest of your life!
>anticipate : **to imagine or expect that something will happen**
>example : 
>We had one or two difficulties along the way that we didn't anticipate.

>lumber : **to move slowly and awkwardly**
>example : 
>**In the distance**, we could see a herd of elephants lumbering across the plain.

- Something about shell scripts seems to make them particularly likely to be badly indented, and since the main control structures are if/then/else and loops, indentation is critical for understanding what a script does.
> indent : shifting text or code inward from the margin

- Of course, this kind of thing is what the OS is there for, and it's normally pretty efficient at doing it.
> be there for someone : **to be available to provide help and support for someone**
>example : 
 >Best friends are always there for each other in times of trouble.
 
 Some Unices are more efficient than others at what they call "building up and **tearing down** processes" - i.e., loading them up, executing them, and clearing them away again.
 > tear sth down : **to intentionally destroy a building or other structure because it is not being used or it is not wanted any more**
 > example : 
 > They're going to tear down the old hospital and build a new one.
 
 - The Award For The Most Gratuitous Use Of The Word Cat In A Serious Shell Script being **bandied** about on the `comp.unix.shell` newsgroup **from time to time**.
 >bandy about : **to talk about something without careful consideration:**
 >example : 
 >Wild guesses of the value of the painting were being bandied about.
 
 >from time to time : sometimes, but not often
 >example : 
 >My former students get back in touch with me from time to time and it's really nice.

---
# A first script
- This is a special directive which Unix treats specially.
>directive : **an official instruction**
>example : 
>The boss issued a directive about not using the fax machine.

---
# Variables - Part 1
- Let's look back at our first Hello World example. This could be done using variables (though it's such a simple example that it doesn't really **warrant** it!)
>warrant : **to make a particular activity necessary**
>example :
>It's a relatively simple task that really doesn't warrant a great deal of time being spent on it.

- This is because the external program `expr` only expects numbers. But there is no syntactic difference between:
>syntatic :  **relating to the grammatical arrangement of words in a sentence**
>example : 
>Readers use their syntactic and semantic knowledge to decode the text.

- We can interactively set variable names using the `read` command; the following script asks you for your name then **greets** you personally:
>greet : **to welcome someone with particular words or a particular action, or to react to something in the stated way**
>example :
>The teacher greeted each child with a friendly "Hello!"

- **What happens is that** the `read` command automatically places quotes around its input, so that spaces are treated correctly.
>What happend is that : **It is a common English phrase used to introduce an explanation, description, or consequence of a situation**, essentially meaning "the result is that" or "here's the situation".

- When you call `myvar2.sh` from your interactive shell, a new shell is **spawned** to run the script.
>spawn : **to cause something new, or many new things, to grow or start suddenly**
>example :
>The new economic freedom has spawned hundreds of new small businesses.

- This can be the **downfall** of many a new shell script programmer, as the source of the problem can be difficult to **track down**.
>downfall : **(something that causes) the usually sudden destruction of a person, organization, or government and their loss of power, money, or health**
>example :
>Rampant corruption brought about the downfall of the government.

>track something/someone down : **to find something or someone after looking for it, him, or her in a lot of different places**
>example :
>He finally managed to track down the book he wanted. 
# Wildcard 
- Wildcards are really nothing new if you have used Unix **at all** before.
>at all : **(used to make negatives and questions stronger) in any way or of any type**
>example : 
>Why bother getting up at all when you don't have a job to go to?

- This section is really just to get the old **grey cells** thinking how things look when you're in a shell script - predicting what the effect of using different syntaxes are.
>get the old grey cells thinking : **to use your brain, specifically your intelligence and mental power, to solve a problem, often with careful, logical thought.**

---
