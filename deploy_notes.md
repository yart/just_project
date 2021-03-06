Примечания к [заметкам][1] [Максима Филатова][2].

Раздел Primary Settings
=======================

Change root password
--------------------
Не выполнял. Смысл раздела: сменить дефолтный, выдаваемый сисадмином
провайдера хостинга пароль root'а на свой, желательно более сложный, в целях
безопасности. Для этого Макс предлагает воспользоваться программой apg. Я,
для тех же целей, пользуюсь программой KeePassX.

Add admin user
--------------
Добавляем нового пользователя, отличного от пользователя root. Всё в тех же
целях безопасности.

Мне не совсем понятно, зачем Максим создаёт пользователя в группе admin. Этой
группы, по-дефолту, в убунте нет. И команда useradd ругается на несуществующую
группу. Пришлось добавлять вручную:

     root@remote# groupadd admin 

А потом уже добавлять пользователя.

Далее, устанавливаем пароль для созданного пользователя. Тут вроде бы всё
ясно. Следующая команда, visudo, открывает файл конфигурации команды sudo.
Макс предложил записать туда вот такую строку:

    %admin ALL NOPASSWD:ALL

Судя по содержанию, предполагается, что пользователи состоящие в группе admin,
будут выполнять команды требующие прав root'а, без ввода пароля. Смутило меня
вот что: в файле /etc/sudoers, который мы и редактируем, с помощью команды
visudo, есть такая строчка:
    
    # %sudo ALL=NOPASSWD: ALL

В комментарии, расположенном строкой выше, предлагается её раскомментировать,
если есть желание добиться тех же целей, но для всех. Как видно, синтаксис
несколько отличается, от предлагаемого Максом.

Чтобы хоть как-то разобраться, нет ли тут какой ошибки, я воспользовался
гуглом. Запрос по ключевым словам "ALL NOPASSWD:ALL", привёл меня вот на эту
страницу: <http://www.net4me.net/info/3/net102.html>

Н-да. Прочтение статьи ясности не прибавило. Ладно. Положимся на Макса, и
пока оставим всё, как есть.

Гм... Всё-таки хорошая это штука — защита от дурака! visudo увидел ошибку во
вновь введённой строке, и предложил на выбор исправить, убить или сохранить
файл, как есть. Я выбрал исправить:
    %admin ALL=NOPASSWD:ALL
После чего сохранился без каких-либо эксцессов.

Однако!

Public key auth
---------------
Судя по всему, в этом подразделе, речь идёт об авторизации по публичному
ключу. В гуглогруппе по RoR, мне было предложено несколько вариантов этого
дела.

Пожалуй, начну с начала. Первым делом, надо перебраться с пользователя root,
на ранее созданного пользователя. У Макса это spbruby, у меня другой.
Делается это командой "su юзер", где "юзер" — имя ранее созданного
пользователя. Чтобы не париться с именованием, предположим, я назвал этого
самого юзера — new-user.

Заодно, я решил проверить, корректно ли выполнился предыдущий подраздел:

     root@remote# su new-user             
     new-user@remote$ sudo apt-get update 
    
Ух ты! Работает, шайтан-машина!

Поехали дальше. Домашний каталог у нового пользователя девственно чист, за
исключением трёх скрытых файлов:

     new-user@remote$ pwd                  
    /home/new-user                         
     new-user@remote$ ls -a                
    .  ..  .bash_logout  .bashrc  .profile 

Максим предлагает следующее: посмотреть локальный файл с публичным ключом
командой cat, создать каталог .ssh в домашней папке пользователя new-user и,
скопировав содержимое, через буфер обмена ввести его, командой "cat >", в
файл .ssh/authorized\_keys, на VDS/VPS попутно создавая этот самый файл.

Также, в гуглогруппе, мне предложили воспользоваться и такими командами:

1.  предложение от [sysadm][3]'а

        local$ ssh-copy-id -i ~/.ssh/id_rsa.pub логин@имя_хоста_или_IP_удалённой_машины

    Здесь предлагается использовать скрипт ssh-copy-id, который может не
    идти в комплекте с ОС. В Убунту он есть. Я проверил. Кстати, sysadm
    утверждает, что этот скрипт идёт в комплекте с ssh для Убунту. Как я понял,
    логиниться предполагается от имени new-user, чтобы публичный ключ попал
    по адресу. И, судя по всему, нужный каталог и файл будут созданы
    автоматически.

2.  предложение от [Димы][4]
    
        scp ~/.ssh/id_rsa.pub user@remotebox:.ssh/authorized_keys

    Здесь мы используем команду scp. Как я понял из названия и man'а,
    копирует необходимые файлы по протоколу ssh. В Убунту она также имеется.
    И вроде бы никто не упомянул, что в других ОС она может отсутствовать.

3.  предложение от [Thymothy N. Tsvetkov][5]
    Воспользоваться утилитой из мира Ruby: ssh-forever. Вот её страничка:
    https://rubygems.org/gems/ssh-forever. И описание:
    *Provides a replacement for the SSH command which automatically copies
    your public key while logging in*
        gem install ssh-forever
    
    Если я всё правильно понял и перевёл (да-да, плюс ко всему, я ещё
    пытаюсь выучить английский), имеется в виду вот что: ssh-forever
    обеспечивает замену для стандартных команд протокола ssh, которые
    автоматически копируют ваш публичный ключ в процессе вашего логина.

    В общем, как-то так. Это предложение я пока не рассматривал. Во-первых,
    не совсем понял, куда надо устанавливать утилиту: на локальную машину,
    или на VDS/VPS. Во-вторых, если всё-таки на удалённую машину, то Ruby там
    ещё нет. Он там будет позже.

В общем, в конечном итоге, решил воспользоваться первым предложением. Правда
есть тут одна сложность: публичный ключ, который у меня есть, создавался для
github. И по рекомендации с help.github.com, я повесил на него пароль.
Пароль, ясное дело, отличается от того, который есть у пользователя new-user,
на удалённой машине.

Чтобы избежать действий вслепую, нашёл документацию на opennet.ru:
<http://www.opennet.ru/base/sec/ssh_pubkey_auth.txt.html>
Правда, как оно обычно бывает на работе, когда занимаешься чем-то левым,
непосредственно с работой не связанным, тебя начинают атаковать начальство,
клиенты и прочие непонятные люди, которым ты срочно нужен. После нескольких
последовательных атак со стороны вышеперечисленных, мозг наотрез отказался
воспринимать новые знания. Статья показалась ужасно запутанной.

В общем, вот что я из неё понял: если при создании ключа, указывается пароль,
то будьте уверенны, что ssh у вас его попросит.

Так это, или не так, сейчас проверим.

     user@local$ ssh-copy-id -i ~/.ssh/id_rsa.pub new-user@000.111.222.333  
    new-user@000.111.222.333's password: <enter password>                   
    Now try logging into the machine, with "ssh 'new-user@000.111.222.333'",
    and check in:                                                           
                                                                            
    .ssh/authorized_keys                                                    
                                                                            
    to make sure we haven't added extra keys that you weren't expecting.    

Всё прошло успешно, и ssh-copy-id предлагает залогиниться через ssh, как
new-user@000.111.222.333, и проверить файл .ssh/authorized\_keys, "чтобы
убедиться, что мы не добавили левые ключи, которых вы не ожидали". Заботливый
скрипт, однако!

     user@local$ ssh new-user@000.111.222.333                               
    Linux remote 2.6.bla-bla-bla #1 SMP Fri Oct 1 14:17:14 MSD 2010 i686    
                                                                            
    The programs included with the Ubuntu system are free software;         
    the exact distribution terms for each program are described in the      
    individual files in /usr/share/doc/*/copyright.                         
                                                                            
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by    
    applicable law.                                                         
                                                                            
    To access official Ubuntu documentation, please visit:                  
    http://help.ubuntu.com/                                                 
    To run a command as administrator (user "root"), use "sudo <command>".  
    See "man sudo_root" for details.                                        
                                                                            
     new-user@remote$                                                       

Всё прошло удачно. Как я и предполагал, ssh попросил именно тот пароль,
который я указывал при создании публичного ключа. Из чего следует два вывода:

1.  Надо надёжно забэкапить локальный файл ~/.ssh/id\_rsa
2.  Никого и не под каким видом не подпускать к локальной машине. Во избежание.

Следующий, немаловажный, как я понял момент: выставляем права на каталог
~/.ssh:
    
     new-user@remote$ chmod 600 .ssh/authorized_keys // только пользователь может читать
                                                     // и изменять файл
     new-user@remote$ chmod 700 .ssh                 // только пользователь может видеть и
                                                     // изменять каталог

Уф! Вроде бы разобрался.

Configure OpenSSH
-----------------
Здесь меняем стандартный порт, что, кстати, вызвало небольшую дискуссию
в гуглогруппе. Основные аргументы против сводились к некоторому неудобству в
дальнейшем использовании. Но Максим Филатов предложил внести следующие
изменения конфигурационный файл для ssh (~/.ssh/config) на локальной машине:
    Host <any-name>
    Hostname <ip-or-fqdn>
    User some_user
    Port 22222

В общем, на данном этапе решил порт не менять. Но взял себе на заметку найти
и почитать что-нибудь по теме.

Помимо смены порта, запрещаем доступ по ssh через root'а. Это, как я понял,
первое имя пользователя, которое проверяют взломщики. Не будем рисковать.

Следом добавляем своего пользователя в список разрешённых к подключению по ssh.
Иначе мы к себе не попадём.

И самое главное: всё здесь надо сделать предельно внимательно. Иначе придётся
подламывать собственный сервер. Или переустанавливать. Хрен редьки не слаще.

Вроде бы всё.

Firewall
--------
Этот пункт опять вогнал меня в ступор. И вовсе не потому, что я знаю, как
сделать лучше. Совсем наоборот - я вообще поначалу не понял, о чём тут речь.
Спасибо Максиму. Он любезно разъяснил, что файл с настройками для файрвола,
находится в гит-репозитарии, которым я и пользуюсь сейчас.

В общем, вот что пришлось делать:
     user@local$ sudo aptitude install git-core iptables
    //Этого добра на моём VDS не водилось изначально.
     user@local$ git clone git://github.com/Bregor/spbruby-deploy-notes.git
     user@local$ cd spbruby-deploy-notes
Далее — по инструкции Макса.

В файле /var/lib/iptables/rules\_save, в строчке:

    -A INPUT -s x.x.x.x/32 -m comment --comment "Admin home" -j ACCEPT

надо **x.x.x.x** поменять на свой статический ip-адрес. Если он есть. Желательно,
чтобы он был домашним, то есть личным, ip-адресом. С этого адреса будет
позволен полный доступ к серверу.

И надо иметь в виду, **если** в предыдущем разделе **не изменялся** стандартный порт
для ssh, то в следующей строчке **надо исправить "22222"** на **"22"**.

Всё. С предварительной настройкой покончено. Для проверки разорвал
ssh-соединение и залогинился снова. Получилось. Завтра попробую проделать это
с динамического ip-адреса.

Раздел User shell settings
==========================
Colors, aliases, git-branch, etc
--------------------------------
Тут вроде как всё понятно.

Locale settings
---------------
Тут тоже.

Раздел Primary Software Install
===============================
Prerequisites
-------------
Думается мне, что этот раздел, как и предыдущий, стоило бы перенести в начало
инструкции. Ну, да ладно.

RubyEE
------
До этого как-то не доверял этому способу установки руби. Но если профессионалы
советуют, то деваться некуда. Будем испытать.

Гм... Надо бы и на локальной машине это дело установить.

Особенность, ранее мне незнакомая: Руби будет собираться из git-репозитария.

RubyGems
--------
Всё штатно.

Rails
-----
Аналогично.

NGINX and Phusion Passenger
---------------------------
А тут начались сюрпризы.
apt-get категорически отказался устанавливать пакеты libpcre3, libxml2,
libxslt-dev и curl-ssl:

     new-user@remote$ sudo apt-get install -y libpcre3 libpcre3-dev libperl-dev libxml2-dev libxml2 libxslt-dev curl-ssl libcurl4-openssl-dev       
    Reading package lists... Done                                           
    Building dependency tree                                                
    Reading state information... Done                                       
    libpcre3 is already the newest version.                                 
    libxml2 is already the newest version.                                  
    libxml2 set to manually installed.                                      
    Note, selecting libxslt1-dev instead of libxslt-dev                     
    Package curl-ssl is a virtual package provided by:                      
      curl 7.18.2-8ubuntu4.1                                                
    You should explicitly select one to install.                            
    E: Package curl-ssl has no installation candidate                       

aptitude тоже ругнулся, но установку не прервал:

     new-user@remote$ sudo aptitude install -y libpcre3 libpcre3-dev libperl-dev libxml2-dev libxml2 libxslt-dev curl-ssl libcurl4-openssl-dev      
    Reading package lists... Done                                           
    Building dependency tree                                                
    Reading state information... Done                                       
    Reading extended state information                                      
    Initializing package states... Done                                     
    Writing extended state information... Done                              
    Note: selecting "libxslt1-dev" instead of the                           
          virtual package "libxslt-dev"                                     
    Note: selecting "curl" instead of the                                   
          virtual package "curl-ssl"                                        
    The following NEW packages will be installed:                           
      curl libcurl3{a} libcurl4-openssl-dev libglib2.0-0{a} libglib2.0-data{a} libidn11-dev{a} libldap2-dev{a} libpcre3-dev
      libpcrecpp0{a} libperl-dev libperl5.10{a} libxml2-dev libxslt1-dev libxslt1.1{a} pkg-config{a} shared-mime-info{a} 
    0 packages upgraded, 16 newly installed, 0 to remove and 0 not upgraded.
    Need to get 8887kB of archives. After unpacking 28,7MB will be used.    
    Writing extended state information... Done                              
    Get:1 http://fr.archive.ubuntu.com jaunty-updates/main libcurl3 7.18.2-8
    ubuntu4.1 [220kB]            

                                   ~ ~ ~                                    

    Initializing package states... Done                                     
    Writing extended state information... Done                              

Как видно, aptitude заменил libxslt-dev на libxslt1-dev, и curl-ssl на curl.
Также, они (apt-get и aptitude) любезно сообщили, что вышеупомянутые пакеты
являются виртуальными. Я озадачился. Нашёл вот эту статью:
<http://www.debian.org/doc/FAQ/ch-pkg_basics.ru.html#s-virtual>, 
почитал. Понял только одно: всё нормально. Но вот нахрена нужны эти
виртуальные пакеты, я так и не понял.

Остальная часть прошла, как по маслу.

Configure RDBMS
===============

PostgreSQL related stuff
------------------------
Первые два пункта прошли успешно. Затык получился на третьем:

*   Adding user and database

psql ни в какую не хотел признавать существование юзера postgres:

     new-user@remote$ psql -Upostgres                                         
    psql: FATAL:  Ident authentication failed for user "postgres"             

Но я нашёл, как поправить это гадство. На форуме сайта sql.ru, в ветке 
<http://www.sql.ru/Forum/actualthread.aspx?bid=7&tid=451845&hl=>
пользователь [Shweik][6] сказал,
что надо бы поменять юзеру метод идентификации.

Я подумал, и решил, что это резонно. Снова открыл файл /etc/postgresql/8.4/main/pg\_hba.conf
и изменил строку
    # Database administrative login by UNIX sockets
    local   all         postgres                          ident
на
    # Database administrative login by UNIX sockets
    local   all         postgres                          trust

После чего перезапустил PostgeSQL:

    new-user@remote$ sudo /etc/init.d/postgresql-8.4 restart                 
    * Restarting PostgreSQL 8.4 database serv                          [ OK ]

Остальные действия прошли без эксцессов.

Configure SSL
=============
Надо отметить один момент: команды отмеченные "#" надо выполнять именно под
рутом: sudo -s. Иначе ничего не выйдет.

Чуть не забыл! Камрад [Alex L. Demidov][8], из гуглогрупп, любезно разъяснил, что на
самом деле, sudo -s вовсе не является обязательным условием. Достаточно добавить
ключ -E к команде, чтобы добиться всё прошло штатно. 
  
Фишка в том, что по дефолту, sudo не сохраняет переменные пользователя, 
определявшиеся в предыдущей сессии. А по условиям, мы их как раз меняем. Если 
выполнять команды просто через sudo, то ничего не выйдет. bash будет ругаться, и 
требовать их (переменные), назначить. sudo -s приводит нас в режим бога, что не 
всегда нужно. А sudo -E, запрещает sudo забывать переменные пользователя, 
определённые в предыдущей сессии.

Configure nginx
===============
Тут практически всё ровно, особенно если не забывать о том, что везде вместо
spbruby.org, надо втыкать название своего сайта. И стоит обратить пристальное
внимание на файл /opt/nginx/conf/sites-available/spbruby.org в клоне проекта.
Начать стоит с того, что этот файл должен называться несколько иначе.

Backups
=======
Здесь аналогично предыдущему разделу. Только пристальное внимание надо обратить
на совсем другой файл — /etc/safe.rb

Logs Rotation
=============
Тут тоже надо внести необходимые изменения. Например, поправить предлагаемый
Максом файл etc/logrotate.d/spbruby.org, и переименовав его как надо. Сам файл,
надо избавить от тех записей, где присутствуют несуществующие в приложении
папки. Это папки current и rails. Макс обмолвился, что current требуется для
Capistrano. А для чего была нужна rails, он уже и сам не помнит.

Mail
====
Этот раздел я пока пропустил. На данном этапе он мне не нужен, так как я уже
привесил к своему сайту почтовую приблуду от Яндекса. Пока я ещё точно не 
уверен, будет ли размещаться мой сайт у текущего провайдера. Плюс к тому, 
считаю, что на данном этапе разбираться ещё и с почтовыми заморочками — это 
выше крыши.

Чуть позже, когда освоюсь в своём, пока ещё толком не созданном приложении, то
скорее всего, прикручу к нему самописную почту. Вроде как и в рельсах были 
какие-то классы на эту тему...

Autostart services
==================
Здесь особых проблем не возникло. Как всегда, надо внимательно изучить выкла-
дываемые Максом конфиги, на тему изменения названий папок. Но, насколько я 
помню, в конфигах для этой части, все пути стандартные.

Search
======
Эту часть не устанвливал. Пока не вижу необходимости. Возможно, позже вернусь
к этому разделу.

POST SCRIPTUM
=============
Возникли некоторые грабли при попытке запустить стандартное приложение.

1.  postgresql наотрез отказался работать с пользователем, только с рутом.
2.  Почему-то, после команды rake db:create, не создалась база данных для 
    версии production. Пришлось добавлять руками, через шелл postgreSQL.
3.  postgreSQL ненавидит точку в названии базы данных. Поэтому имена вида
    example.com_development не принимаются.

Все вышеперечисленные нестыковки, вызывали ошибку: *"We're sorry, but something
went wrong. 
We've been notified about this issue and we'll take a look at it 
shortly."*

В логах сообщалось, что ошибка возникает при обращении к несуществующей базе
данных. Так как я сразу запускал стандартное приложение в окружении production,
то, в-принципе, понятно откуда ошибка: эта база тупо не была создана (см. п.2)
  
Теперь имею на руках ошибку: *"The page you were looking for doesn't exist.
You may have mistyped the address or the page may have moved."*

Проверил лог production, пишет что ошибка маршрутизатора.

Сильно изнасиловал гугл, и выяснил, что в режиме production, /rails/info/properties
недоступна. Посмотреть окружение можно из режима development.

Кстати, список существующих баз данных, для postgreSQL, можно получить командой
    
    psql -l

--------------------------------------------------------------------------------
[1]: https://github.com/Bregor/spbruby-deploy-notes
[2]: https://groups.google.com/groups/profile?hl=ru&enc_user=R-LXqBMAAAAEutvFgtXHbFbSFkm_dcwBWMj6vob75xS36mXc24h6ww
[3]: https://groups.google.com/groups/profile?hl=ru&enc_user=Ya1LkxoAAABJYKQQhUJ5Er_EYZsKOnUpfVkDoaoMBC1ZX5YCLbSZfw
[4]: https://groups.google.com/groups/profile?hl=ru&enc_user=dCjNoxwAAAAuAF-vzCYXVD56xWznEgf-rzmJETj_idM-nzbzlXS03g
[5]: https://groups.google.com/groups/profile?hl=ru&enc_user=JD4F_hoAAAA2rc_oEQk6P3IPyNs2Pkg5fVkDoaoMBC1ZX5YCLbSZfw
[6]: http://www.sql.ru/Forum/memberinfo.aspx?mid=8596
[7]: https://groups.google.com/groups/profile?hl=ru&enc_user=z9OqOhcAAADiNZxeYw1MzhzhHqQzo9dNHqZiDvCVswhrZ6TQxKj0ww
