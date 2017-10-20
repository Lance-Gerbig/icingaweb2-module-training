# Expand Icinga Web 2

## Write your own Icinga Web Module

Welcome! Nice that you are here to write your first own Icinga Web Module. Icinga Web makes it easy for us to get started. In the next few hours we will discover how pleasant the whole is by means of a series of practical examples.

## Should I really? Why?

Absolutely, why not? It is wonderfully easy, and Icinga is 100% free open source software with a great community. Icinga Web 2 is a stable, easy-to-understand and future-proof platform. That is exactly what you want to build your own projects.

## Only for monitoring?

Not at all! Sure, monitoring is where Icinga Web comes from. There it has its strength, there it is at home. After monitoring systems communicate with all possible systems in and outside their own computer center, we found it obvious to do so in the frontend in a similar way.

Icinga Web is a modular framework that makes the integration of third-party software as easy as possible. At the same time, we want to make it easy for third parties to use the logic of Icinga as easily as possible in their own projects.

Whether it's just linking third-party systems, connecting a CMDB, or visualizing complex systems as a complement to traditional check plugins - there is no limit to the imagination.

## I'm not a PHP / JavaScript / HTML5 hacker

No problem. However, it does not hurt to have a thorough knowledge of web development. However, Icinga Web allows you to write your own modules, even without the need for extensive PHP / HTML / CSS knowledge.

# Preparation

We use a debian base installation for the training to show how little dependencies Icinga Web 2 has. There are packages for all common distributions, but we will work directly with the GIT source tree in the training for the better learning effect.

So that our notebooks wake up from the weekend, we give them a small task:

    apt-get update

    # A few useful tools for the workshop:
    apt-get install git vim wget

    # Dependencies for Icinga Web 2:
    apt-get install php5-cli php5-mysql php5-gd php5-intl php5-ldap

    # Current source code from the Icinga Web 2 Master:
    cd / usr / local
    git clone https://git.icinga.org/icingaweb2.git

In the meantime, we will devote ourselves to the introduction!
## Structure of the training

* Setting up Icinga Web 2
* Create your own module
  * Own CLI commands
  * Working with parameters
  * Colors and other gimmiks
* Extension of the web frontends
  * Own pictures
  * Own Stylesheets
  * Extension of the menu
  * Provision of dashboards
* Working with data
  * Providing data
  * Pack code in libraries
  * Working with parameters
  * Tricks for comfortable working
* Configuration
* Translations
* Integration into third-party software
* Free Lab


## Icinga Web 2 Architecture

The development of Icinga Web 2 focused on three key areas:

* Simplicity.
* Speed
* Reliability


We have devoted ourselves to the DevOps movement, but our target group with Icinga Web 2 is clearly the operator, the admin. We try to have as little dependencies on external components. We do not give up on one or the other hippe feature, but then goes however also less broken if the Hippster in the next version again to make everything different. The best example is this workshop: now a year old, long before the first stable release of Icinga Web 2 written - and yet almost all exercises still work without any changes to the code.

The web interface was designed to be able to easily hang weeks and months on the same screen on the wall. We want to be able to rely on what we see there corresponds to the current state of our environment. If there are problems, they are visualized, even if they are in the application itself. If the problem is resolved, everything must continue as before. And without the fact that someone has to plug a keyboard and have to intervene manually.


## Used libraries

* Zend Framework 1.x
* jQuery 1.11 and 2.1
* Smaller PHP libraries
  * HTMLPurifier
  * DOMPdf
  * lessphp
  * Parsedown
* Smaller JS libraries
  jquery

## Anatomy of an Icinga Web 2 module

Icinga Web 2 follows the paradigm "convention before configuration". After the experience with Icinga Web 1, we came to the conclusion that one of the best tools for XML processing is on each disk: `/ bin / rm`. Anyone who adheres to a few simple conventions saves a lot of configuration work. Basically, in Icinga Web you only need to configure paths for very specific cases. It is usually enough to simply save a file to the correct location.

An extensive, adult module could have the following structure:

    ,
    └── training base directory of the module
        ├── application
        │ ├── clicommands CLI commands
        │ ├── controllers web controller
        │ ├── forms forms
        │ ├── locale translations
        │ └── views
        │ ├── helpers View Helper
        │ └── scripts View scripts
        ├── configuration.php Providing Menu, Dashlets, Permissions
        ├── doc documentation
        Library
        │ └── Training Library code, module namespace
        ├── module.info Metadata for the module
        ├── public
        │ ├── css Own CSS code
        │ ├── img Own pictures
        │ └── js Own JavaScript
        ├── run.php Registration of hooks and more
        └── test
            └── php PHP unit tests

In the course of this training, we will work out this step by step and fill it with life.

## Prepare the source tree

To be able to get started we need first and foremost Icinga Web 2. This can be checked out of the GIT Source Tree and used directly on the spot. If you then set `DocumentRoot` of a correspondingly configured web server to the public directory, you can also get started. For test purposes it is also even easier:

    cd / usr / local
    # If not done yet:
    git clone https://git.icinga.org/icingaweb2.git
    ./icingaweb2/bin/icingacli web serve

Finished. To use the installation wizard, a token is required for security reasons. It is from the web interface that we ensure that there is no time between installation and setup, to which an attacker could assume an environment. For Packager, this point is completely opional, the same applies to those Icinga Web with a CM tool such as Puppet roll out: if a configuration is on the system, you do not get the Wizard to face.

  http: // localhost

## Full installation with web server

We have been running fun Icinga Web 2 without an external web server. Even for most productive environments, this would be performant enough, but most of us feel more comfortable with a "real" web server. We stop so if not already done the PHP process and clear up:

```Sh
rm -rf / tmp / FileCache_icingaweb / / var / lib / php5 / sess_ *
```

These files, which are probably created by root, would otherwise only give us problems. Then we install our webserver:

```
apt-get install libapache2-mod-php5
./icingaweb2/bin/icingacli setup config webserver apache \
  > /etc/apache2/conf.d/icingaweb2.conf
service apache2 restart
```

You can see, Icinga Web 2 can generate its own configuration for Apache (2.x, also compatible to 2.4) itself. This applies not only to Apache, but also to Nginx.

## The configuration directory

If not configured otherwise, Icinga Web 2 searches its configuration in `/ etc / icingaweb`. This can be overwritten at any time with the environment variable ICINGAWEB_CONFIGDIR. We can also use this in the web server:

    SetEnv ICINGAWEB_CONFIGDIR / another / directory

## Manage multiple module paths

Anyone who wants to work with the latest version, or who wants to switch between GIT-Branches safely, will usually have to change files in his workspace. This is why it is recommended to use several module paths in parallel from the beginning. This can be done in the system settings or in the configuration under `/ etc / icingaweb / config.ini ':

    [global]
    module_path = "/ usr / local / icingaweb-modules: / usr / local / icingaweb2 / modules"


## Configuring Icinga CLI

When installing from packages, you do not have to worry about anything, for our convenience we also create a symbolic link for our GIT work:

    ln / usr / local / icingaweb2 / bin / icingacli / usr / local / bin /

## Installation from packages

The Icinga project builds up-to-date snapshots for a wide range of operating systems, and packages are available at [packages.icinga.org] (https://packages.icinga.org/). The current build status can be found at [build.icinga.org] (https://build.icinga.org/jenkins/view/Icinga%20Web%202/), and the current development at any time under [git.icinga.org] (https://git.icinga.org/?p=icingaweb2.git) or to [GitHub] (https://github.com/Icinga/icingaweb2/).

For our training we use the Git repository directly. And also in production, I do not reluctantly. Checksums for everything, changed files never remain undetected, version changes happen in a fraction of seconds - which package management can offer that already? In addition, this procedure nicely shows how simple Icinga Web 2 is actually. We did not have to modify a single file in the source directory for installation. Automake, configure? What for?! The configuration is somewhere else, and where it is, the runtime environment is reported.

# Create your own module

## Where should I start?

Probably the most important question is usually what you want to do with your module. In our training, we will first experiment with the given possibilities and then implement a small practical example.

## How should I name my module?

Once you know what the module is about to do, the most difficult task is often choosing a good name. In the ideal case, this shows what the module actually does. However, the name should not be too complicated. After all, we will use it in PHP namespaces, directory names, and URLs.

For your first own attempts, you often have your own (company) name. Our favored module name for our first try in training today is `training`.

## Create and activate a new module

    mkdir -p / usr / local / icingaweb-modules / training
    icingacli module list installed
    icingacli module enable training

Finished!

# Extending the Icinga CLI

The Icinga CLI is designed to provide as much as possible of the application logic available in Icinga Web 2 and its modules on the command line. The project aims to make the creation of cronjobs, plugins, useful tools and possibly small services as easy as possible.

## Configure Vim

We work in training with VIM and do a few initial settings:

    echo 'syntax on
    set expandab
    set tabstop = 4
    set shiftwidth = 4 '> /root/.vimrc

## Own CLI Commands

Structure of the CLI commands:

    icingacli <modul> <command> <action>


Creating a CLI command is very simple. In the `application / clicommands` directory, a file is created whose name corresponds to the desired command:

    cd / usr / local / icingaweb-modules / training
    mkdir -p application / clicommands
    vim application / clicommands / HelloCommand.php

Here, Hello corresponds to the desired command with a large initial letter. The ending command MUST always be set.

Example Command:

```Php
<? Php

namespace Icinga \ Module \ Training \ Clicommands;

use Icinga \ Cli \ Command;

class HelloCommand extends Command
{
}
``` 

## Namespaces

* Namespaces help to delineate modules neatly against each other
* Each module receives a namespace, which is derived from the module name:

```
Icinga \ modules \ <module name>
```

* The initial letter MUST be capitalized
* For CLI commands, a dedicated clicommand namespace is available

## Inheritance

All CLI Commands MUST inherit the Command class in the namespace `Icinga \ Cli`. This brings us a whole range of advantages, which we shall discuss later. It is important that our class name match the name of the file. In our HelloCommand.php this would be class HelloCommand.

## Command-Actions

Each command can have multiple actions. Any new public method that ends with action is automatically converted to a CLI command action:

```Php
<? Php
class HelloCommand extends Command
{
    public function worldAction ()
    {
        echo "Hello World! \ n";
    }
}
```

## Task 1

We create a CLI command with an action, which is served as follows and generates the following output:

    icingacli training say hello


## Bash Autocompletion

The Icinga CLI provides auto-completion for all modules, commands, and actions. If you install Icinga Web 2 per package everything is already in the right place, for our test environment we manually put on hand:

## Bash completion

    apt-get install bash-completion
    cp etc / bash_completion.d / icingacli /etc/bash_completion.d/
    , / Etc / bash_completion

If the input is ambiguous as with `icingacli mo`, then a corresponding help is displayed.


## Inline documentation for CLI commands

Commands and their actions can be easily documented via inline comments. The comment text is immediately available on the CLI as a help.

```Php
<? Php

/ **
 * This is where we say hello
 *
 * The hello command allows us to be friendly to everyone
 * and his dog. That's how nice people behave!
 * /
class HelloCommand extends Command
{
    / **
     * Use this to greet the world
     *
     * Greeting every single person would take some time,
     * so let's greet the whole world at once!
     * /
    public function worldAction ()
    {
        // ...
```

A few sample combinations of how the help can be displayed:

    icingacli training
    icingacli training hello
    icingacli help training hello
    icingacli training hello world --help

The Help command can be used at the beginning or can be used at any point as a parameter with `--`.

## Exercise 2

Create and test documentation for a `something` action for the` say` command in the `training` module!


## Command line parameters

Of course we can control, use and control command line parameters. Thanks to inheritance, the corresponding instance of `Icinga \ Cli \ Params` is already available in` $ this-> params`. The object has a `get ()` method, which we can specify the desired parameter and optionally a default value. Without default value, we get `null` if the corresponding parameter is not given.

```Php
<? Php

// ...

    / **
     * Say hello as someone
     *
     * Usage: icingacli training hello from --from <someone>
     * /
    public function fromAction ()
    {
        $ from = $ this-> params-> get ('from', 'Nowhere');
        echo "Hello from $ from! \ n";
    }
```

### Example call:

    icingacli training hello from --from Nürnberg
    icingacli training hello from - from "Netways Training"
    icingacli training hello from --help
    icingacli training hello from

## Standalone parameters

It is not absolutely necessary to assign an identifier to each parameter. If you want, you can also simply group parameters together. These are most conveniently accessible through the `shift ()` method:

```Php
<? Php

// ...

/ **
 * Say hello as someone
 *
 * Usage: icingacli training hello from <someone>
 * /
public function fromAction ()
{
    $ from = $ this-> params-> shift ();
    echo "Hello from $ from! \ n";
}
```

### Example call

    icingacli training hello from Nürnberg

## Shiften makes joy

The `shift ()` method behaves in the same way as usual programming languages ​​are used. The first parameter in the list is returned and removed from the list. If ```` ```` ```` ```` ```` ```` ```` ```` ```` ```` ```` ```` ```` ```` ``` With `unshift ()` you can undo an action at any time.

A special case is `shift ()` with an identifier (key) as a parameter. Thus `shift ('to')` would not only return the value of the `--to` parameter, but also remove it from the Params object regardless of its position. Again, it is possible to include a default value:

```Php
<? Php
// ...
$ person = $ this-> params-> shift ('from', 'Nobody');
```

Of course, this is also possible for standalone parameters. Since we have already occupied the first parameter of `shift ()` with the optional identifier (key), but we want to set something for the second (default value), we simply set the identifier to zero:

```Php
<? Php
// ...
public function fromAction ()
{
    $ from = $ this-> params-> shift (null, 'Nowhere');
    echo "Hello from $ from! \ n";
}
```

### Example call

    icingacli training hello from Nürnberg
    icingacli training hello from
    icingacli training hello from --help

## API documentation

The Params class in the `Icinga \ Cli` namespace also documents other methods and their parameters. These are most convenient to access in the API documentation. This can be generated with phpDocumentor, shortly there should be a CLI command.

## Task 3

Extend the say command to support all the following varieties:

    icingacli training say hello World
    icingacli training say hello --to World
    icingacli training say hello World - from "Icinga CLI"
    icingacli training say hello World "Icinga CLI"

## Exceptions

Icinga Web 2 wants to promote clean PHP code. This includes, among other things, that all warnings generate errors. Errors are thrown for error handling. We can try it out:

```Php
<? Php
// ...
use Icinga \ Exception \ ProgrammingError;
// ...
/ **
 * This action will always fail
 * /
public function kaputtAction ()
{
    throw new ProgrammingError ('No way');
}
```

### Calling

    icingacli training hello broken
    icingacli training hello kaputt --trace

## Exit codes

As we can see, the CLI catches all exceptions and gives pleasantly readable error messages together with colored indication of the error. In this case, the exit code is always 1:

    echo $?

Thus, failed jobs can be reliably evaluated. Only the exit code 0 stands for successful execution. Of course everyone is free to use additional exit codes. This is done in PHP using `exit ($ code)`. Example:

```Php
<? Php
echo "CRITICAL \ n";
exit (2);
```

Alternatively, Icinga Web provides the `fail ()` function in the Command class. It is an abbreviation for a colored "ERROR", a status output and `exit (1)`:

```Php
<? Php
$ this-> fail ('An error has occurred');
```

## To dye?

As we've just seen, the Icinga CLI can produce colored output. Using the screen class in the `Icinga \ Cli` namespace, useful help functions are available. We are accessing it in our command classes via `$ this-> screen`. Thus, the output can be colored:

```Php
<? Php
echo $ this-> screen-> colorize ("Hello from $ from! \ n", "lightblue");
```

As an optional third parameter, the `colorize ()` function can be used to specify a background color. ANSI escape codes are used to display the colors. If Icinga CLI detects that output is NOT done in a terminal / TTY, no colors are output. This ensures that e.g. when redirecting the output to a file, no spurious special characters appear.

> To recognize the terminal, the POSIX extension of PHP is used. If these are not present, no ANSI codes are used as a precaution.

Further useful features in the screen class are:

* `clear ()` to delete the screen (used by `--watch`)
* `underline ()` to underline text
* `newlines ($ count = 1)` to output one or more line breaks
* `strlen ()` to determine the character width without ANSI codes
* `center ($ text)` to output text depending on the center of the screen
* `getRows ()` and `getColumns ()` to find the usable place where possible
* `hasUtf8 ()` to query UTF8 support for the terminal

Caution: of course, does not work out to find someone in a UTF8 terminal with an ISO8859 putty on the road.

### Task

Our `hello` action in the` say` command should output the text in color and both horizontally and vertically. We use `--watch` to have the output flash alternately in at least two colors.

# The own module in the web frontend

Icinga Web would not carry __ "Web" __ in the name if its true qualities did not come to the fore. As we shall soon see, __convention before configuration__. According to the classic __MVC concept__, there are, of course, controllers with all available actions and suitable view scripts for output and display.

We deliberately refrained from the separation Library / Model, each additional layer also increases the complexity. One could also consider the library code as a "model" in many modules, but the specialists should clarify to us. We want at least as many smart modules, usually with a lot of reusable code, which then also benefits other modules.

## A first controller

Each `action` in a` controller` will automatically become a `route` in our web front-end. This looks something like this:

    http (s): // <host> / icingaweb / <module> / <controller> / <action>

If we want to create our "Hello World" again for our training module, we will first create the basic directory for our controllers:

    mkdir -p training / application / controllers

We then connect our controller. As you already guess, this HelloController.php must be named in the controller namespace of our module:

```Php
<? Php

namespace Icinga \ Module \ Training \ Controllers;

use Icinga \ Web \ Controller;

class HelloController extends Controller
{
    public function worldAction ()
    {
    }
}
```

If we call the Url (training / application / controllers) now, we get an error message:


    Server error: script 'hello / world.phtml' not found in path
    (/ Usr / local / icingaweb-modules / training / application / views / scripts /)

Practically she tells us what we have to do next.

## Create a view script

The corresponding base directory is still missing. Since we create a view script per "action" in a dedicated file, there is one directory per "controller":

    mkdir -p training / application / views / scripts / hello
    
The view script is then simply like the "action", so world.phtml:

```Php
<h1> Hello World! </ h1>
```

That's it, our new URL is now available. We could now use the full range for our module and style it accordingly. We can also use a few predefined elements. Two important classes are e.g. `controls` and` content` for header elements and page content.

```Php
<div class = "controls">
<h1> Hello World! </ h1>
</ Div>

<div class = "content">
Here you go ...
</ Div>
```

This automatically creates even distances to the page margins and also has the effect of scrolling down the `controls` while the content is scrolling. Of course, we will notice this only when we fill our module with more content.

## Menu items

Menu items in Icinga Web 2 can be personalized and / or specified by the administrator (*). However, they can be provided by modules. This is a global configuration, which can be made in the base directory of the own module in `configuration.php`:

```Php
<? Php

$ This-> menuSection ( 'training')
     -> add ('Hello World')
     -> setUrl ( 'training / hello / world');
```

### Icons for menu items

In order for our menu point to look better, we miss the same icon:

```Php
<? Php

$ This-> menuSection ( 'training')
     -> setIcon ( 'thumbs-up')
     -> add ('Hello World')
     -> setUrl ( 'training / hello / world');
```

To find out which icons are available, we enable the `doc` module under` System` / `Module`. Then we find the icon list under `Dokumentation` /` Developer - Style`. These are icons that are embedded in a font. This has the great advantage that very few requests have to be directed - the icons are simply "always there".

Alternatively, you can still use classic icons (.png etc) on request. This is especially useful when you want to use a special icon (for example, a company logo) for your module, which can be badly integrated into the official Icinga Icon font:

```Php
<? Php

$ This-> menuSection ( 'training') -> setIcon ( 'img / icons / success.png');
```

## Add pictures

If you would like to use your own pictures in your module, simply make them available in `public / img`:

    mkdir -p public / img
    wget https://www.icinga.org/wp-content/uploads/2014/06/tgelf.jpg
    mv tgelf.jpg public / img /

Our images are immediately accessible via web, the URL pattern is as follows:

    http (s): // <icingaweb> / img / <module> / <image>

For our concrete case, therefore, http: //localhost/img/training/tgelf.jpg. This can also be used wonderfully in our view script. Instead of creating an img tag (which of course would be possible) we use one of the many practical View-Helper:

```Php
...
<div class = "content">
<? = $ this-> img ('img / training / tgelf.jpg'), array ('title' => 'Thomas Gelf'))> Here you go ...
</ Div>
```

## Task

Create the URLs `training / hello / thomas` and` training / say / hello` and add an additional menu item. Also search for our training module a more beautiful icon from the Internet and set it accordingly.

## Dashboards

Before we deal with serious issues we want to provide our useful URL as a default dashboard. This can also be done in the `configuration.php`:

```Php
<? Php
$ this-> dashboard ('training') -> add ('hello', 'training / hello / world');
```

# We need data!

After our web-routes now work so wonderfully, we want to do something sensible. An application can still be so beautiful, without any useful content, it will quickly get boring. In an MVC environment, the `Controllers` usually use '

## Fill out our view with data

The controller provides access to our view in `$ this-> view`. In this way, it can be easily refueled:

```Php
<? Php

public function worldAction ()
{
    $ this-> view-> application = 'Icinga Web 2';
    $ this-> view-> moreData = array (
        'Work' => 'done',
        'Result' => 'fantastic'
    );
}
```

We are now expanding our View script and presenting the data transferred accordingly:

```Php
<h3> Some data ... </ h3>

This example is provided by <a href="http://www.netways.de"> Netways </a>
and based on <? = $ this-> application?>.

<Table>
<? php foreach ($ this-> moreData as $ key => $ val):?>
    <tr> <th> <? = $ key?> </ td> <? = $ val?> </ td> </ tr>
<? php endforeach?>
</ Table>
```

## Task

In `training / list / files` the content of our module directory should be listed in table form.
* Note: use `$ this-> Module () -> getBaseDir ()` to find out our module directory
* More about opening directories at [http://en.php.net/opendir](http://en.php.net/opendir)

# But please with style!

This is not directly related to our subject, but one thing is obvious: our table is not exactly particularly pretty. Luckily, we can also pack CSS in our module. We create a suitable directory, the name should be obvious:

    mkdir public / css

We then place our CSS statements in the file `module.less`. Less is a CSS extension for all kinds of functions, more can be found under [...] (). Traditional CSS is valid in any case. The nice thing about Icinga Web is that I do not have to worry about whether my CSS affects other modules or Icinga Web itself: this is not the case.

So we can easily define the following without making tables "broken":

    table {
        width: 100%;
    }

    th {
        width: 20%;
        text-align: right;
        line-height: 2em;
        padding-right: 2em;
    }

When we check the requests in the developer tools of our browser, we see that Icinga Web is the only CSS file to load css / icings.min.css. We can also load css / icinga.css to see what Icinga Web has made from our CSS code:

    .icinga-module.module-training table {
      width: 100%;
    }
    .icinga-module.module-training th {
      width: 20%;
      text-align: right;
      line-height: 2em;
      padding-right: 2em;
    }

As we see, prefixes ensure that our CSS is only valid in those containers where our module is content.

## Useful CSS classes

Icinga Web 2 provides a set of CSS classes that make things easier for us. For example, `common-table` is useful for common lists in tables,` name-value-table`. For names / value pairs where the identifier is represented as th and the right value is represented in a td. Also useful is `table-row-selectable` - this changes the behavior of the table. The whole line is highlighted when you move the mouse over it. If you click anywhere, the first link of the line comes to the train. In combination with `common-table`, the whole looks as good without any additional work.

# Real data cleans up

As we have seen before, such a module is really interesting with real data. What we did wrong, however, is that our controller is concerned about the data itself. This is unattractive and we would be at the latest when we want to use this data also on the CLI problems.

## Our own library

We create a new directory for our library in our module and follow the schema `library / <modulename>`. In our case:

    mkdir -p library / Training

For our module we use the name space `Icinga \ Module \ <modulename>` as already learned. Icinga Web 2 searches for all the namespaces under it automatically in the newly created directory. Exceptions are those seen previously, e.g. `Clicommands` or` Controllers`.

A Biblithek, which does the task implemented in the exercise, could lie in `File.php` and look as follows:

```Php
<? Php

namespace Icinga \ Module \ Training;

use DirectoryIterator;

class directory
{
    public static function listFiles ($ path)
    {
        $ result = array ();

        foreach (new directoryIterator ($ path) as $ file) {
            if ($ file-> isDot ()) continue;

            $ result [] = (object) array (
                'name' => $ file-> getFilename (),
                'path' => $ file-> getPath (),
                'size' => $ file-> getSize (),
                'type' => $ file-> getType ()
            );
        }
        return $ result;
    }
}
```

Our controller can now easily retrieve the data from our small library:

```Php
<? Php

// ...
use Icinga \ Module \ Training \ Directory;

class FileController extends Controller
{
    public function listAction ()
    {
        $ this-> view-> files = directory :: listFiles ($ this-> Module () -> getBaseDir ());
    }
}
```

## Task

Put this or a similar library into your module. Provide a view script, which can list the individual files for this purpose. Important: use `$ this-> escape ()` in the View script to escapen data whose origin is uncertain (e.g., filenames).

# Parameter handling

So far we have not given any parameters to our URLs. This is also quite simple. As in the command line, we have a simple access to params in Icinga Web. The access is as usual:

```Php
<? Php
$ file = $ this-> params-> get ('file');
```

Also `shift ()` and consorts are of course available again.
## Task

In `training / file / show? File = <filename>`, additional information about the desired file should be displayed. Hard-working show owners, permissions, last change and Mime type - but it is also just enough to display file name and size "in nice".

## Related Links

In our file list we now want to link from each file to the corresponding detail area. To avoid problems with the escaping of parameters, we use a new helper, `qlink`:

```Php
<td> <? = $ this-> qlink (
    $ File-> name,
    'Training / file / show'
    array ('file' => $ file-> name)
)?> </ td>
```

The first parameter is the text to be displayed, the second is the link to be created, and the third is the optional parameter for this link. As a fourth parameter, you could also specify an array with any additional HTML attributes.

If we now click on a file in our list, we end up with the corresponding details. But this is also more convenient. Just try to put `data-base-target =" _ next "` in the content-div:

    <div class = "content" data -base-target = "_ next">

This is the first time we have been managing the multi-column layout of Icinga Web!

# URL handling

Anyone who has observed how the browser behaves, may have noticed that the page is not reloaded with every click. Icinga Web 2 captures all requests and sends them by XHR request. On the server side this is recognized, and then only the respective HTML snippet is sent. This usually corresponds only to the output generated by the corresponding View-Script.

Nevertheless, each link remains a link and can be, e.g. in a new tab. Here again, it is recognized that this is not an XHR request, the complete layout is delivered.

Usually, links always end up in the same container, but you can influence the behavior with `data-base-target`. The next element closest to the selected element will win. If you want to undo `_next` for a part of the page, simply set` data-base-target = "_ self" `.

# Data handling made easy

Icinga Web still offers some nice tools. One thing we want to look at is the so-called DataSources. We include the ArrayDatasource and extend our library code by another function:

```Php
<? Php

use Icinga \ Data \ DataArray \ ArrayDatasource;

// ...

    public static function selectFiles ($ path)
    {
        $ ds = new ArrayDatasource (self :: listFiles ($ path));
        return $ ds-> select ();
    }
```

Then we change our controller very easily:


```Php
<? Php
$ query = directory :: selectFiles (
    $ This-> Modules () -> getBaseDir ()
) -> order ( 'type') -> order ( 'name');

$ this-> view-> files = $ query-> fetchAll ();
```

## Task 1
Modify the list so that you can sort it in ascending or descending order.
## Additional task

```Php
<? Php

$ editor = widget :: create ('filterEditor') -> handleRequest ($ this-> getRequest ());
$ Query-> applyFilter ($ editor-> getFilter ());
```

## Autorefresh

As a monitoring interface it is self-evident that Icinga Web provides a reliable and stable Autorefresh function. This can be conveniently controlled from the controllers:

```Php
<? Php

$ This-> setAutorefreshInterval (10);
```

## Exercise 2

Our file list should be updated automatically, the detail info as well. Show the change time of a file (`$ file-> getMtime ()`) and use the helper `timeSince` to display the time. Change a file to the hard drive and watch what happens. How can that be explained?

# Configuration

If you want to develop a module this probably synonymous configure. Configuration for a module can be found under `/ etc / icingaweb / modules / <modulename>`. What is found there in a `config.ini` is accessible in the controller as follows:

```Php
<? Php
$ config = $ this-> Config ();

/ *
[section]
entry = "value"
* /
echo $ config-> get ('section', 'entry');

// returns "default value" because "no entry" does not exist:
echo $ config-> get ('section', 'no entry', 'default value');

// Reads from the special.ini instead of the config.ini:
$ config = $ this-> Config ('special');
```

## Task
The base path for the list controller of our training module should be configurable. If no path is configured, we continue to use our module directory.

# Translations
For a detailed description of the translation possibilities, we open the documentation for the `translation` module. Here are the steps:

```Php
<h1> <? = $ this-> translate ('My files')?> </ h1>
```

    apt-get install gettext poedit
    icingacli module enable translation
    icingacli translation: refresh module training de_DE
    # Translate using Poedit
    icingacli translation compile module training de_DE

# Icinga Web use logic in third-party software?

With Icinga Web 2, we want to make the integration of third-party software as simple as possible. We also want others to easily use Icinga Web logic in their software.

This is basically the following call in any PHP file:

```Php
<? Php

require_once 'Icinga / Application / EmbeddedWeb.php';
Icinga \ Application \ EmbeddedWeb :: start ();
```

Finished! No authentication, no bootstrapping of the full web interface. But all the library code that is available can be used.

## Task
Create an additional PHP file, which Icinga Web 2 embeds. Then use the library for directory handling from your training module.

# Free Lab

You really have arrived here? In a single day? Respect. The speaker is now guaranteed to have an exciting final exercise session, to end the day with a practical example and many new tricks.

Have fun with Icinga Web 2 !!!