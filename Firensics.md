# 🔓 Firensics - 275pts

```
💡 I got extremely lucky and managed to finish this very quickly, my advice is just to not be intimidated at all,
   there are a lot of files.
```

### Required Files (Places is a zip bc compression 👍)

[formhistory.sqlite](https://prod-files-secure.s3.us-west-2.amazonaws.com/f4c74cae-e956-4d85-b7e7-836e2e353ae6/185ee2e3-b838-49b5-aac0-193f1b792d11/formhistory.sqlite)

[places.zip](https://prod-files-secure.s3.us-west-2.amazonaws.com/f4c74cae-e956-4d85-b7e7-836e2e353ae6/dc0a4b64-123c-4f28-ac00-d2dd98a5c6c4/places.zip)

### My Thought Process (Very flawed)

- Looking at even just the file names, it becomes clear that you are working with some type of `browser data dump`
    - Determining the exact browser used could help you to sort through the data in a manageable way using open-source software or smth
    - It’s a little obvious from the `moz` tags but its **************Firefox************** (doesn't really matter)
- I hate `JSON` files, I will not touch them yet
- I actually just don’t recognize any of the file types besides `sqlite` so I focus on that

### Viewing SQLite

Now I really respect people who used a real terminal for this, but my favorite browser-based SQLite Viewer is by **[INLOOPX](https://inloop.github.io/sqlite-viewer/)**

First file I viewed was `formhistory.sqlite`, it was the first one in the list (besides `favicon` which I know, from past knowledge, that it probably just has image data)

- Looking for significant details like `logins` or `passwords`

`SELECT * FROM 'moz_formhistory' LIMIT 0,30`

![image of formhistory.sqlite](/assets/firensics1.png)

The name `Rick Ash` is noteworthy along with the username `ashrick` and the password `r0llr1ck0202!`

Using these details, I just casually combed through some of the SQLite tables until something was found in `places.sqlite`

`SELECT * FROM 'moz_places' LIMIT 30,30`

![image of places.sqlite](/assets/firensics2.png)

Bingo, navigating to `ashrick’s pastebin` account, you will find a password protected file

![image of pastebin](firensics3.png)

Using the password from earlier, `r0llr1ck0202!`, you can see the details of the pastebin with the flag

```markup
Hello Boss,
 
I'll start transferring the documents as discussed earlier. For now, keep this secret: HTB{br0ws3r_f0r3ns1cs_iz_ez!}
```
