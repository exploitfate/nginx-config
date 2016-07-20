#Installation and config guide

##Installation

####1. Install mysql, nginx, php-fpm, memcache, php
> Hint: [What's New in MySQL 5.6](http://dev.mysql.com/tech-resources/articles/whats-new-in-mysql-5.6.html) 



    sudo apt-get update && sudo apt-get upgrade


####2. Install nginx



    sudo apt-get install nginx


####3. Install php



    sudo apt-get install php7.0 php7.0-cli php7.0-fpm php7.0-mbstring php7.0-mcrypt php7.0-intl php-imagick php7.0-curl && sudo phpenmod mcrypt


####4. Install MySQL (optional)
    


    sudo apt-get install mysql-server mysql-client php7.0-mysql


####5. Install redis (optional)



    sudo apt-get install redis-server php-redis


####3. Install Memcached



    sudo apt-get install memcached php-memcache php-memcached


####3. Install mongodb (optional)



    sudo apt-get install mongodb php-mongodb


####4. Install tools and utility



    sudo apt-get install zsh git htop curl zip unzip

##Configuration

###User

####1. Create user `web`




    sudo mkdir /web /web/www
    sudo useradd -d /web/www web
    sudo chown -R web:web /web/www
    


> Hint: Add a existing user to existing group



    sudo usermod -a -G group user
    


####2. Create project dirs


    sudo su web    
    mkdir /web/www/project.tld /web/www/project.tld/log /web/www/project.tld/html

### php-fpm

####1. Create php mods file `sudo nano /etc/php/7.0/mods-available/nginx.ini`



    cgi.fix_pathinfo= 0
    expose_php = Off

####2. Create php mods file `sudo nano /etc/php/7.0/mods-available/php-override.ini`


    memory_limit = 512M
    post_max_size = 128M
    upload_max_filesize = 128M

####3. Create php mods file `sudo nano /etc/php/7.0/mods-available/realpath-cache.ini`


    realpath_cache_size = 1M
    realpath_cache_ttl = 3600

####4. Enable this mods


    sudo phpenmod nginx php-override realpath-cache


####5. Update file `sudo nano /etc/php/7.0/fpm/pool.d/www.conf` set


    user = web
    group = web
    listen.owner = web
    listen.group = web

####6. Restart php-fpm


    sudo service php7.0-fpm restart

###mysql
####1. Improve MySQL Installation Security (for product environment).
#####Run the script called "mysql_secure_installation"

 
 
    mysql_secure_installation

####2. Login to mysql console.



    mysql -u root -p
> Hint: for reset mysql root password use


    sudo dpkg-reconfigure mysql-server-5.6

####3. Create mysql project db and user

#####Create mysql project db
 
 
    CREATE DATABASE `projectdatabase`  CHARACTER SET utf8 COLLATE utf8_general_ci;

#####Create mysql project user


    CREATE USER 'projectuser'@'localhost' IDENTIFIED BY 'password';

#####Grant privileges
    
    
    GRANT ALL PRIVILEGES ON `projectdatabase`. * TO 'projectuser'@'localhost';

#####Reload all the privileges.
    
    
    
    FLUSH PRIVILEGES;

###nginx

####1. Update file `sudo nano /etc/nginx/nginx.conf` set



    user web

####2. Disable default site (optional)


    sudo rm /etc/nginx/sites-enabled/default
    

####3. Create config helpers

#####- `sudo nano /etc/nginx/cloudflare.conf`

```
##CloudFlare IP Forwarding
set_real_ip_from 199.27.128.0/21;
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 188.114.96.0/20; 
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 104.16.0.0/12;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
real_ip_header CF-Connecting-IP;
```

#####- `sudo nano /etc/nginx/conf.d/charset.conf`

```
#Specify a charset
charset utf-8;

client_max_body_size 128M;
server_tokens off;

merge_slashes off;
```

#####- `sudo nano /etc/nginx/conf.d/gzip.conf`

```
# Gzip Settings
#gzip on;               # enabled by default
#gzip_disable "msie6";  # enabled by default
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_types 
    application/atom+xml
    application/javascript
    application/x-javascript
    application/json
    application/ld+json
    application/manifest+json
    application/rss+xml
    application/vnd.geo+json
    application/vnd.ms-fontobject
    application/x-font-ttf
    application/x-web-app-manifest+json
    application/xhtml+xml
    application/xml
    font/opentype
    image/bmp
    image/svg+xml
    image/x-icon
    text/cache-manifest
    text/css
    text/plain
    text/vcard
    text/vnd.rim.location.xloc
    text/vtt
    text/x-component
    text/x-cross-domain-policy;
```

#####- `sudo nano /etc/nginx/expires.conf`

```
# Expire rules for static content

# No default expire rule. This config mirrors that of apache as outlined in the
# html5-boilerplate .htaccess file. However, nginx applies rules by location,
# the apache rules are defined by type. A consequence of this difference is that
# if you use no file extension in the url and serve html, with apache you get an
# expire time of 0s, with nginx you'd get an expire header of one month in the
# future (if the default expire rule is 1 month). Therefore, do not use a
# default expire rule with nginx unless your site is completely static

# cache.appcache, your document html and data
location ~* \.(?:manifest|appcache|html?|xml|json)$ {
  expires -1;
  #access_log logs/static.log;
}

# Feed
location ~* \.(?:rss|atom)$ {
  expires 1h;
  add_header Cache-Control "public";
}

# Media: images, icons, video, audio, HTC
location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc|webp|mp3)$ {
  expires 1M;
  access_log off;
  add_header Cache-Control "public";
}

# CSS and Javascript
location ~* \.(?:css|js)$ {
  expires 1y;
  access_log off;
  add_header Cache-Control "public";
}
# favicon disable logs
location = /favicon.ico {
  access_log off;
  log_not_found off;
}
```

#####- `sudo nano /etc/nginx/cross-domain-fonts.conf`

```
# Cross domain webfont access
location ~* \.(?:ttf|ttc|otf|eot|woff|woff2)$ {
    # Also, set cache rules for webfonts.
    #
    # See http://wiki.nginx.org/HttpCoreModule#location
    # And https://github.com/h5bp/server-configs/issues/85
    # And https://github.com/h5bp/server-configs/issues/86
    expires 1M;
    access_log off;
    add_header Cache-Control "public";
}
```

#####- `sudo nano /etc/nginx/protect-system-files.conf`

```
# Prevent clients from accessing hidden files (starting with a dot)
# This is particularly important if you store .htpasswd files in the site hierarchy
location ~* (?:^|/)\. {
    deny all;
}

# Prevent clients from accessing to backup/config/source files
location ~* (?:\.(?:bak|config|sql|fla|psd|ini|log|sh|inc|swp|dist|md)|~)$ {
    deny all;
}
```

#####- SKIP THIS `sudo nano /etc/nginx/deny-bots.conf`

```
if ($http_user_agent ~* (bingbot|Googlebot|MJ12bot|Riddlerbot|sogou\ spider|YandexBot|Baiduspider|ia_archiver|UptimeRobot|Yahoo) ) {
    return 403;
}
if ($http_user_agent ~* (TurnitinBot|XoviBot|Daumoa|DotBot|AhrefsBot|Exabot|CCBot|ZumBot|OpenWebSpider|SearchmetricsBot|SMTBot) ) {
    return 403;
}
if ($http_user_agent ~* (MSNBot|stq_bot|SeznamBot|archive.org_bot|gocrawl|proximic|Genieo\ Web\ filter|EasouSpider|meanpathbot) ) {
    return 403;
}
if ($http_user_agent ~* (JamesBOT|Sitedomain-Bot|ichiro|contentDetection|spbot|ChangeDetection|coccoc|MetaJobBot|OrangeBot|Spinn3r) ) {
    return 403;
}
if ($http_user_agent ~* (Vagabondo|GrapeshotCrawler|WorldBrewBot|NaverBot|Mail.Ru\ bot|sistrix|SemrushBot|WeViKaBot|BacklinkCrawler) ) {
    return 403;
}
if ($http_user_agent ~* (ShopWiki|WeSEE|GigablastOpenSource|Crawler4j|bitlybot|ProductoDownloadUrlBot/|Qualidator.com\ Bot|BLEXBot) ) {
    return 403;
}
if ($http_user_agent ~* (rogerbot|Kraken|FlipboardProxy|Lipperhey\ Spider|Jyxobot|Job\ Roboter\ Spider|Speedy\ Spider|ImplisenseBot) ) {
    return 403;
}
if ($http_user_agent ~* (MojeekBot|Wotbox|linkdexbot|voltron|Seobility|aiHitBot|yacybot|ICC-Crawler|ShowyouBot|psbot|iCjobs) ) {
    return 403;
}
if ($http_user_agent ~* (SEOkicks-Robot|WBSearchBot|oBot|EuripBot|Alexabot|magpie-crawler|Woko|omgilibot|NetcraftSurveyAgent) ) {
    return 403;
}
if ($http_user_agent ~* (UASlinkChecker|A6-Indexer|Qirina\ Hurdler|memoryBot|Netseer|JobdiggerSpider|AntBot|uMBot|MergeFlow-PageReader) ) {
    return 403;
}
if ($http_user_agent ~* (bnf.fr_bot|netEstate\ Crawler|007AC9|bl.uk_lddc_bot|Blekkobot|VoilaBot|SEOdiver|NextGenSearchBot|TinEye) ) {
    return 403;
}
if ($http_user_agent ~* (AboutUsBot|bot-pge.chlooe.com|trendictionbot|HubSpot\ Crawler|DomainAppender|linguatools|YioopBot|YoudaoBot) ) {
    return 403;
}
if ($http_user_agent ~* (YamanaLab-bot|kinshoo|bixocrawler|ThumbSniper|Dlvr.it/1.0|MetaGeneratorCrawler|Plukkie|OpenCalaisSemanticProxy) ) {
    return 403;
}
if ($http_user_agent ~* (SBSearch|Aboundexbot|Impressumscrawler|Infohelfer|R6\ bot|LivelapBot/0.2|URLAppendBot|\ Scrubby|SEOENGBot) ) {
    return 403;
}
if ($http_user_agent ~* (Experibot|UnisterBot|FreeWebMonitoring\ SiteChecker|LinkWalker|Browsershots|BDCbot|IstellaBot|SpiderLing) ) {
    return 403;
}
if ($http_user_agent ~* (CareerBot|SEOCentro\ bot|BDFetch|Online\ Domain\ Tools|WordPress.com\ mShots|WebTarantula.com\ Crawler|NerdyBot) ) {
    return 403;
}
if ($http_user_agent ~* (CrazyWebCrawler-Spider|Scrapy|IntegromeDB|WebThumbnail|Panscient\ web\ crawler|Cliqzbot|dlvr.it) ) {
    return 403;
}
if ($http_user_agent ~* (HypeStat|AddThis.com|datagnionbot|MiaDev|Pinterest|MixBot|SurcentroBot|webmastercoffee|Peeplo\ Screenshot\ Bot) ) {
    return 403;
}
if ($http_user_agent ~* (ZeerchBot|AMZNKAssocBot|SCFCrawler|idmarch|SputnikBot|LinkedInBot|seegnifybot|dlcbot|thumbshots-de-Bot|NalezenCzBot) ) {
    return 403;
}
if ($http_user_agent ~* (pr-cy.ru\ Screenshot\ Bot|LoadImpactPageAnalyzer|AskQuickly|STINGbot|WebCorp|BUbiNG|Motoricerca-Robots.txt-Checker) ) {
    return 403;
}
if ($http_user_agent ~* (socialbm_bot|HubSpot\ Connect|Company\ News\ Search\ engine|x28-job-bot|COMODOSpider|EveryoneSocialBot|Ezooms) ) {
    return 403;
}
if ($http_user_agent ~* (Symfony\ Spider|Iframely|KrOWLer|Twingly\ Recon|Robots_Tester|FacebookExternalHit|Arachnophilia|eCommerceBot) ) {
    return 403;
}
if ($http_user_agent ~* (emefgebot|Nuhk|Najdi.si|SecurityResearchBot|CloudServerMarketSpider|YYSpider|200PleaseBot|Steeler|nekstbot) ) {
    return 403;
}
if ($http_user_agent ~* (360Spider|LoadTimeBot|webinatorbot|Leikibot|musobot|search.KumKie.com|Nigma.ru|CompSpyBot|SeoCheckBot|hawkReader) ) {
    return 403;
}
if ($http_user_agent ~* (PercolateCrawler|Butterfly|008|Slackbot|Falconsbot|SSL-Crawler|Embedly|backlink-check.de|adressendeutschland.de) ) {
    return 403;
}
if ($http_user_agent ~* (XRL|IdeelaborPlagiaat|SiteCondor|Web-Monitoring|Vedma|parsijoo|GarlikCrawler|FyberSpider|classbot|Feedly|WebCookies) ) {
    return 403;
}
if ($http_user_agent ~* (CloudFlare-AlwaysOnline|Readability|suggybot|CatchBot|Jabse.com\ Crawler|woriobot|ExB\ Language\ Crawler|kulturarw) ) {
    return 403;
}
if ($http_user_agent ~* (BrainbruBot|KomodiaBot|IXEbot|CMS\ Crawler|immediatenet\ thumbnails|Shareaholicbot|Qualidator.com\ SiteAnalyzer\ 1.0) ) {
    return 403;
}
if ($http_user_agent ~* (BegunAdvertising|LuminateBot|linkdex.com|Curious\ George|Fetch-Guess|alexa\ site\ audit|AraBot|CliqzBot|findlinks) ) {
    return 403;
}
if ($http_user_agent ~* (CCResearchBot|Semantifire|LinkAider|Zookabot|ScreenerBot\ Crawler|PaperLiBot|QuerySeekerSpider|Crowsnest|UnwindFetchor) ) {
    return 403;
}
if ($http_user_agent ~* (MetaURI\ API|AcoonBot|Gigabot|firmilybot|Sosospider|OpenindexSpider|MetaHeadersBot|Strokebot|GeliyooBot|ownCloud\ Server\ Crawler) ) {
    return 403;
}
if ($http_user_agent ~* (CirrusExplorer|ProCogSEOBot|Open\ Web\ Analytics\ Bot|RyzeCrawler|discoverybot|crawler\ for\ netopian|ADmantX\ Platform\ Semantic\ Analyzer) ) {
    return 403;
}
if ($http_user_agent ~* (Linguee\ Bot|SolomonoBot|Grahambot|Automattic\ Analytics\ Crawler|PiplBot|FlightDeckReportsBot|fastbot\ crawler|4seohuntBot) ) {
    return 403;
}
if ($http_user_agent ~* (Updownerbot|JikeSpider|NLNZ_IAHarvester|wsAnalyzer|YodaoBot|Esribot|Thumbshots.ru|BlogPulse|bot.wsowner.com|wscheck.com) ) {
    return 403;
}
if ($http_user_agent ~* (Qseero|drupact|HuaweiSymantecSpider|PagePeeker|HomeTags|facebookplatform|Pixray-Seeker|MeMoNewsBot|ProCogBot|WillyBot) ) {
    return 403;
}
if ($http_user_agent ~* (peerindex|MLBot|WebNL|Peepowbot|Semager|MIA\ Bot|heritrix|Eurobot|DripfeedBot|Whoismindbot|Bad-Neighborhood|Hailoobot) ) {
    return 403;
}
if ($http_user_agent ~* (akula|MetamojiCrawler|Page2RSS|EasyBib\ AutoCite|NerdByNature.Bot|EventGuruBot|quickobot|gonzo|Influencebot|MSRBOT|Ronzoobot) ) {
    return 403;
}
if ($http_user_agent ~* (ScoutJet|Twikle|SWEBot|RADaR-Bot|DCPbot|Castabot|imbot|EdisterBot|WASALive-Bot|Accelobot|PostPost|factbot|Setoozbot|biwec) ) {
    return 403;
}
if ($http_user_agent ~* (Search17Bot|Lijit|JUST-CRAWLER|Apercite|pmoz.info\ ODP\ link\ checker|LemurWebCrawler|Covario-IDS|Holmes|RankurBot|AdsBot-Google) ) {
    return 403;
}
if ($http_user_agent ~* (envolk|Ask\ Jeeves/Teoma|LexxeBot|StackRambler|Abrave\ Spider|EvriNid|arachnode.net|CamontSpider|wikiwix-bot|Nymesis) ) {
    return 403;
}
if ($http_user_agent ~* (trendictionbot|SEODat|SygolBot|Snapbot|ZookaBot|CligooRobot|cityreview|nworm|SBIder|TwengaBot|Dot\ TK\ -\ spider|ParchBot) ) {
    return 403;
}
if ($http_user_agent ~* (Peew|YRSpider|Urlfilebot\ (Urlbot)|Gaisbot|WatchMouse|Tagoobot|WebWatch/Robot_txtChecker|urlfan-bot|StatoolsBot|page_verifier) ) {
    return 403;
}
if ($http_user_agent ~* (SSLBot|SAI\ Crawler|DomainDB|WMCAI_robot|voyager|copyright\ sheriff|Ocelli|Twiceler|amibot|abby|NetResearchServer|VideoSurf_bot) ) {
    return 403;
}
if ($http_user_agent ~* (XML\ Sitemaps\ Generator|BlinkaCrawler|nodestackbot|Pompos|taptubot|BabalooSpider|Yaanb|Girafabot|livedoor\ ScreenShot|eCairn-Grabber) ) {
    return 403;
}
if ($http_user_agent ~* (FauBot|Toread-Crawler|Setoozbot|MetaURI|L.webis|Web-sniffer|FairShare|Ruky-Roboter|ThumbShots-Bot|BotOnParade|Amagit.COM|HatenaScreenshot) ) {
    return 403;
}
if ($http_user_agent ~* (HolmesBot|dotSemantic|Karneval-Bot|AportWorm|XmarksFetch|FeedFinder/bloggz.se|CorpusCrawler|Willow\ Internet\ Crawler|OrgbyBot|GingerCrawler) ) {
    return 403;
}
if ($http_user_agent ~* (pingdom.com_bot|Nutch|baypup|Mp3Bot|192.comAgent|Surphace\ Scout|WikioFeedBot|Szukacz|DBLBot|Thumbnail.CZ\ robot|LinguaBot|GurujiBot) ) {
    return 403;
}
if ($http_user_agent ~* (Charlotte|50.nu|SanszBot|moba-crawler|HeartRails_Capture|SurveyBot|MnoGoSearch|smart.apnoti.com\ Robot|Topicbot|JadynAveBot|OsObot) ) {
    return 403;
}
if ($http_user_agent ~* (WebImages|WinWebBot|Scooter|Scarlett|GOFORITBOT|DKIMRepBot|Yanga|DNS-Digger-Explorer|Robozilla|YowedoBot|botmobi|Fooooo_Web_Video_Crawl) ) {
    return 403;
}
if ($http_user_agent ~* (UptimeDog|Metaspinner/0.01|Touche|RSSMicro.com\ RSS/Atom\ Feed\ Robot|SniffRSS|Kalooga|FeedCatBot|WebRankSpider|Flatland\ Industries\ Web\ Spider) ) {
    return 403;
}
if ($http_user_agent ~* (DealGates\ Bot|Link\ Valet\ Online|Shelob|Technoratibot|Flocke\ bot|FollowSite\ Bot|Visbot) ) {
    return 403;
}
```

#####- `sudo nano /etc/nginx/php-fpm-bootstrap.conf`

```
set           $bootstrap  index.php;
index         $bootstrap;

location / {
        try_files $uri $uri/ /$bootstrap$is_args$args;
}

location ~ \.php$ {
        try_files $uri =404;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index               $bootstrap;

        # Connect to php-fpm via socket
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;

        fastcgi_connect_timeout     30s;
        fastcgi_read_timeout        30s;
        fastcgi_send_timeout        60s;
        fastcgi_ignore_client_abort on;
        fastcgi_pass_header         "X-Accel-Expires";

        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_param  PATH_INFO        $fastcgi_path_info;
        fastcgi_param  HTTP_REFERER     $http_referer;
        include fastcgi_params;
}
```

#####- `sudo nano /etc/nginx/php-fpm-html.conf`

```
set           $bootstrap    index.php;
index         $bootstrap    index.html    index.htm;

error_page    404     /404.php;

location / {
        try_files $uri $uri/ =404;
}

location ~ \.php$ {
        try_files $uri =404;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index               $bootstrap;

        # Connect to php-fpm via socket
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;

        fastcgi_connect_timeout     30s;
        fastcgi_read_timeout        30s;
        fastcgi_send_timeout        60s;
        fastcgi_ignore_client_abort on;
        fastcgi_pass_header         "X-Accel-Expires";

        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_param  PATH_INFO        $fastcgi_path_info;
        fastcgi_param  HTTP_REFERER     $http_referer;
        include fastcgi_params;
}
```


####4. Create project site config `sudo nano /etc/nginx/sites-available/project.tld`

```
##
# For product environment uncomment "include expires.conf" and comment "include deny-bots.conf;"
##

# frontend www redirect
server {
        server_name www.project.tld;
        listen 80;
        return 301 $scheme://project.tld$request_uri;
}

# frontend
server {
        server_name project.tld;
        listen 80;

        root          /web/www/project.tld/html/frontend/web;

        access_log    /web/www/project.tld/log/access.log combined buffer=50k;
        error_log     /web/www/project.tld/log/error.log notice;


        # Remove all double or triple slashes
        #rewrite ^(.*)//+(.*)$ $1/$2 permanent;
        
        # Avoid index.php in URI 
        #if ($request_uri ~* "^(.*/)index\.php(?:.*)$") {
        #    return 301 $1$is_args$args;
        #}
        
        # CloudFlare restore original visitor IP
        #include cloudflare.conf;
        #include deny-bots.conf;
        include php-fpm-bootstrap.conf; # Replace with php-fpm-html.conf for no CMS project
        
        include cross-domain-fonts.conf;
        include protect-system-files.conf;
        
        
        # Uncomment this on product environment
        #include expires.conf;
}

# backend
server {
        server_name backend.project.tld;
        listen 80;

        root          /web/www/project.tld/html/backend/web;

        access_log    /web/www/project.tld/log/access.backend.log combined buffer=50k;
        error_log     /web/www/project.tld/log/error.backend.log notice;
        
        # CloudFlare restore original visitor IP
        #include cloudflare.conf;
        #include deny-bots.conf;
        include php-fpm-bootstrap.conf; # Replace with php-fpm-html.conf for no CMS project
        
        include cross-domain-fonts.conf;
        include protect-system-files.conf;
        
        
        # Uncomment this on product environment
        #include expires.conf;
}
```


####5. Enable project.tld site config


    sudo ln -s /etc/nginx/sites-available/project.tld /etc/nginx/sites-enabled/

####6. Check config


    sudo nginx -t


####7. Enable log rotation for project.tld nginx logs ` sudo nano /etc/logrotate.d/project.tld`
> Hint: For testing log rotation run command `sudo logrotate -f /etc/logrotate.d/project.tld` 



```
/web/www/project.tld/log/*.log {
 daily
 missingok
 rotate 60
 compress
 delaycompress
 notifempty
 create 0640 web web
 su web web
 sharedscripts
 prerotate
  if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
   run-parts /etc/logrotate.d/httpd-prerotate; \
  fi \
 endscript
 postrotate
  [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
 endscript
}
```

####8. Restart nginx


    sudo service nginx restart


###Clone project

####1. Generate deployment ssh key



    sudo su web
    ssh-keygen -t rsa -b 4096 -C "deploy@project.tld"
 and add `project.tld` to the repo deployment key


    cat ~/.ssh/id_rsa.pub


####2. Clone project


    cd ~/project.tld/
    git clone git@bitbucket.org:gitaccount/project.git html

####3. Install composer.phar



    cd ~/project.tld/html/
    curl -s http://getcomposer.org/installer | php

####4. Add composer assets plugin


    php composer.phar global require "fxp/composer-asset-plugin:@dev"

####5. Install packages and init project environment


    cd ~/project.tld/html/
    php composer.phar install
    ./init

####6. Update config/main-local.php files


####7. Run migrations



    ./yii migrate

