## Dupbot unauthenticated remote code execution leading to system takeover

0day :)
Found: Monday 2018/01/15

By David

This is about the [Dupbot Discord Bot](https://github.com/jorenvandeweyer/Node_Dupbot/) < 11.3



### Requirements

* A discord account
* An empty discord server

### Info

This exploit abuses bad authentication checks when sending the bot commands through personal messages.

### How it works

When querying the bot to let it execute a command while you are in a server, it checks if you have the permission level required to execute said command.

However, due to a seriously sloppy bug, the permission level is not checked at all if you query a command via PMs

Unfortunately, the bot is configured to only allow certain commands to run when controlled via PM...

Luckily one of those commands is `!db`, which is used like this `!db <SQL query here>`. That's right! None of that fancy SQL injection stuff is needed when you have a publicly accessible SQL console!

Right off the bat we have complete access to the database. Unfortunately, on the bot that I tested this with, the MYSQL user was not able to interact with the file system (SELECT LOAD_FILE(), SELECT INTO OUTFILE did not work.)

What we **can** do however, is change the permissions needed to execute specific commands. This info is stored in the `permissions` table.

This is where your discord server comes into play. Start off by inviting the bot to your server. Of course we do **not** want to give every user in every discord server where the bot is active privileges to execute commands like `!eval`, `!evalt`, `!reload`, etc. 

This is why we first need to find out the internal guild ID that points to our discord server. You can do this easily by posting something like `!remindme foobar in 200 days` in your discord server. The bot will create a reminder for you, and now you can PM the both with something like `!db SELECT guild_id from events where data REGEXP '.*in 200 days.*'`. This will (hopefully) give you the guild id of your server. If multiple values get emitted, set a new reminder with a different date and change the regex.

Now that you have the guild_id, you need to PM the bot `!db UPDATE permissions set value=1 where guild_id={{GUILD_ID}}`. Of course replace `{{GUILD_ID}}` with the guild id you just retrieved.

Now, the last step: Go into your discord server and type in `!reload`.

Congrats, you now have code execution!

Type `!evalt ls -l /` into the discord server, and you will get back the directory listing of `/`.

It's up to you what to do with this exploit.

Cheers.

David
