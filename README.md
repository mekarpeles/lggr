# Lggr - Python Logging For Humans

Have you ever tried to do anything with the python logging module?

I have. I didn't like it at all. The API was very confusing. Instead of dealing with all of its intricacies, I decided to roll my own.

I've been inspired by [dabeaz](http://www.dabeaz.com/)'s presentation on coroutines and Kenneth Reitz's [presentation](http://python-for-humans.heroku.com/) on better python libraries.

# Install
```bash
pip install lggr
```

# How does it work?

Create a logger object.

```python
import lggr
d = lggr.Lggr()
```

Add a coroutine (or any function or object with `send` and `close` methods) to consume log messages. `lggr` includes some default ones:

* `lggr.Printer()` writes to stdout
* `lggr.StderrPrinter()` writes to stderr
* `lggr.Printer(filepath)` opens a file at `filepath` and writes to that.
* `lggr.SocketWriter(host, port)` writes to a network socket
* `lggr.Emailer(recipients)` sends emails
* `lggr.GMailer(recipients, gmail_username, gmail_password, subject="optional")` also sends emails, but does it from Gmail which is way sexier than doing it from your own server.

You can choose to add different coroutines to different levels of logging. Maybe you want to receive emails for all of your critical messages, but only print to stderr for everything else.

```python
d.add(d.ALL, lggr.Printer()) # d.ALL is a shortcut to add a coroutine to all levels
d.add(d.CRITICAL, lggr.Emailer("peterldowns@gmail.com"))
```

Do some logging.

```python
d.info("Hello, world!")
d.warning("Something seems to have gone {desc}", {"desc":"amuck!"})
d.critical("Someone {} us {} the {}!", "set", "up", "bomb")
d.close() # stop logging
```

# What kind of information can I log?
Anything you want. Log messages are created using `str.format`, so you can really create anything you want. The default format includes access to the following variables:

* `levelname` = level of logging as a string (`"INFO"`)
* `levelno` =  level of logging as an integer (`0`)
* `pathname` = path to the file that the logging function was called from (`~/test.py`)
* `filename` = filename the logging function was called from (`test.py`)
* `module` = module the logging function was called from (in this case, `None`)
* `exc_info` = execution information, either passed in or `sys.info()`
* `stack_info` = stack information, created if the optional `inc_stack_info` argument is `True` (it defaults to `False` if not explicitly passed) or the logging function is called with instance functions `critical`, `debug`, or `error`.
* `lineno` = the line number
* `funcname` = the function name 
* `code` = the exact code that called the logging function
* `codecontext` = surrounding 10 lines surrounding `code`
* `process` = current process id
* `processname` = name of the current process, if `multiprocessing` is available
* `asctime` = time as a string (from `time.asctime()`)
* `time` = time as seconds from epoch (from `time.time()`)
* `threadid` = the thread id, if the `threading` module is available
* `threadname` = the thread name, if the `threading` module is available
* `messagefmt` = the format string to be used to create the log message
* `logmessage` = the user's formatted message
* `defaultfmt` = the default format of a log message

If you want to use any extra information, simply pass in a dict with the named argument `extra`:

```python
>>> d.config['defaultfmt'] = '{name} sez: {logmessage}'
>>> d.info("This is the {}", "message", extra={"name":"Peter"})
Peter sez: This is the message
```
### A `stack_info` example

`stack_info` is cool because it lets you do really helpful tracebacks to where exactly your logging function is being called. For example, with some logger d, I could run the following:

```python
d.config['defaultfmt'] = '{asctime} ({levelname}) {logmessage}\nIn {pathname}, line {lineno}:\n{codecontext}'

def outer(a):
	def inner(b):
		def final(c):
			d.critical("Easy as {}, {}, {}!", a, b, c)
		return final
	return inner

outer(1)(2)(3)
```

output:

```python
Mon Apr  2 23:31:22 2012 (CRITICAL) Easy as a, b, c!
In test.py, line 29:
d.config['defaultfmt'] = '{asctime} ({levelname}) {logmessage}\nIn {pathname}, line {lineno}:\n{codecontext}'

def outer(a):
	def inner(b):
		def final(c):
>			d.critical("Easy as {}, {}, {}!", a, b, c)
		return final
	return inner

outer(1)(2)(3)
```

# Is it robust?

Not *yet*. It's definitely still in the early days, so use at your own risk. I wouldn't put this into production right now, but it's been working fine for my toy projects. Please submit any bugs you find to the github tracker!

Also, for the risk-averse, it ignores all errors when calling the `.log(...)` method. So if you seriously mess up, the worst thing that can happen is that you won't see a log message. For the less risk-averse, this can be overridden when the `Lggr` object is created. 

# What's next?
I'm still working on text-sending and IRC/IM-writing log functions - maybe one of you could help? 

Also, I've sort of ignored the Unicode problem entirely. That needs to change.

I'd also like to add serialization / loading of both `Lggr` objects and their configurations, for easier saving / loading.


# Who did this?
[Peter Downs.](http://peterdowns.com)  
[peterldowns@gmail.com](mailto:peterldowns@gmail.com)  
[@peterldowns](http://twitter.com/peterldowns)


