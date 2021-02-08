We have been working with [Microsoft Oryx](https://github.com/microsoft/oryx) to provide the ability for App Service users to use any SDK they want their deployment scripts to run/Kudu Bash to have in it's context.

**Changing the context in the bash/custom build scripts**

The SDKs can be loaded into the context by executing:

```
oryx prep --skip-detection --platforms-and-versions "php=7.3.21"
source benv php=7.3.21
```

Now the above PHP binaries are accessible to the bash:
```
kudu_ssh_user@45b8676e71f1:~$ php -v
PHP 7.3.21 (cli) (built: Aug 20 2020 08:09:36) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.21, Copyright (c) 1998-2018 Zend Technologies
```


***
**Please note**: For custom deployment scripts, certain new SDKs listed below may not be present in the bash context by default, the above commands can be used to get these SDKs into the context:
* Node:14-LTS
* PHP:7.4
* Dotnet:5.0
* Python:3.9
***

Please create a new issue if you find any bugs. Happy Debugging!