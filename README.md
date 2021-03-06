# Craft CMS Manager

Craft CMS is an excellent content management tool, but its configuration and setup can be a bit troublesome for those not familiar with PHP.
It's the main reason `craftman` arise, to help speed set up and start a new Craft CMS installation smoothly.

## Installation

First you'll need to make sure your system has git, docker and docker-compose.

Note: `craftman` does not support Windows.

### Install script

To install or update craftman, you can use cURL:

    curl -o- https://raw.githubusercontent.com/hatchwd/craftman2/master/craftman_install | CM="craftman2" sh

or Wget:

    wget -qO- https://raw.githubusercontent.com/hatchwd/craftman2/master/craftman_install | CM="craftman2" sh

<sub>The script clones the craftman repository to `~/.craftman2/bin` and adds the source line to your profile (`~/.bash_profile`, `~/.zshrc` or `~/.profile`).</sub>

You can customize the install source, directory and profile using the `CRAFTMAN_DIR`, and `PROFILE` variables.
Eg: `curl ... | CRAFTMAN_DIR="path/to/craftman2" sh`

## Usage

 Output from `craftman -h`:

    Craft CMS Manager

    Usage: craftman2 [options] <COMMAND> [args]

    Commands:
     craftman2 install               Install Craft CMS in current directory
     craftman2 open [path]           Open Craft CMS public site
     craftman2 admin                 Open Craft CMS admin dashboard
     craftman2 start                 Start Craft CMS docker containers
     craftman2 stop                  Stop Craft CMS docker containers
     craftman2 status                Check Craft CMS docker containers status
     craftman2 ip                    Show Craft CMS docker container IP address
     craftman2 run                   Open bash or run a command on Craft docker container
     craftman2 regenerate            Regenerate all configuration files
     craftman2 reconfigure           Run all scripts/install.* files
     craftman2 copy                  Copy scripts/override/**/* to craft container root /
     craftman2 remove                Remove all containers
     craftman2 --upgrade             Upgrade Craftman

    Options:
     -h, --help
     -v, --version
     -P, --port            HTTP port to expose on host

     -F, --force-all       Force redownload Craft CMS, regenerate and overwrite configurations and recreate containers
     -D, --force-download  Force to download latest Craft CMS from site
     -O, --force-overwrite Force to overwrite generated configuration files at app/ and scripts/ directories
     -R, --force-recreate  Force to reconfigure and recreate containers

    PHP Composer commands:
     craftman2 composer:lock         Regenerate composer.lock for your composer.json file
     craftman2 composer:prepare      Generate required files to deploy to Heroku

    Heroku commands:
     craftman2 heroku:prepare        Generate required files to deploy to Heroku

    MySQL commands:
     craftman2 mysql:run             Open mysql client or run a command on MySQL docker container
     craftman2 mysql:backup          Create a backup at backups/
     craftman2 mysql:restore <file>  Restore a backup from <file> (.sql.gz) to MySQL database

    PHP Docker Image commands:
     craftman2 phpimage:build [name] [base-image]  Build a new docker php image from this base docker image
     craftman2 phpimage:prepare                    Create base structure for php image generation
     craftman2 phpimage:set <name>                 Save default php image
     craftman2 phpimage:get                        Shows current php image
     craftman2 phpimage:remove                     Clear php image

    Full Documentation: https://github.com/gabrielmoreira/craftman



If you want to develop locally a new site using Craft CMS:

    mkdir mysite && cd mysite

    craftman --port=8080 install

If you want to deploy on heroku:

    craftman heroku:prepare

    # commit your files, and then:

    heroku create
    heroku addons:create cleardb:ignite
    git push heroku master
    heroku open /admin

How to access craft container terminal?

    craftman run

You can also access mysql container running:

    craftman mysql:run

If you need make a mysql backup, run it:

    craftman mysql:backup

If you want restore a mysql backup, run it:

    craftman mysql:restore <BACKUP_FILE_NAME>

## Advanced

Sometimes we need a little more customization. For that, you can use craftman hooks:

If you want hook mysql restore to edit .sql file before import:

Edit your `[YOUR-PROJECT-DIRECTORY]/.craftman2/config` and add this function:

<sub>**IMPORTANT**: THIS HOOK CODE IS JUST A SAMPLE USAGE</sub>

    function mysql_restore_hook()
    {
        cm_log "Processing '$1' file before import"
    	cm_log "+ Replace www.mysite.com to localhost"
    	cm_log "+ Remove https from localhost URLs"
    	cm_log "+ Doesn't import craft_searchindex (We prefer to rebuild index using craft)"
    	cat "$1" \
    		| sed -e 's/www\.mysite\.com/localhost/g' \
    		| sed -r 's/https?([:\/\\]+)localhost/http\1localhost/g' \
    		| perl -pe 'BEGIN{undef $/;} s/INSERT INTO .?craft_searchindex.*?DROP TABLE/DROP TABLE/sg' \
    			> "$1.tmp"
    	cm_log "+ Showing 20 lines of diff after file processed"
    	diff "$1" "$1.tmp" | head -n 20
    	cm_log "+ Removing temporary files"
    	mv "$1" "$1.bak"
    	mv "$1.tmp" "$1"
    	rm -f "$1.bak"
    }

There are other hooks:

function cm_config_hook()

        Use this hook to override craftman configurations

function mysql_backup_hook( *[sql-file-path]* )

        Use this hook to preprocess .sql before gzip file

function mysql_restore_hook( *[sql-file-path]* )

        Use this hook to preprocess .sql before import to MySQL

## Plugins

Create your plugin and put at `~/.craftman2/plugins/[your-plugin]/[your-plugin].plugin`

All public functions must start with `pluginname__`

        hello__usage() { ... }

All private functions name must start with `__pluginname_`

        __hello_some_util() { ... }

All variables must start with `PLUGINNAME__`

        HELLO__VAR="Hello World"

All hooks must start with "pluginname_" and ends with "_hook"

        hello_world_hook() { ... }

Notes:

- You can use `CM_` variables and `cm_` functions.
- Never use private craftman functions prefixed with `_cm`.
- Prefixes `cm_`, `_cm`, `CM_`, `CRAFTMAN_` for variables and functions are all reserved to `craftman`- Never do any work outside of functions.

## License

craftman is released under the MIT license.


Copyright (C) 2016 Gabriel Moreira

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Problems

If you try to install craft and the installation fails, you might get an error when trying to reinstall them again or you might get an error like the following:

    file not found

You can try force `-F` to redownload craft, `-O` to regenerate configurations and `-R` to recreate containers, using:

    craftman -F -O -R install

## TODO

- [x] Build a local Docker base image with updated php and libraries for fast installations
- [ ] Create user and group id on container if not exist
- [ ] Rename craftman private variables to _CM
- [ ] Rearrange functions in logical groups or files
- [ ] Rename app/ to site/
- [ ] Fix craft open browser on '0.0.0.0'

## Roadmap

- [x] MySQL backup and restore
- [x] Heroku support (only using S3 or other cloud asset source)
- [x] Plugins support
- [ ] Support bash completion
- [ ] Plugin install and update commands
- [ ] Advanced composer support (E.g. craftman compose:run install, craftman compose:run update, ...)
- [ ] Destroy all containers command
- [ ] Full backup and restore (code directory and database)
- [ ] OSX support
- [ ] Redis support for session cache
- [ ] Multi locale site structure generation
