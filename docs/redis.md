#Update Redis config

## Add as the last line of `/etc/redis/redis.conf`
    
    
    
    include /etc/redis/conf.d/override.conf

## Create `/etc/redis/conf.d/override.conf`


    requirepass "soMe:%wery1#_str0ng';pasSword"
    rename-command CONFIG ""
    rename-command SHUTDOWN ""
    rename-command FLUSHALL ""
    rename-command FLUSHDB ""
    
    
For external conection add



    bind 0.0.0.0

