## Задание
> установить MongoDB одним из способов: ВМ, докер;
> заполнить данными;
> написать несколько запросов на выборку и обновление данных

## Задание повышенной сложности
> создать индексы и сравнить производительность.

## Решение

### Установка и настройка mongodb
Создал виртуальную машину в яндекс mdb1, пользователь vassaev
Установил mongodb

**wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -**
```log
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
```

**sudo su**
**echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list**
```log
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse
```

**apt updated**
```log
W: https://repo.mongodb.org/apt/ubuntu/dists/focal/mongodb-org/6.0/Release.gpg: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

**apt-get install -y mongodb-org**
```log
The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but it is not installable
E: Unable to correct problems, you have held broken packages.
```
Возникли ошибки. Решено по https://askubuntu.com/questions/1403619/mongodb-install-fails-on-ubuntu-22-04-depends-on-libssl1-1-but-it-is-not-insta

**echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt-get update
sudo apt-get install libssl1.1
sudo rm /etc/apt/sources.list.d/focal-security.list**

Далее - по инструкции

Создадим каталог для данных

**root@mdb1:/home/vassaev# sudo mkdir /home/mongo && sudo mkdir /home/mongo/db1 && sudo chmod 777 /home/mongo/db1**

Запустим монго с параметрами для этого каталога

**root@mdb1:/home/vassaev# mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid**
```log
about to fork child process, waiting until server is ready for connections.
forked process: 2395
child process started successfully, parent exiting
```

**ps aux | grep mongo| grep -Ev "grep"**
```log
root        2395  3.2  9.7 2599784 96980 ?       Sl   15:52   0:00 mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid
```

Добавляем пользователя

**mongosh --port 27001**

**```log
use admin
db.createUser( { user: "root", pwd: "otus", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
exit```**

добавляем security:authorization: enabled и bindIpAll: true

**nano /etc/mongod.conf**

перестартуем

**systemctl restart mongod**

### Загрузка данных 

Данные по Fakenews с https://www.kaggle.com/competitions/fake-news/data?select=train.csv с помощью sftp загружаем данные в /var/www-data

**mongoimport --type csv -d otus -c fakenews --headerline /var/www-data/train.csv -u root --authenticationDatabase admin**
```log
Enter password for mongo user:

2023-04-23T16:45:41.453+0000    connected to: mongodb://localhost/
2023-04-23T16:45:44.454+0000    [######..................] otus.fakenews        24.0MB/94.1MB (25.5%)
2023-04-23T16:45:47.454+0000    [#############...........] otus.fakenews        53.2MB/94.1MB (56.6%)
2023-04-23T16:45:50.453+0000    [#####################...] otus.fakenews        84.2MB/94.1MB (89.5%)
2023-04-23T16:45:51.741+0000    [########################] otus.fakenews        94.1MB/94.1MB (100.0%)
2023-04-23T16:45:51.741+0000    20800 document(s) imported successfully. 0 document(s) failed to import.
'''
```

### Запросы

Всего документов

**otus> db.fakenews.countDocuments()**
```log
20800
```

Вывести 2 документа

**otus> db.fakenews.find().limit(2)**
```js
[
  {
    _id: ObjectId("644560b571d671bf2b58cdd6"),
    id: 0,
    title: 'House Dem Aide: We Didn’t Even See Comey’s Letter Until Jason Chaffetz Tweeted It',
    author: 'Darrell Lucus',
    text: 'House Dem Aide: We Didn’t Even See Comey’s Letter Until Jason Chaffetz Tweeted It By Darrell Lucus on October 30, 2016 Subscribe Jason Chaffetz on the stump in American Fork, Utah ( image courtesy Michael Jolley, available under a Creative Commons-BY license) \n' +
      'With apologies to Keith Olbermann, there is no doubt who the Worst Person in The World is this week–FBI Director James Comey. But according to a House Democratic aide, it looks like we also know who the second-worst person is as well. It turns out that when Comey sent his now-infamous letter announcing that the FBI was looking into emails that may be related to Hillary Clinton’s email server, the ranking Democrats on the relevant committees didn’t hear about it from Comey. They found out via a tweet from one of the Republican committee chairmen. \n' +
      'As we now know, Comey notified the Republican chairmen and Democratic ranking members of the House Intelligence, Judiciary, and Oversight committees that his agency was reviewing emails it had recently discovered in order to see if they contained classified information. Not long after this letter went out, Oversight Committee Chairman Jason Chaffetz set the political world ablaze with this tweet. FBI Dir just informed me, "The FBI has learned of the existence of emails that appear to be pertinent to the investigation." Case reopened \n' +
      '— Jason Chaffetz (@jasoninthehouse) October 28, 2016 \n' +
      'Of course, we now know that this was not the case . Comey was actually saying that it was reviewing the emails in light of “an unrelated case”–which we now know to be Anthony Weiner’s sexting with a teenager. But apparently such little things as facts didn’t matter to Chaffetz. The Utah Republican had already vowed to initiate a raft of investigations if Hillary wins–at least two years’ worth, and possibly an entire term’s worth of them. Apparently Chaffetz thought the FBI was already doing his work for him–resulting in a tweet that briefly roiled the nation before cooler heads realized it was a dud. \n' +
      'But according to a senior House Democratic aide, misreading that letter may have been the least of Chaffetz’ sins. That aide told Shareblue that his boss and other Democrats didn’t even know about Comey’s letter at the time–and only found out when they checked Twitter. “Democratic Ranking Members on the relevant committees didn’t receive Comey’s letter until after the Republican Chairmen. In fact, the Democratic Ranking Members didn’ receive it until after the Chairman of the Oversight and Government Reform Committee, Jason Chaffetz, tweeted it out and made it public.” \n' +
      'So let’s see if we’ve got this right. The FBI director tells Chaffetz and other GOP committee chairmen about a major development in a potentially politically explosive investigation, and neither Chaffetz nor his other colleagues had the courtesy to let their Democratic counterparts know about it. Instead, according to this aide, he made them find out about it on Twitter. \n' +
      'There has already been talk on Daily Kos that Comey himself provided advance notice of this letter to Chaffetz and other Republicans, giving them time to turn on the spin machine. That may make for good theater, but there is nothing so far that even suggests this is the case. After all, there is nothing so far that suggests that Comey was anything other than grossly incompetent and tone-deaf. \n' +
      'What it does suggest, however, is that Chaffetz is acting in a way that makes Dan Burton and Darrell Issa look like models of responsibility and bipartisanship. He didn’t even have the decency to notify ranking member Elijah Cummings about something this explosive. If that doesn’t trample on basic standards of fairness, I don’t know what does. \n' +
      'Granted, it’s not likely that Chaffetz will have to answer for this. He sits in a ridiculously Republican district anchored in Provo and Orem; it has a Cook Partisan Voting Index of R+25, and gave Mitt Romney a punishing 78 percent of the vote in 2012. Moreover, the Republican House leadership has given its full support to Chaffetz’ planned fishing expedition. But that doesn’t mean we can’t turn the hot lights on him. After all, he is a textbook example of what the House has become under Republican control. And he is also the Second Worst Person in the World. About Darrell Lucus \n' +
      "Darrell is a 30-something graduate of the University of North Carolina who considers himself a journalist of the old school. An attempt to turn him into a member of the religious right in college only succeeded in turning him into the religious right's worst nightmare--a charismatic Christian who is an unapologetic liberal. His desire to stand up for those who have been scared into silence only increased when he survived an abusive three-year marriage. You may know him on Daily Kos as Christian Dem in NC . Follow him on Twitter @DarrellLucus or connect with him on Facebook . Click here to buy Darrell a Mello Yello. Connect",
    label: 1
  },
  {
    _id: ObjectId("644560b571d671bf2b58cdd7"),
    id: 1,
    title: 'FLYNN: Hillary Clinton, Big Woman on Campus - Breitbart',
    author: 'Daniel J. Flynn',
    text: 'Ever get the feeling your life circles the roundabout rather than heads in a straight line toward the intended destination? [Hillary Clinton remains the big woman on campus in leafy, liberal Wellesley, Massachusetts. Everywhere else votes her most likely to don her inauguration dress for the remainder of her days the way Miss Havisham forever wore that wedding dress.  Speaking of Great Expectations, Hillary Rodham overflowed with them 48 years ago when she first addressed a Wellesley graduating class. The president of the college informed those gathered in 1969 that the students needed “no debate so far as I could ascertain as to who their spokesman was to be” (kind of the like the Democratic primaries in 2016 minus the   terms unknown then even at a Seven Sisters school). “I am very glad that Miss Adams made it clear that what I am speaking for today is all of us —  the 400 of us,” Miss Rodham told her classmates. After appointing herself Edger Bergen to the Charlie McCarthys and Mortimer Snerds in attendance, the    bespectacled in granny glasses (awarding her matronly wisdom —  or at least John Lennon wisdom) took issue with the previous speaker. Despite becoming the first   to win election to a seat in the U. S. Senate since Reconstruction, Edward Brooke came in for criticism for calling for “empathy” for the goals of protestors as he criticized tactics. Though Clinton in her senior thesis on Saul Alinsky lamented “Black Power demagogues” and “elitist arrogance and repressive intolerance” within the New Left, similar words coming out of a Republican necessitated a brief rebuttal. “Trust,” Rodham ironically observed in 1969, “this is one word that when I asked the class at our rehearsal what it was they wanted me to say for them, everyone came up to me and said ‘Talk about trust, talk about the lack of trust both for us and the way we feel about others. Talk about the trust bust.’ What can you say about it? What can you say about a feeling that permeates a generation and that perhaps is not even understood by those who are distrusted?” The “trust bust” certainly busted Clinton’s 2016 plans. She certainly did not even understand that people distrusted her. After Whitewater, Travelgate, the vast   conspiracy, Benghazi, and the missing emails, Clinton found herself the distrusted voice on Friday. There was a load of compromising on the road to the broadening of her political horizons. And distrust from the American people —  Trump edged her 48 percent to 38 percent on the question immediately prior to November’s election —  stood as a major reason for the closing of those horizons. Clinton described her vanquisher and his supporters as embracing a “lie,” a “con,” “alternative facts,” and “a   assault on truth and reason. ” She failed to explain why the American people chose his lies over her truth. “As the history majors among you here today know all too well, when people in power invent their own facts and attack those who question them, it can mark the beginning of the end of a free society,” she offered. “That is not hyperbole. ” Like so many people to emerge from the 1960s, Hillary Clinton embarked upon a long, strange trip. From high school Goldwater Girl and Wellesley College Republican president to Democratic politician, Clinton drank in the times and the place that gave her a degree. More significantly, she went from idealist to cynic, as a comparison of her two Wellesley commencement addresses show. Way back when, she lamented that “for too long our leaders have viewed politics as the art of the possible, and the challenge now is to practice politics as the art of making what appears to be impossible possible. ” Now, as the big woman on campus but the odd woman out of the White House, she wonders how her current station is even possible. “Why aren’t I 50 points ahead?” she asked in September. In May she asks why she isn’t president. The woman famously dubbed a “congenital liar” by Bill Safire concludes that lies did her in —  theirs, mind you, not hers. Getting stood up on Election Day, like finding yourself the jilted bride on your wedding day, inspires dangerous delusions.',
    label: 0
  }
]
```

Количество фейков и всего статей по авторам

**```otus> db.fakenews.aggregate([{$group:{_id:"$author", count_fake:{$sum:"$label"}, count_all:{$sum:1}}}])```**
```js
[
  { _id: 'Tara Siegel Bernard', count_fake: 0, count_all: 4 },
  { _id: 'Gretchen Reynolds', count_fake: 0, count_all: 10 },
  { _id: 'Ildefonso Ortiz', count_fake: 0, count_all: 17 },
  { _id: 'Citizen Satirist', count_fake: 2, count_all: 2 },
  { _id: 'darkbake', count_fake: 1, count_all: 1 },
  { _id: 'TNA Video', count_fake: 2, count_all: 2 },
  { _id: 'Jekaterina Sinelschtschikowa', count_fake: 2, count_all: 2 },
  { _id: 'Bob Livingston', count_fake: 1, count_all: 1 },
  {
    _id: 'Michael S. Schmidt and Eric Lichtblau',
    count_fake: 0,
    count_all: 1
  },
  { _id: 'AssHat900', count_fake: 1, count_all: 1 },
  { _id: 'Nicola Clark', count_fake: 0, count_all: 2 },
  {
    _id: 'Anne Barnard and Michael R. Gordon',
    count_fake: 0,
    count_all: 2
  },
  { _id: 'Jodi Kantor', count_fake: 0, count_all: 1 },
  { _id: 'Karen Workman', count_fake: 0, count_all: 2 },
  {
    _id: 'Adam Nagourney and Jonathan Martin',
    count_fake: 0,
    count_all: 1
  },
  {
    _id: 'Michael Paulson and Michael Barbaro',
    count_fake: 0,
    count_all: 1
  },
  { _id: 'Caryn Ganz', count_fake: 0, count_all: 1 },
  { _id: 'Sheri Fink and James Risen', count_fake: 0, count_all: 1 },
  {
    _id: 'Adam Goldman, Eric Lichtblau and Matt Apuzzo',
    count_fake: 0,
    count_all: 1
  },
  { _id: 'Natalie Dailey', count_fake: 7, count_all: 7 }
]
```

Рейтинг от правдивых-плодовитых до лгунов

**```
otus> db.fakenews.aggregate([{$group:{_id:"$author", count_fake:{$sum:"$label"}, count_all:{$sum:1}}}]).sort({"count_fake":1,"count_all":-1})```**
```js
[
  { _id: 'Jerome Hudson', count_fake: 0, count_all: 166 },
  { _id: 'Charlie Spiering', count_fake: 0, count_all: 141 },
  { _id: 'John Hayward', count_fake: 0, count_all: 140 },
  { _id: 'Katherine Rodriguez', count_fake: 0, count_all: 124 },
  { _id: 'Warner Todd Huston', count_fake: 0, count_all: 122 },
  { _id: 'Ian Hanchett', count_fake: 0, count_all: 119 },
  { _id: 'Breitbart News', count_fake: 0, count_all: 118 },
  { _id: 'Daniel Nussbaum', count_fake: 0, count_all: 112 },
  { _id: 'Jeff Poor', count_fake: 0, count_all: 107 },
  { _id: 'AWR Hawkins', count_fake: 0, count_all: 107 },
  { _id: 'Joel B. Pollak', count_fake: 0, count_all: 106 },
  { _id: 'Trent Baker', count_fake: 0, count_all: 102 },
  { _id: 'Breitbart London', count_fake: 0, count_all: 97 },
  { _id: 'Bob Price', count_fake: 0, count_all: 93 },
  { _id: 'Ben Kew', count_fake: 0, count_all: 90 },
  { _id: 'Charlie Nash', count_fake: 0, count_all: 88 },
  { _id: 'Lucas Nolan', count_fake: 0, count_all: 80 },
  { _id: 'John Binder', count_fake: 0, count_all: 75 },
  { _id: 'Breitbart Jerusalem', count_fake: 0, count_all: 75 },
  { _id: 'Penny Starr', count_fake: 0, count_all: 67 }
]
```

Авторы, которые допустили хотя бы одну фейковую новость

**```
otus> db.fakenews.aggregate([{$group:{_id:"$author", count_fake:{$sum:"$label"}, count_all:{$sum:1}}},{$match:{"count_fake":{$gt:0}}}]).sort({"count_fake":1,"count_all":-1})```**
```js
[
  { _id: 'Pam Key', count_fake: 1, count_all: 243 },
  { _id: 'AFP', count_fake: 1, count_all: 3 },
  { _id: 'Alex Lantier', count_fake: 1, count_all: 1 },
  { _id: 'Jacques de Seingalt', count_fake: 1, count_all: 1 },
  { _id: 'Dairy✓ᵀᴿᵁᴹᴾ', count_fake: 1, count_all: 1 },
  { _id: 'William Dunkerley', count_fake: 1, count_all: 1 },
  { _id: 'Orang kiyani', count_fake: 1, count_all: 1 },
  { _id: 'ColdTrifecta', count_fake: 1, count_all: 1 },
  { _id: 'Daniel Thomas', count_fake: 1, count_all: 1 },
  { _id: 'Greg Corombos', count_fake: 1, count_all: 1 },
  { _id: 'Chris Wilson', count_fake: 1, count_all: 1 },
  { _id: 'Amanda', count_fake: 1, count_all: 1 },
  { _id: 'Dolores Vek', count_fake: 1, count_all: 1 },
  { _id: 't172048', count_fake: 1, count_all: 1 },
  { _id: 'Swath - PROUD DEPLORABLE', count_fake: 1, count_all: 1 },
  { _id: 'David Macaray', count_fake: 1, count_all: 1 },
  { _id: 'Murray Dobbin', count_fake: 1, count_all: 1 },
  { _id: 'starmount', count_fake: 1, count_all: 1 },
  {
    _id: 'Anonymous Coward (UID 73266834)',
    count_fake: 1,
    count_all: 1
  },
  { _id: 'Frankie', count_fake: 1, count_all: 1 }
]
```

Пишут правду и ложь

**```
otus> db.fakenews.aggregate([{$group:{_id:"$author", count_fake:{$sum:"$label"}, count_all:{$sum:1}}},{$match:{$and:[{$expr:{$gt:["$count_all", "$count_fake"]}},{"count_fake":{$gt:0}}]}}]).sort({"count_fake":1,"count_all":-1})
```**

```js
[
  { _id: 'Pam Key', count_fake: 1, count_all: 243 },
  { _id: 'AFP', count_fake: 1, count_all: 3 },
  { _id: 'Reuters', count_fake: 2, count_all: 6 },
  { _id: 'Pamela Geller', count_fake: 4, count_all: 5 },
  { _id: 'Ann Coulter', count_fake: 5, count_all: 21 },
  { _id: NaN, count_fake: 1931, count_all: 1957 }
]
```

-----
db.fakenews.aggregate([{$group:{_id:"$author", count_fake:{$sum:"$label"}, count_all:{$sum:1}}},{$set:{total:{$sum:"$count_all"}}},{$match:{"count_fake":{$gt:0}}}]).sort({"count_fake":1,"count_all":-1})
db.fakenews.aggregate([
{
$bucket:{groupBy:"$author", output:{count_fake:{$sum:"$label"}, count_all:{$sum:1}}}
},
{$match:{count_fake:{$lt:"$count_all"}}}
]).sort({"count_fake":1,"count_all":-1})
