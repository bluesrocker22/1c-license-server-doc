### Сервер лицензирования для 1с

---

#### Общее описание

* Имя сервера: lic1c.superpuperdomain.loc
* IP-адрес: 192.168.199.199
* ОС Deban 11.6 x86-64

---
#### Принцип работы

Все по документации от 1С или [с инфостарта](https://infostart.ru/1c/articles/1027743/) - добавляем сервер в кластер, назначаем и применяем требования назначения функциональности

---

#### Порядок активации новых лицензий на сервере лицензирования

* ***Важно!** Выполняем команды 5 из под root'а, иначе команда генерации лицензии не положит готовый файл в папку /var/1C/licenses/*

1. определим номер комплекта, пин и предыдущий пин, создадим рабочую папку под следующую операцию

```
regnumber=XXXXXXXXX
pin=111-111-111-111-111
prpin=222-222-222-222-222
work=/home/superpuperadmin/1c-licenses/
tools=/home/superpuperadmin/1c-licenses/$regnumber
mkdir $tools
```

2. создадим файл запроса
(поправить в соответствии с данными выше, при первом получении убрать параметр --prpin!!!)

```
ring license prepare-request --company "ООО ЧОО \"Рога и копыта\"" --last-name "Бендер" --first-name "Остап" --middle-name Ибрагимович" --email ostapbender@rogaikopyta.loc --country "RU" --zip-code "357504" --town "Пятигорск" --street "Бульвар Гагарина" --house "1" --apartment "1" --validate --serial $regnumber --pin $pin --previous-pin $prpin --request $tools/LicData_$regnumber.txt >> $tools/lic_$regnumber.log
```

3. запросим лицензию у 1с

```
ring license acquire --request $tools/LicData_$regnumber.txt --response $tools/response_$regnumber.txt >> $tools/lic_$regnumber.log
```

4. сгенерируем файл лицензии

```
ring license generate --request $tools/LicData_$regnumber.txt --response $tools/response_$regnumber.txt --license $tools/license_$regnumber.txt >> $tools/lic_$regnumber.log
```

5. положим лицензию в хранилище

```
ring license put --license $tools/license_$regnumber.txt >> $tools/lic_$regnumber.log
```

6. назначим владельцем папки хранилища лицензий пользователя 1с

```
chown -R usr1cv8 /var/1C/licenses/
```

7. сгенерируем новый список лицензий на сервере

```
ring license list >> $work/1c_licenses_$(date '+%F_%H-%M').txt
```

---

#### Как посмотреть информацию о лицензиях:

```
ring license list --send-statistics false
ring license info --name xxxxxxxxxxxxxxx-xxxxxxxxx --send-statistics false
```

* _Параметры сервера, к которым привязана лицензия **не менять ни в коем случае**_!

Можно посмотреть базовую информацию о привязке лицензии в файле лицензии, полученном с серверов 1с, - там в конце файла есть данные о сервере

На нашем сервере данная информация лежит здесь:

> /home/superpuperadmin/1c-licenses/<номер комплекта>/license_xxx.txt

> Привязка: lic1c, , 1478Mb RAM, 11.6 x86_64
---
**Использованные материалы:**

*<https://habr.com/ru/post/575080/>*

*<https://kifarunix.com/install-java-11java-17java-18-on-debian-11/>*

*<https://its.1c.ru/db/v8313doc#bookmark:adm:TI000000689>*

*<https://its.1c.ru/db/v838doc#bookmark:adm:TI000000665>*

**Если ~~ничего~~ что-то не работает:**

Проблема | Решение | Примечание
--- | --- | ---
Java ругается при каждом запросе | Это нормально (потому что 11-й версия старая и методы работы с ней используются старые)| Можно попробовать подключить Liberica JDK
Ничего не работает | Проверь версию Java - должна быть не новее 11-й | Можно попробовать подключить Liberica JDK

*Как подключить импортозамещенное java-окружение инструкции не существует (сама машина Liberica ставится вместе с 1с, но путь к ней нужно подбирать эмпирическим путем, видимо, она нигде не прописывает себя, или прописывет, но это не задокументировано, но и фиг с ним). Если не обновлять Java JDK, то и проблем не возникнет - поэтому просьба в эту трясину не лезть шаловливыми ручками*

---

#### Примечание: порядок поиска лицензий сервером 1С

Только для 8.3 и клиент-серверного режима работы (как у нас)

![картинка с инфостарта](https://infostart.ru/upload/iblock/e8d/e8df16c52c025fe23e915db50df46ac9.png)

