Block uploads directories
===


    location ~* /(?:uploads|files)/.*\.php$ {
      deny all;
    }
    # Block PHP files in content directory.
    location ~* /wp-content/.*\.php$ {
      deny all;
    }
    # Block PHP files in includes directory.
    location ~* /wp-includes/.*\.php$ {
      deny all;
    }
    # Block PHP files in uploads, content, and includes directory.
    location ~* /(?:uploads|files|wp-content|wp-includes)/.*\.php$ {
      deny all;
    }
