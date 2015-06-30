#Installation and config guide

##Installation

####1. Install mysql, nginx, php-fpm, memcache, php
> Hint: [What's New in MySQL 5.6](http://dev.mysql.com/tech-resources/articles/whats-new-in-mysql-5.6.html) 



    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install nginx memcached mysql-server-core-5.6 mysql-server-5.6 mysql-client-5.6 mysql-client-core-5.6 mysql-common-5.6
    sudo apt-get install php5-fpm php5 php5-cli php5-curl php5-mcrypt php5-intl php5-mysql php5-memcache php5-memcached php5-gd php-apc
    sudo php5enmod mcrypt
    

####2. Install redis (optional)



    sudo apt-get install redis-server php5-redis


####3. Install mongodb (optional)



    sudo apt-get install mongodb php5-mongo


####4. Install tools and utility



    sudo apt-get install git htop curl

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

####1. Create php mods file `/etc/php5/mods-available/nginx.ini`



    cgi.fix_pathinfo= 0
    expose_php = Off

####2. Create php mods file `/etc/php5/mods-available/php-override.ini`


    memory_limit = 512M
    post_max_size = 128M
    upload_max_filesize = 128M

####3. Enable this mods


    sudo php5enmod nginx
    sudo php5enmod php-override


####4. Update file `/etc/php5/fpm/pool.d/www.conf` set


    user = web
    group = web
    listen.owner = web
    listen.group = web

####5. Restart php-fpm


    sudo service php5-fpm restart

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

####1. Update file `/etc/nginx/nginx.conf` set



    user web

####2. Disable default site


    sudo rm /etc/nginx/sites-enabled/default
    

####3. Create config helpers

#####- `/etc/nginx/conf.d/charset.conf`

```
#Specify a charset
charset utf-8;

client_max_body_size 128M;
server_tokens off;
```

#####- `/etc/nginx/conf.d/gzip.conf`

```
# Gzip Settings
#gzip on;               # enabled by default
#gzip_disable "msie6";  # enabled by default
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
```

#####- `/etc/nginx/expires.conf`

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

#####- `/etc/nginx/cross-domain-fonts.conf`

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

#####- `/etc/nginx/protect-system-files.conf`

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

#####- `/etc/nginx/deny-bots.conf`

```
map $http_user_agent $limit_bots {
	default 0;
	~*(bingbot|Googlebot|MJ12bot|Riddlerbot|sogou\ spider|YandexBot|Baiduspider|ia_archiver|UptimeRobot|Yahoo) 1;
	~*(TurnitinBot|XoviBot|Daumoa|DotBot|AhrefsBot|Exabot|CCBot|ZumBot|OpenWebSpider|SearchmetricsBot|SMTBot) 1;
	~*(MSNBot|stq_bot|SeznamBot|archive.org_bot|gocrawl|proximic|Genieo\ Web\ filter|EasouSpider|meanpathbot) 1;
	~*(JamesBOT|Sitedomain-Bot|ichiro|contentDetection|spbot|ChangeDetection|coccoc|MetaJobBot|OrangeBot|Spinn3r) 1;
	~*(Vagabondo|GrapeshotCrawler|WorldBrewBot|NaverBot|Mail.Ru\ bot|sistrix|SemrushBot|WeViKaBot|BacklinkCrawler) 1;
	~*(ShopWiki|WeSEE|GigablastOpenSource|Crawler4j|bitlybot|ProductoDownloadUrlBot/|Qualidator.com\ Bot|BLEXBot) 1;
	~*(rogerbot|Kraken|FlipboardProxy|Lipperhey\ Spider|Jyxobot|Job\ Roboter\ Spider|Speedy\ Spider|ImplisenseBot) 1;
	~*(MojeekBot|Wotbox|linkdexbot|voltron|Seobility|aiHitBot|yacybot|ICC-Crawler|ShowyouBot|psbot|iCjobs) 1;
	~*(SEOkicks-Robot|WBSearchBot|oBot|EuripBot|Alexabot|magpie-crawler|Woko|omgilibot|NetcraftSurveyAgent) 1;
	~*(UASlinkChecker|A6-Indexer|Qirina\ Hurdler|memoryBot|Netseer|JobdiggerSpider|AntBot|uMBot|MergeFlow-PageReader) 1;
	~*(bnf.fr_bot|netEstate\ Crawler|007AC9|bl.uk_lddc_bot|Blekkobot|VoilaBot|SEOdiver|NextGenSearchBot|TinEye) 1;
	~*(AboutUsBot|bot-pge.chlooe.com|trendictionbot|HubSpot\ Crawler|DomainAppender|linguatools|YioopBot|YoudaoBot) 1;
	~*(YamanaLab-bot|kinshoo|bixocrawler|ThumbSniper|Dlvr.it/1.0|MetaGeneratorCrawler|Plukkie|OpenCalaisSemanticProxy) 1;
	~*(SBSearch|Aboundexbot|Impressumscrawler|Infohelfer|R6\ bot|LivelapBot/0.2|URLAppendBot|\ Scrubby|SEOENGBot) 1;
	~*(Experibot|UnisterBot|FreeWebMonitoring\ SiteChecker|LinkWalker|Browsershots|BDCbot|IstellaBot|SpiderLing) 1;
	~*(CareerBot|SEOCentro\ bot|BDFetch|Online\ Domain\ Tools|WordPress.com\ mShots|WebTarantula.com\ Crawler|NerdyBot) 1;
	~*(CrazyWebCrawler-Spider|Scrapy|IntegromeDB|WebThumbnail|Panscient\ web\ crawler|Cliqzbot|dlvr.it) 1;
	~*(HypeStat|AddThis.com|datagnionbot|MiaDev|Pinterest|MixBot|SurcentroBot|webmastercoffee|Peeplo\ Screenshot\ Bot) 1;
	~*(ZeerchBot|AMZNKAssocBot|SCFCrawler|idmarch|SputnikBot|LinkedInBot|seegnifybot|dlcbot|thumbshots-de-Bot|NalezenCzBot) 1;
	~*(pr-cy.ru\ Screenshot\ Bot|LoadImpactPageAnalyzer|AskQuickly|STINGbot|WebCorp|BUbiNG|Motoricerca-Robots.txt-Checker) 1;
	~*(socialbm_bot|HubSpot\ Connect|Company\ News\ Search\ engine|x28-job-bot|COMODOSpider|EveryoneSocialBot|Ezooms) 1;
	~*(Symfony\ Spider|Iframely|KrOWLer|Twingly\ Recon|Robots_Tester|FacebookExternalHit|Arachnophilia|eCommerceBot) 1;
	~*(emefgebot|Nuhk|Najdi.si|SecurityResearchBot|CloudServerMarketSpider|YYSpider|200PleaseBot|Steeler|nekstbot) 1;
	~*(360Spider|LoadTimeBot|webinatorbot|Leikibot|musobot|search.KumKie.com|Nigma.ru|CompSpyBot|SeoCheckBot|hawkReader) 1;
	~*(PercolateCrawler|Butterfly|008|Slackbot|Falconsbot|SSL-Crawler|Embedly|backlink-check.de|adressendeutschland.de) 1;
	~*(XRL|IdeelaborPlagiaat|SiteCondor|Web-Monitoring|Vedma|parsijoo|GarlikCrawler|FyberSpider|classbot|Feedly|WebCookies) 1;
	~*(CloudFlare-AlwaysOnline|Readability|suggybot|CatchBot|Jabse.com\ Crawler|woriobot|ExB\ Language\ Crawler|kulturarw) 1;
	~*(BrainbruBot|KomodiaBot|IXEbot|CMS\ Crawler|immediatenet\ thumbnails|Shareaholicbot|Qualidator.com\ SiteAnalyzer\ 1.0) 1;
	~*(BegunAdvertising|LuminateBot|linkdex.com|Curious\ George|Fetch-Guess|alexa\ site\ audit|AraBot|CliqzBot|findlinks) 1;
	~*(CCResearchBot|Semantifire|LinkAider|Zookabot|ScreenerBot\ Crawler|PaperLiBot|QuerySeekerSpider|Crowsnest|UnwindFetchor) 1;
	~*(MetaURI\ API|AcoonBot|Gigabot|firmilybot|Sosospider|OpenindexSpider|MetaHeadersBot|Strokebot|GeliyooBot|ownCloud\ Server\ Crawler) 1;
	~*(CirrusExplorer|ProCogSEOBot|Open\ Web\ Analytics\ Bot|RyzeCrawler|discoverybot|crawler\ for\ netopian|ADmantX\ Platform\ Semantic\ Analyzer) 1;
	~*(Linguee\ Bot|SolomonoBot|Grahambot|Automattic\ Analytics\ Crawler|PiplBot|FlightDeckReportsBot|fastbot\ crawler|4seohuntBot) 1;
	~*(Updownerbot|JikeSpider|NLNZ_IAHarvester|wsAnalyzer|YodaoBot|Esribot|Thumbshots.ru|BlogPulse|bot.wsowner.com|wscheck.com) 1;
	~*(Qseero|drupact|HuaweiSymantecSpider|PagePeeker|HomeTags|facebookplatform|Pixray-Seeker|MeMoNewsBot|ProCogBot|WillyBot) 1;
	~*(peerindex|MLBot|WebNL|Peepowbot|Semager|MIA\ Bot|heritrix|Eurobot|DripfeedBot|Whoismindbot|Bad-Neighborhood|Hailoobot) 1;
	~*(akula|MetamojiCrawler|Page2RSS|EasyBib\ AutoCite|NerdByNature.Bot|EventGuruBot|quickobot|gonzo|Influencebot|MSRBOT|Ronzoobot) 1;
	~*(ScoutJet|Twikle|SWEBot|RADaR-Bot|DCPbot|Castabot|imbot|EdisterBot|WASALive-Bot|Accelobot|PostPost|factbot|Setoozbot|biwec) 1;
	~*(Search17Bot|Lijit|JUST-CRAWLER|Apercite|pmoz.info\ ODP\ link\ checker|LemurWebCrawler|Covario-IDS|Holmes|RankurBot|AdsBot-Google) 1;
	~*(envolk|Ask\ Jeeves/Teoma|LexxeBot|StackRambler|Abrave\ Spider|EvriNid|arachnode.net|CamontSpider|wikiwix-bot|Nymesis) 1;
	~*(trendictionbot|SEODat|SygolBot|Snapbot|ZookaBot|CligooRobot|cityreview|nworm|SBIder|TwengaBot|Dot\ TK\ -\ spider|ParchBot) 1;
	~*(Peew|YRSpider|Urlfilebot\ (Urlbot)|Gaisbot|WatchMouse|Tagoobot|WebWatch/Robot_txtChecker|urlfan-bot|StatoolsBot|page_verifier) 1;
	~*(SSLBot|SAI\ Crawler|DomainDB|WMCAI_robot|voyager|copyright\ sheriff|Ocelli|Twiceler|amibot|abby|NetResearchServer|VideoSurf_bot) 1;
	~*(XML\ Sitemaps\ Generator|BlinkaCrawler|nodestackbot|Pompos|taptubot|BabalooSpider|Yaanb|Girafabot|livedoor\ ScreenShot|eCairn-Grabber) 1;
	~*(FauBot|Toread-Crawler|Setoozbot|MetaURI|L.webis|Web-sniffer|FairShare|Ruky-Roboter|ThumbShots-Bot|BotOnParade|Amagit.COM|HatenaScreenshot) 1;
	~*(HolmesBot|dotSemantic|Karneval-Bot|AportWorm|XmarksFetch|FeedFinder/bloggz.se|CorpusCrawler|Willow\ Internet\ Crawler|OrgbyBot|GingerCrawler) 1;
	~*(pingdom.com_bot|Nutch|baypup|Mp3Bot|192.comAgent|Surphace\ Scout|WikioFeedBot|Szukacz|DBLBot|Thumbnail.CZ\ robot|LinguaBot|GurujiBot) 1;
	~*(Charlotte|50.nu|SanszBot|moba-crawler|HeartRails_Capture|SurveyBot|MnoGoSearch|smart.apnoti.com\ Robot|Topicbot|JadynAveBot|OsObot) 1;
	~*(WebImages|WinWebBot|Scooter|Scarlett|GOFORITBOT|DKIMRepBot|Yanga|DNS-Digger-Explorer|Robozilla|YowedoBot|botmobi|Fooooo_Web_Video_Crawl) 1;
	~*(UptimeDog|Metaspinner/0.01|Touche|RSSMicro.com\ RSS/Atom\ Feed\ Robot|SniffRSS|Kalooga|FeedCatBot|WebRankSpider|Flatland\ Industries\ Web\ Spider) 1;
	~*(DealGates\ Bot|Link\ Valet\ Online|Shelob|Technoratibot|Flocke\ bot|FollowSite\ Bot|Visbot) 1;
}

if ($limit_bots = 1) {
	return 403;
}
```

#####- `/etc/nginx/php-fpm.conf`

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
        fastcgi_pass unix:/var/run/php5-fpm.sock;

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


####4. Create project site config `/etc/nginx/sites-available/project.tld`

```
##
# For product environment uncomment "include expires.conf" and comment "include deny-bots.conf;"
##

# Must be included in http directive 
include deny-bots.conf;

# frontend
server {
        server_name project.tld;
        listen 80;

        root          /web/www/project.tld/html/frontend/web;

        access_log    /web/www/project.tld/log/access.log combined buffer=50k;
        error_log     /web/www/project.tld/log/error.log notice;

        include php-fpm.conf;
        
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

        include php-fpm.conf;
        
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


####7. Restart nginx


    sudo service nginx restart


###Clone project

####1. Generate deployment ssh key



    sudo su web
    ssh-keygen -t rsa -C "deploy@project.tld"
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

