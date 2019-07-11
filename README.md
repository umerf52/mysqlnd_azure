# PHP mysqlnd redirection extension mysqlnd_rd
The source code here is a PHP extension implemented using mysqlnd plugin API (https://www.php.net/manual/en/mysqlnd.plugin.php), which provides redirection feature support. 

## Name and Extension Version
The extension is named with **mysqlnd_rd**, source code here has two branches:
* mysqlnd-rd-php-7-2 : Compatiable with PHP7.2 latest version and PHP7.3 latest version
* mysqlnd-rd-php-7-4 : Compatiable with PHP7.4

Following is a brief guide of how to build and test the extension. 
## Step to build on Linux
* Install phpize for build using following command
  - Ubuntu: apt-get install php7.x-dev  
  //use the version that corresponds to your PHP version, i.e. if you use PHP7.3, use php7.3-dev here
  - Redhat: yum install php7x-php-devel
  //e.g. php72-php-devel. after install, you need to link phpize and php-config in order to make it work correctly:
    - ln -s /opt/remi/php72/root/bin/phpize /usr/bin/phpize
    - ln -s /opt/remi/php72/root/bin/php-config /usr/bin/php-config
* Chose your folder to hold the source code, e.g. php-mysqlnd_extension, and cd to it
* git clone https://github.com/microsoft/php-mysqlnd .
* git checkout to branch mysqlnd-rd-php-7-2 or mysqlnd-rd-php-7-4
* cd to the ext/mysqlnd_rd source folder, and run
  - phpize
  - ./configure
  - make 
* After that, you will find a .so file named **mysqlnd_rd.so** under ./modules folder. Then you can run **make install** to put the .so to your php so library. However this may not add the configuration file for you automatically. So the alternative way is using following steps:
  - run php -i, you will find two field **extension_dir** and  **Scan this dir for additional .ini files**, you can find it using php -i | grep "extension_dir" or grep "dir for additional .ini files".
  - put mysqlnd_rd.so under extension_dir
  - under directory for additional .ini files, you will find the ini files for the common used modules, e.g. 10-mysqlnd.ini for mysqlnd, 20-mysqli.ini for mysqli. Create a new ini file for mysqlnd_rd here. **Make sure the alphabet order of the name is after that of mysqnld**, since the modules are loaded according to the name order of the ini files. E.g. if mysqlnd ini is with name 10-mysqlnd.ini,then name the ini as 20-mysqlnd-rd.ini. In the ini file, add the following two lines:
      - extension=mysqlnd_rd
      - mysqlnd_rd.enabled = on  ; you can also set this to off to disable redirection


## Step to build on Windows
#### Set up 
There need more steps to build the extension under Windows, and it willl use the php-sdk-binary-tools developed for help. The detailed step is decribed on https://wiki.php.net/internals/windows/stepbystepbuild_sdk_2. And it is repeated here:
* If compiling PHP 7.2+:
  Install Visual Studio 2017
  If compiling PHP 7.4+:
  Install Visual Studio 2019

* If compiling PHP 7.2+ open either the **“VS2017 x64 Native Tools Command Prompt”** or the **“VS2017 x86 Native Tools Command Prompt”**.
* Fetch the latest stable SDK tag from https://github.com/Microsoft/php-sdk-binary-tools
  The new PHP SDK is required, when building PHP 7.2+
  Read the PHP SDK specific notes on the Github repository page

* Run the phpsdk_buildtree batch script which will create the desired directory structure: 
  - phpsdk_buildtree phpdev
* The phpsdk_buildtree script will create the path according to the currently VC++ version used and switch into the newly created directory. 
  cd to C:\php-sdk\phpdev\vX##\x##, where:
    vX## is the compiler version you are using (eq vc14 or vs16)
    x## is your architecture (x86 or x64)
    For example: C:\php-sdk\phpdev\vc15\x64\  for PHP7.2+
* Under the folder, git clone php source code from https://github.com/php/php-src.git  Fecth the related version, e.g. PHP-7.2.20
* use the PHP SDK tools to fetch the suitable dependencies automatically by calling 
  - phpsdk_deps -u
* Extract mysqlnd_rd code https://github.com/microsoft/php-mysqlnd from ext folder to the php-source-code-folder\ext\, e.g. to C:\php-sdk\phpdev\vc15\x64\php-src\ext. Or create another folder with name "pecl" which is in parallel with php source, e.g. C:\php-sdk\phpdev\vc15\x64\pecl, and extract the code there.
After this, the code directory should look like C:\php-sdk\phpdev\vc15\x64\php-src\ext\mysqlnd_rd, or C:\php-sdk\phpdev\vc15\x64\pecl\mysqlnd_rd
  
#### Compile
* Invoke the starter script, for example for Visual Studio 2017 64-bit build, invoke     
  - phpsdk-vc15-x64.bat
* run **buildconf**
* run **configure.bat --disable-all --enable-cli --with-mysqlnd --enable-mysqlnd_rd=shared**
* run **nmake**
* after that, you will find a dll under .\x64\x64\Release_TS\ with name **php_mysqlnd_rd.dll**, this is the target library.

#### Install
* Use php -i to find the ini to find the extension_dir, copy php_mysqlnd_rd.dll there
* Find the ini file location, modify the ini file with the following extra lines:
    - extension=mysqlnd_rd
    - [mysqlnd_rd]
    - mysqlnd_rd.enabled = on


## Test
* Currently redirection is only possible when the connection is via ssl, and it need that the redirection feature switch is enabled on server side. Following is a snippet to test connection with redirection:

```php
  echo "mysqlnd_rd.enabled: ", ini_get("mysqlnd_rd.enabled") == true?"On":"Off", "\n";
  $db = mysqli_init();
  $link = mysqli_real_connect ($db, 'your-hostname-with-redirection-enabled', 'user@host', 'password', "db", 3306, NULL, MYSQLI_CLIENT_SSL);
  if (!$link) {
     die ('Connect error (' . mysqli_connect_errno() . '): ' . mysqli_connect_error() . "\n");
  }
  else {
    echo $db->host_info, "\n"; //if redrection succeed, you will find the host_info differ from your-hostname used to connect
    $res = $db->query('SHOW TABLES;'); //test query with the connection
    print_r ($res);
	$db->close();
  }
```