# CryptoFolio installation

* You need accounts on coinmarketcap.com and optionally on cryptoapis.com

* Install php 7.4, apache, mySQL, composer, node.js, npm, redis, supervisor
* Bind a domain to apache, set up SSL keys
* Installation
  * Clone the repo (in the home directory): `git clone https://github.com/Arodnk88/CryptoFolio.git`
  * Modify apache `DocumentRoot` to the folder `public` in the cloned repository
  * Change directory to the repo and copy config from example:
    ```cd /path/to/project
    cp .env.example .env
    ```
  * Replace `APP_ENV=local` with `APP_ENV=production` in `.env` file
  * Install composer: `composer install`
  * And generate keys: `php artisan key:generate`
  * Edit config file `.env` and setup database, Jabber and redis accounts in this varriables:
    ```DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=
    DB_USERNAME=
    DB_PASSWORD=

    JABBER_HOST=
    JABBER_PORT=
    JABBER_USERNAME=
    JABBER_PASSWORD=
    JABBER_RESOURCE="CF"
    JABBER_USE_TLS=true
    JABBER_LOG=false

    REDIS_HOST=127.0.0.1
    REDIS_PASSWORD=null
    REDIS_PORT=6379  
    ```
  * Migrate DB: `php artisan migrate`
  * Create Admin user: `php artisan tinker` and type
    ```$user = new User([
      'login' => 'ADMIN_LOGIN',
      'password' => Hash::make('ADMIN_PASS'),
      'user_id' => Str::uuid(),
      'is_active' => true,
      'is_admin' => true,
      'api_token' => Str::uuid(),
    ])
    $user->save()
    ```
  * If there is no error on the site write `APP_DEBUG=false` to `.env` file
  * Run `php artisan config:clear`
  * Run `crontab -e` and add an entry: `* * * * * cd /path/to/project && php artisan schedule:run >> /dev/null 2>&1`
  * Configure supervisor `sudo nano /etc/supervisor/conf.d/jabber.conf`:
    ```[program:jabber]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/NEW_USER/CryptoFolio/artisan jabber:start
    autostart=true
    autorestart=true
    redirect_stderr=true
    user=www-data
    stdout_logfile=/home/NEW_USER/CryptoFolio/storage/jabber.log
    stdout_logfile_maxbytes=10MB
    logfile_backups=10
    ```
  * Reload supervisor: `supervisorctl reread && supervisorctl update`



# CryptoFolio tweaks

* coinmarketcap.com
  * Recommended plan - Standart. If you take a tariff plan at cheaper - pre-loading of historical courses will not be available
  * You can change the `$schedule->command('rate:refresh')->everyMinute();` string in the file `app/Console/Commands/Kernel.php` on:
    a) `$schedule->command('rate:refresh')->everyFiveMinutes();` - when you first start, not to spend in advance all API loans.
    b) `$schedule->command('rate:refresh')->everyThirtyMinutes();` - if the you want to use free plan
    c) Comment this string to manually update currency rates. To update, you must run `php artisan rate:refresh` in the console from the project directory. 

* cryptoapis.com
  * You can skip this step if you do not need the RealTime function of balance updates!
  * Recommended plan - Advanced. Free Plan, unlike coinmarketcap, it is impossible to use (wallet balances on free plan can only receive from Testnet networks)
  * It is necessary to calculate how much credits you will spend daily by the formula: Number of wallets * Frequency of updates (by default every 10 minutes), so 1 wallet will spend 144 credits daily.
  * In the `config/services.php` file, change `'network' => 'ropsten'` to `'network' => 'mainnet'`, so that the application receives data from the main blockchain.
  * You can change the update frequency in the `app/Console/Commands/Kernel.php` file. For example: substitution from `$schedule->command('wallets:refresh')->everyTenMinutes();` to `$schedule->command('wallets:refresh')->hourly()` will change the update interval from once per 10 minutes to once per hour
