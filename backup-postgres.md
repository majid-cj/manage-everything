# Backup Postgres script and set crontab to automate it

after installing your postgres, change the following to enable terminal authentication

    $ sudo nano /etc/postgresql/10/main/pg_hba.conf

    # scroll down the file until you find this commented line
    <# Database administrative login by Unix domain socket>
    and below it you will find something like this

    local   all             postgres                                peer

    comment this line of your working database is named postgres else
    let this line and write below it

    local   all             <your_db_name>                           md5


# make sure to align them

    # save and exit

when you done changing this file go to home directory do the following

    1- create a new file

        $ nano ~/.pgpass

    2- writer inside it

        localhost:5432:<your_db_name>:<db_user>:<db_password>

    3- save and exit

    4- change mode to

        $ chmod 0600 ~/.pgpass

backup script

1- create a .sh file name it anything you want it's better to create it at your user home dir

    eg:

    $ touch ~/<backup>.sh

2- create your back up directory (where you want to save your db backups)

    eg:

    $ mkdir ~/back_up_postgres


3- write the following scripts into your .sh file

    BACKUP_DIR=/home/<user_name>
    DAYS_TO_KEEP=<any_number_you_want>
    FILE_SUFFIX=_pg_backup.sql
    DATABASE=<your_db_name>
    USER=<db_user>

    FILE=`date +"%Y%m%d%H%M"`${FILE_SUFFIX}

    OUTPUT_FILE=${BACKUP_DIR}/${FILE}

    pg_dump -w -c -U ${USER} ${DATABASE} -F p -f ${OUTPUT_FILE}

    gzip $OUTPUT_FILE

    # prune old backups
    find $BACKUP_DIR -maxdepth 1 -mtime +$DAYS_TO_KEEP -name "*${FILE_SUFFIX}.gz" -exec rm -rf '{}' ';'

4- save and exit

5- change file mode to executable file

    $ chmod +x ~/<backup>.sh

6- create crontab to repeat this backup

    $ crontab -e (choose your preferred editor)

    * * 1,15 * * /path/to/<backup>.sh

    # this cron job it repeated every first and 15th day of every month

    you can run your own by changing it to fit your need

# and when your are here your done 😎