# URL Rewrites

Once you start  using the embedded server for your development projects, you may wish to enable URL rewriting.  Rewrites are used by most popular frameworks to do things like add the `index.cfm` back into SES URLs.  

You may be used to configuring URL rewrites in Apache or IIS, but rewrites are also possible in CommandBox's embedded server via a [Tuckey servlet filter](http://tuckey.org/urlrewrite/).

http://tuckey.org/urlrewrite/

## Default Rules 

We've already added the required jars and created a default rewrite [XML file](http://cdn.rawgit.com/paultuckey/urlrewritefilter/master/src/doc/manual/4.0/index.html#filterparams) that will work out-of-the-box with the ColdBox MVC Platform.  To enable rewrites, start your server with the `--rewritesEnable` flag.

http://cdn.rawgit.com/paultuckey/urlrewritefilter/master/src/doc/manual/4.0/index.html#filterparams

```bash
start --rewritesEnable
```

Now URLs like 
```
http://localhost/index.cfm/main
```
can now simply be 
```
http://localhost/main
```

In `server.json`

```bash
server set web.rewrites.enable=true
server show web.rewrites.enable
```

> **info** The default rewrite file can be found in `~\.CommandBox\cfml\system\config\urlrewrite.xml`

## Custom Rules 

If you want to customize your rewrite rules, just create your own XML file and specify it when starting the server with the `rewritesConfig` parameter.  Here we have a simple rewrite rule that redirects  `/foo` to `/index.cfm`


**customRewrites.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<urlrewrite>
	<!-- this will redirect the user from /foo to /index.cfm -->
	<rule>
		<from>^/foo$</from>
		<to type="redirect">/index.cfm</to>
	</rule>
	<!-- internally redirect the requested URL from /gallery to /index.cfm?page=gallery with query string appended -->
	<rule>
		<from>^/gallery</from>
		<to type="passthrough" qsappend="true">/index.cfm?page=gallery</to>
	</rule>

</urlrewrite>
```

Then, fire up your server with its custom rewrite rules:
```bash
start --rewritesEnable rewritesConfig=customRewrites.xml
```
 
In `server.json`

```bash
server set web.rewrites.enable=true
server set web.rewrites.config=customRewrites.xml


server show web.rewrites.enable
server show web.rewrites.config
```
 
## Apache mod_rewrite-style rules

If you're coming from Apache, Tuckey supports a large subset of the `mod_rewrite` style rules like what you would put in `.htaccess`.  You can simply put your rules in a file named `.htacess` and point the `web.rewrites.config` property to that file.  

_Note: The name of the file matters with mod_reqrite-style rules. It must be called `.htaccess`. With xml rewrites, the filename is not important, only the content._

Here are some simple rewrite rules:

```bash
RewriteEngine on
RewriteRule ^/foo/                         /

# Defend your computer from some worm attacks
RewriteRule .*(?:global.asa|default\.ida|root\.exe|\.\.).* . [F,I,O]

# Redirect Robots to a cfm version of your robots.txt
RewriteRule ^/robots\.txt                   /robots.cfm

# Change your default cfm file to index.cfm
RewriteRule ^/default.cfm                   /index.cfm [I,RP,L]
RewriteRule ^/default.cfm((\?.+)|())$       /index.cfm$1  [I,RP,L]

RewriteRule ^/News.html((\?.+)|())$         /News/index.cfm$1 [I,RP,L]

# redirect mozilla to another area
RewriteCond  %{HTTP_USER_AGENT}  ^Mozilla.*
RewriteRule  ^/no-moz-here$                 /homepage.max.html  [L]
```

Please see the docs here on what's supported:

http://cdn.rawgit.com/paultuckey/urlrewritefilter/master/src/doc/manual/4.0/index.html#mod_rewrite_conf

>**info** For more information on custom rewrite rules, consult the [Tuckey docs](http://cdn.rawgit.com/paultuckey/urlrewritefilter/master/src/doc/manual/4.0/index.html#configuration).

## SES URLs
Your servers come ready to accept SES-style URLs where any text after the file name will show up in the `cgi.path_info`.  If rewrites are enabled, the `index.cfm` can be omitted.
```
site.com/index.cfm/home/login
```

SES URLs will also work in a sub directory, which used to only work on a "standard" Adobe CF Tomcat install.  Please note, in order to hide the `index.cfm` in a subfolder, you'll need a custom rewrite rule.

```
site.com/myFolder/index.cfm/home/login
```

