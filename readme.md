telegram-mailgate
=================

telegram-mailgate is very similar to [gpg-mailgate](https://github.com/uakfdotb/gpg-mailgate).
This script is a content filter for Postfix to send mails via telegram.
It is very minimalistic but it can be useful to receive say outputs from CRON directly to your mobile.

    usage: telegram-mailgate.py [-h] [--config CONFIG] [--from FROM]
                                [--queue-id QUEUE_ID] [--raw | --simple-header]
                                to [to ...]
    
    positional arguments:
      to                   the recipients
    
    optional arguments:
      -h, --help           show this help message and exit
      --config CONFIG      use a custom configuration file
      --from FROM          overwrite field "From" from the header. Ignored if
                           --raw is specified.
      --queue-id QUEUE_ID  specify the queue ID of the message for logging
      --raw                include all sections, header and attachments
      --simple-header      include a header in a separate message

Installation
------------

    sudo apt update && sudo apt install python3 python3-pip
    sudo adduser --system telegram-mailer  # create a specific user to run telegram-mailgate
    sudo -u telegram-mailer pip3 install --user python-telegram-bot
    sudo cp telegram-mailgate/telegram-mailgate.py /usr/local/bin/
    sudo chown telegram-mailer /usr/local/bin/telegram-mailgate.py
    sudo chmod 700 /usr/local/bin/telegram-mailgate.py
    sudo mkdir /etc/telegram-mailgate
    sudo cp telegram-mailgate/{main.cf,logging.cf,aliases} /etc/telegram-mailgate/    

Edit `/etc/telegram-mailgate/main.cf` to configure your telegram API key.
Edit `/etc/telegram-mailgate/aliases` to configure the mapping between the recipients' addresses and the corresponding chat ID. The format is:

    Adam 00000000
    Bob 00000001
    Charlie 00000002

The format does not support comments nor empty lines.

You can optionally edit `/etc/telegram-mailgate/logging.cf` to configure the logging. The default logs to syslog.

Postfix configuration
---------------------

In `/etc/postfix/master.cf`, add the following process:

    # =======================================================================
    # telegram-mailgate
    telegram-mailgate unix -     n       n       -        -      pipe
      flags= user=telegram-mailer argv=/usr/local/bin/telegram-mailgate.py --simple-header --queue-id $queue_id $recipient

Make sure that whatever user you specify above has permissions to execute telegram-mailgate and python dependencies installed. Of course you can set whatever arguments for the script.
    
In `/etc/postfix/main.cf` add telegram-mailgate as a content filter:

    content_filter = telegram-mailgate
    
Restart postfix

    # postfix reload

Testing
-------

    echo "This is the body of the email" | mail -s "This is the subject line" <my_recipent>

replace <my_recipient> by who ever is listed in the alias file. If local user, you may need to add @<hostname> at the end.
You can see logs with `sudo tail -f /var/log/mail.log`.
