#Update MySQL config

## Create `/etc/mysql/conf.d/override.cnf` config file (1GB RAM example)



    [mysqld]

    key_buffer_size                 = 32M
    thread_cache_size               = 50
    max_connections                 = 500
    innodb_buffer_pool_size         = 384M
    
    
    default_time_zone               = '+00:00'
    skip-name-resolve
    myisam-recover-options          = BACKUP

## For external conection add



    bind-address                    = 0.0.0.0
