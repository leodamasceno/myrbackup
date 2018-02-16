
# myrbackup
Ruby application to create and store backups from different engines (MySQL, Oracle and PostgreSQL).

## Dependencies
To start, you need ruby 2.x installed. Then, check the dependencies below.
### Gems
 You will also need to install the following gems:
* aws-sdk-s3
* net-sftp

Type the following command to install it:

    gem install aws-sdk-s3 net-sftp

### Software
Install mysql CLI, specifically the *mysqldump* command, which will be used by the application to create the backup. 

### Environment variables
You may have issues running the application when the storage type SSH is specified in the configuration file if you don't have these variables set *LANG, LANGUAGE and LC_ALL*. You can set it in your session or in the bashrc file:

    export LANG=en_US.UTF-8
    export LANGUAGE=en_US.UTF-8
    export LC_ALL=en_US.UTF-8

### AWS configuration
Set the environment variables from AWS to use the S3 storage type. You can do that by either creating the file */var/root/.aws/credentials* manually or simply executing the aws CLI:

    aws configure

### Generate RSA keys
You will need RSA keys (private and public) to encrypt and decrypt passwords. All the passwords in the configuration file should be encrypted. Generate the private key using the following command:

    openssl genrsa -out private.pem 2048

Now, create the public key:

    openssl rsa -in private.pem -out public.pem -outform PEM -pubout
You should not type a password when requested.

## Configuration file
With all the dependencies installed, it's time to create the configuration file. The application will look for it */etc/myrbackup*, so the full path should be */etc/myrbackup/config.yaml*. Check the example below:

    config:
      private_key: /Users/leonardo.damasceno/Documents/Ruby/backup/lib/private.pem
      public_key: /Users/leonardo.damasceno/Documents/Ruby/backup/lib/public.pem
      
    database:
      dev-db01:
        engine: postgresql
	      storage:
          - type: s3
          bucket: my-backups-bucket
          path: backups
          region: eu-west-1
        host: dev-db01.company.com
        database: dev_db01
        user: backup
        password: fwPUwUGubT219lekFNYgPU4Sx5148udiaxIEXEwrpn6WzTtG+2dE3cLYsi2gm7HE1EIq5vxJ5bKuu77oGl6WVjSNgVew5CZ9BW2iR9YzIAcUvpB1P37CiBaizMtdQ4z5/rqNytybwf8ZhoOt2RGYznxKOPSR0ul1hl782JOwPzuLn+H+n2EO44//xq13fc1veS/1DhU+uQjZkjBre2Vq3a57roS24JAaJKywSGZ9T9GMUpQ2EjCuJ0YNi2euevHiFzltxRNI2RZQ/7F9pnHSoTakwgz5mIfN1kIsDmsu34HvOe18vCT8vswGSQ4xx7g6G3vza1mxG/Ctnj+j0KBvDg==
          
      dev-db02:
        engine: mysql
        storage:
          - type: ssh
          server: 172.20.209.168
          user: deploy
          key_file: /home/deploy/.ssh/id_rsa
          path: /mnt/backups
        host: dev-db02.company.com
        database: dev_db02
        user: backup
        password: My9HRnKXByWroR4EBwgoHCMK6r5H0jaC2J31F7jrCS9N1nmqH286JjeaDkCtoCWUJ33qfDtu5BOGEj9KqWV9kJDHNzCgtAqjCORe1x7TWec7jAmuUheyogwgMNpvCQXE2K7C6mXpjjVo13RYhg2XOz4Hcpt3OIk6flHFTqi32wjrDaLQCbBIcFhCVVNUfhjqeWXYmV3+wES8FZGfkqTzythaIpoOBODZc7q7dRucyu6Ur04ToVZ4fwNjkyov/lD3kwFqzQ8qb4xEJYujT4G3gGrX31yczRhpyIj889NcB57IQtZxi283UbMKnDsT6uK43guSMP1xhX+YtVnUpPDqbA==
      dev-db03:
        engine: postgresql
        storage:
          - type: local
          path: /mnt/backups
        host: dev-db03.company.com
        database: dev_db03
        user: admin
        password: YijkO4uOh+hdrB0KF7PiXIY/YvXK+fRj0jdYacupUp2cWXN45MXQy69EQ3a/oqkh38exT2akb3tba7eJ9ZHDaYdl6bsXocL/8TwkcRV2ylDluFs67+CkTiQ79TnksA1agdDlqAhIOJpnjdnh4HNT9Ww1e20P0WSunOggUHs92+P5bm7mEz5RARktRy79wtvVQJvXgwb25EtSZopvKd5Iub4FJLfpfBkzwidK9ViDlAoQxctlNW+JI1TGcp2N6tqSUqfpelSG9+nDUtPdwJ566ZvuVe6M2Z0xqWXNQeKvemKSzIXHzreT/j5li7U6itZVpX5PlWfNk9TYUqD8f0x5gA==

In the configuration file above, we have three different servers with different storage types:
* **dev-db01**: It will create the backup and upload it to AWS S3.
* **dev-db02**: This one will create the backup and copy it to a server via SSH used to store all the backups.
* **dev-db03**: The last one will simply create the backup and keep it locally in a path chosen by you.

## Executing the application
The application can be used to create and store backups but you can also encrypt and decrypt strings with the RSA keys. So, use the application to encrypt your password before putting it into the configuration file:

    ./myrbackup -e "hello"
    fwPUwUGubT219lekFNYgPU4Sx5148udiaxIEXEwrpn6WzTtG+2dE3cLYsi2gm7HE1EIq5vxJ5bKuu77oGl6WVjSNgVew5CZ9BW2iR9YzIAcUvpB1P37CiBaizMtdQ4z5/rqNytybwf8ZhoOt2RGYznxKOPSR0ul1hl782JOwPzuLn+H+n2EO44//xq13fc1veS/1DhU+uQjZkjBre2Vq3a57roS24JAaJKywSGZ9T9GMUpQ2EjCuJ0YNi2euevHiFzltxRNI2RZQ/7F9pnHSoTakwgz5mIfN1kIsDmsu34HvOe18vCT8vswGSQ4xx7g6G3vza1mxG/Ctnj+j0KBvDg==

As you can see in the example above, the application created the encrypted version of the text password "hello". You should now copy it and add to your configuration. We do not intend to allow clear text passwords in the configuration file, because it's a dangerous world out there.

You can also decrypt these encrypted passwords if the same public and private keys were used to create it previously:

    ./myrbackup -d "fwPUwUGubT219lekFNYgPU4Sx5148udiaxIEXEwrpn6WzTtG+2dE3cLYsi2gm7HE1EIq5vxJ5bKuu77oGl6WVjSNgVew5CZ9BW2iR9YzIAcUvpB1P37CiBaizMtdQ4z5/rqNytybwf8ZhoOt2RGYznxKOPSR0ul1hl782JOwPzuLn+H+n2EO44//xq13fc1veS/1DhU+uQjZkjBre2Vq3a57roS24JAaJKywSGZ9T9GMUpQ2EjCuJ0YNi2euevHiFzltxRNI2RZQ/7F9pnHSoTakwgz5mIfN1kIsDmsu34HvOe18vCT8vswGSQ4xx7g6G3vza1mxG/Ctnj+j0KBvDg=="
    hello

The string "hello" was returned by the application.

Finally, you can run the application to create backups, you won't need to specify options because that's the default option so simply run the script:

    ./myrbackup
        

## Restore
The aplication creates a bz2 archive, you will need to uncompress it and then restore it. You can simply use the mysql command to do that.
