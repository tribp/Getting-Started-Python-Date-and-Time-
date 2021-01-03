# Python Date and Time

## Intro

### Python has 5 classes:

    - Date: Year / Month / Day
    - Time: Hour/ Minutes / Seconds / Microseconds 
    - Datetime: = Date + Time
    - Timedelta: datetime.timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0)
        -> returns a date. Commonly used for adding/substracting from an initial date, creating a new date, a delta away.
    - tzinfo: is a 'abstract' class with the timezone info, abstract meaning that it can not be created directly
   
## Concept of "Aware" and "Naive" objects

    - Naive date = date with no context of timezone
        -> eg: datatime.datetime(2020,12,25,20,30,15)
            -> It is like a watch indicating 20h30, but are you in New York, Sydney, Brussels or ...?
            -> when printed there will be NO reference to UTC like +01:00
           
    - Aware date = Native date +  a timezone
    
Python doesn not have build-in timezone support. 

**You have to use:**

    - pytz 
    or
    - dateutils



What challenges do we have?

    - a time should be referenced to a timezone
    - we have to be able to cope with DST - Daylight Saving Time. When DST is applied in a timezone, we can be confronted with two times the same time when the clock is reset 1 hour. That night we haven 02h:00 and 1 hour later again 02h:00


```python
import datetime
unaware = datetime.datetime(2020,12,26,16,4,59)
print(unaware)
```

    2020-12-26 16:04:59



```python
# now() creates an unaware date ! so no timezone info or context
unaware = datetime.datetime.now()
print(unaware)
```

    2020-12-27 15:28:27.948268



```python
# if we only want seconds and no milli or microseconds
only_secs = unaware.replace(microsecond =0)
print(only_secs)
# timestamp converts to amount of secs from 1/1/1970
print(only_secs.timestamp())
```

    2020-12-27 15:28:27
    1609079307.0


## Making 'naive' time, time 'Aware'

In order to do so we need a 'timezone object', so we can **link** our "unaware' time to a defined timezone and create an **'Aware'** time.

We have 2 possible solutions (libraries) to create a timezone object:

    - pytz: 
        -> step 1: create a timezone object
        -> step 2: link the time to the timezone via 'localize' or astimezone()
    - dateutil:
        -> step 1: create a timezone object
        -> step 2: create your time with tzinfo = 'yourTimeZone' or use astimezone(

### How to create a timezone object with pytz


```python
import pytz
import datetime

brussels = pytz.timezone('Europe/Brussels')
```

### How to create a timezone object with dateutil


```python
from dateutil import tz
import datetime

brugge =tz.gettz('Europe/Brussels')
```

### How to create a 'Aware' time with pytz

**Important !:** We can NOT use the pytz created timezone in the datetime object -> tzinfo = myTimeZone !!!

eg: datetime.datetime(2020,12,25,20,30,15, tzinfo = brussels)


```python
import pytz
brussels=pytz.timezone('Europe/Brussels')

unaware = datetime.datetime(2020,12,26,16,4,59)

# Method 1: with localize
aware10 = brussels.localize(datetime.datetime(2020,12,26,16,4,59))
print(aware10)

# Method 2: with astimezone
aware11 = unaware.astimezone(brussels)
print(aware11)
```

    2020-12-26 16:04:59+01:00
    2020-12-26 16:04:59+01:00


### How to create a 'Aware' time with dateutil


```python
from dateutil import tz
import datetime

brugge =tz.gettz('Europe/Brussels')

# Method 1: directly when creating the time (something that can NOT be done with puytz !!)
aware20 = datetime.datetime(2020,12,26,16,4,59,tzinfo=brugge)
aware21 = datetime.datetime.utcnow().astimezone(tz=brugge)

unaware20= datetime.datetime.now()

# Method 2: with astimezone
aware22= unaware20.astimezone(tz=brugge)
aware23= datetime.datetime.now().astimezone(tz=tz.UTC)

print(aware20)
print(aware21)
print(unaware10)
print(aware22)
print(aware23)
```

    2020-12-26 16:04:59+01:00
    2020-12-27 14:28:53.100898+01:00
    2020-12-27 15:04:35.598543
    2020-12-27 15:28:53.100971+01:00
    2020-12-27 14:28:53.101049+00:00


## DST - Daylight Saving Time

This is usefull to indicate the difference between 02h00 and 02h00, one hour later when DST change takes place


```python
my_summerdate = datetime.datetime(2020,7,1,8,59,59,tzinfo=brugge)

# dst() indicates of Daylight Saving Time is active or not
print(my_summerdate.dst())

# my_first_02H00 at DST change
unaware = datetime.datetime(2020,10,25,2,0,0)

my_first_02h00 = brussels.localize(unaware,is_dst=True)
my_second_02h00 = brussels.localize(unaware,is_dst=False)
print(my_first_02h00)
print(my_first_02h00.timestamp())
print(my_second_02h00)
print(my_second_02h00.timestamp())
```

    1:00:00
    2020-10-25 02:00:00+02:00
    1603584000.0
    2020-10-25 02:00:00+01:00
    1603587600.0


## Iso 8601 = defacto standard

    - format: YYYY-MM-DDThh:mm:ss+UTC offset or 'Z = Zulu time = +00:00'
        - example: 2020-12-26T16:04:59+01:00
    - format: seconds since 1/1/1970
        -> ideal as timestamp in databases
        -> gives microsecs precision
            example: 2020-12-26 17:18:13.894596 = 1608999493.894596 
                -> 1608999493 secs
                -> 894 millisecs
                -> 596 microsecs


```python
my_iso_datetime = aware.isoformat()
print(my_iso_datetime)
```

    2020-12-26T16:04:59+01:00



```python
my_iso_datetime_in_utc = aware.astimezone(pytz.UTC).isoformat()
print(my_iso_datetime_in_utc)
```

    2020-12-26T15:04:59+00:00



```python
my_timestamp = aware.timestamp()
print(aware)
print(my_timestamp)
print(datetime.datetime.fromtimestamp(my_timestamp,tz=pytz.utc).isoformat()+'Z')
```

    2020-12-26 16:04:59+01:00
    1608995099.0
    2020-12-26T15:04:59+00:00Z


## Timedelta example

datetime.timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0)


```python
today =datetime.datetime.now()
tomorrow_sametime = today + datetime.timedelta(days=1)
print(today)
print(tomorrow_sametime)
```

    2020-12-26 17:18:13.894596
    2020-12-27 17:18:13.894596


## Example:


```python
# we created 2 timezones: brussels and utc = +00:00 = Z (Zulu)
brussels = tz.gettz('Europe/Brussels')
my_utc_timezone = tz.UTC

# we create a datetime -> now() is 'naive -> with no context of knowledge of timezone'
my_time = datetime.datetime.now()

# we create 2 timezone 'Aware' times: both are NOT the same "occurence". eg same 'clock' but exist in a different timezone
my_local_time = my_time.astimezone(tz=brussels)
my_utc_time = my_time.astimezone(tz=my_utc_timezone)

print(my_local_time)
print(my_utc_time)

# My clock in Brussels but seen and referenced from the "zulu" UTC timezone (+00:00) aka GMT
print(my_local_time.astimezone(tz.UTC))
print(my_local_time.astimezone(tz.UTC).isoformat()[:-6]+'Z')

```

    2020-12-27 15:46:41.912484+01:00
    2020-12-27 14:46:41.912484+00:00
    2020-12-27 14:46:41.912484+00:00
    2020-12-27T14:46:41.912484Z


## List all available timezones


```python
for tz in pytz.all_timezones:
    print(tz)
```

    Africa/Abidjan
    Africa/Accra
    Africa/Addis_Ababa
    Africa/Algiers
    Africa/Asmara
    Africa/Asmera
    Africa/Bamako
    Africa/Bangui
    Africa/Banjul
    Africa/Bissau
    Africa/Blantyre
    Africa/Brazzaville
    Africa/Bujumbura
    Africa/Cairo
    Africa/Casablanca
    Africa/Ceuta
    Africa/Conakry
    Africa/Dakar
    Africa/Dar_es_Salaam
    Africa/Djibouti
    Africa/Douala
    Africa/El_Aaiun
    Africa/Freetown
    Africa/Gaborone
    Africa/Harare
    Africa/Johannesburg
    Africa/Juba
    Africa/Kampala
    Africa/Khartoum
    Africa/Kigali
    Africa/Kinshasa
    Africa/Lagos
    Africa/Libreville
    Africa/Lome
    Africa/Luanda
    Africa/Lubumbashi
    Africa/Lusaka
    Africa/Malabo
    Africa/Maputo
    Africa/Maseru
    Africa/Mbabane
    Africa/Mogadishu
    Africa/Monrovia
    Africa/Nairobi
    Africa/Ndjamena
    Africa/Niamey
    Africa/Nouakchott
    Africa/Ouagadougou
    Africa/Porto-Novo
    Africa/Sao_Tome
    Africa/Timbuktu
    Africa/Tripoli
    Africa/Tunis
    Africa/Windhoek
    America/Adak
    America/Anchorage
    America/Anguilla
    America/Antigua
    America/Araguaina
    America/Argentina/Buenos_Aires
    America/Argentina/Catamarca
    America/Argentina/ComodRivadavia
    America/Argentina/Cordoba
    America/Argentina/Jujuy
    America/Argentina/La_Rioja
    America/Argentina/Mendoza
    America/Argentina/Rio_Gallegos
    America/Argentina/Salta
    America/Argentina/San_Juan
    America/Argentina/San_Luis
    America/Argentina/Tucuman
    America/Argentina/Ushuaia
    America/Aruba
    America/Asuncion
    America/Atikokan
    America/Atka
    America/Bahia
    America/Bahia_Banderas
    America/Barbados
    America/Belem
    America/Belize
    America/Blanc-Sablon
    America/Boa_Vista
    America/Bogota
    America/Boise
    America/Buenos_Aires
    America/Cambridge_Bay
    America/Campo_Grande
    America/Cancun
    America/Caracas
    America/Catamarca
    America/Cayenne
    America/Cayman
    America/Chicago
    America/Chihuahua
    America/Coral_Harbour
    America/Cordoba
    America/Costa_Rica
    America/Creston
    America/Cuiaba
    America/Curacao
    America/Danmarkshavn
    America/Dawson
    America/Dawson_Creek
    America/Denver
    America/Detroit
    America/Dominica
    America/Edmonton
    America/Eirunepe
    America/El_Salvador
    America/Ensenada
    America/Fort_Nelson
    America/Fort_Wayne
    America/Fortaleza
    America/Glace_Bay
    America/Godthab
    America/Goose_Bay
    America/Grand_Turk
    America/Grenada
    America/Guadeloupe
    America/Guatemala
    America/Guayaquil
    America/Guyana
    America/Halifax
    America/Havana
    America/Hermosillo
    America/Indiana/Indianapolis
    America/Indiana/Knox
    America/Indiana/Marengo
    America/Indiana/Petersburg
    America/Indiana/Tell_City
    America/Indiana/Vevay
    America/Indiana/Vincennes
    America/Indiana/Winamac
    America/Indianapolis
    America/Inuvik
    America/Iqaluit
    America/Jamaica
    America/Jujuy
    America/Juneau
    America/Kentucky/Louisville
    America/Kentucky/Monticello
    America/Knox_IN
    America/Kralendijk
    America/La_Paz
    America/Lima
    America/Los_Angeles
    America/Louisville
    America/Lower_Princes
    America/Maceio
    America/Managua
    America/Manaus
    America/Marigot
    America/Martinique
    America/Matamoros
    America/Mazatlan
    America/Mendoza
    America/Menominee
    America/Merida
    America/Metlakatla
    America/Mexico_City
    America/Miquelon
    America/Moncton
    America/Monterrey
    America/Montevideo
    America/Montreal
    America/Montserrat
    America/Nassau
    America/New_York
    America/Nipigon
    America/Nome
    America/Noronha
    America/North_Dakota/Beulah
    America/North_Dakota/Center
    America/North_Dakota/New_Salem
    America/Nuuk
    America/Ojinaga
    America/Panama
    America/Pangnirtung
    America/Paramaribo
    America/Phoenix
    America/Port-au-Prince
    America/Port_of_Spain
    America/Porto_Acre
    America/Porto_Velho
    America/Puerto_Rico
    America/Punta_Arenas
    America/Rainy_River
    America/Rankin_Inlet
    America/Recife
    America/Regina
    America/Resolute
    America/Rio_Branco
    America/Rosario
    America/Santa_Isabel
    America/Santarem
    America/Santiago
    America/Santo_Domingo
    America/Sao_Paulo
    America/Scoresbysund
    America/Shiprock
    America/Sitka
    America/St_Barthelemy
    America/St_Johns
    America/St_Kitts
    America/St_Lucia
    America/St_Thomas
    America/St_Vincent
    America/Swift_Current
    America/Tegucigalpa
    America/Thule
    America/Thunder_Bay
    America/Tijuana
    America/Toronto
    America/Tortola
    America/Vancouver
    America/Virgin
    America/Whitehorse
    America/Winnipeg
    America/Yakutat
    America/Yellowknife
    Antarctica/Casey
    Antarctica/Davis
    Antarctica/DumontDUrville
    Antarctica/Macquarie
    Antarctica/Mawson
    Antarctica/McMurdo
    Antarctica/Palmer
    Antarctica/Rothera
    Antarctica/South_Pole
    Antarctica/Syowa
    Antarctica/Troll
    Antarctica/Vostok
    Arctic/Longyearbyen
    Asia/Aden
    Asia/Almaty
    Asia/Amman
    Asia/Anadyr
    Asia/Aqtau
    Asia/Aqtobe
    Asia/Ashgabat
    Asia/Ashkhabad
    Asia/Atyrau
    Asia/Baghdad
    Asia/Bahrain
    Asia/Baku
    Asia/Bangkok
    Asia/Barnaul
    Asia/Beirut
    Asia/Bishkek
    Asia/Brunei
    Asia/Calcutta
    Asia/Chita
    Asia/Choibalsan
    Asia/Chongqing
    Asia/Chungking
    Asia/Colombo
    Asia/Dacca
    Asia/Damascus
    Asia/Dhaka
    Asia/Dili
    Asia/Dubai
    Asia/Dushanbe
    Asia/Famagusta
    Asia/Gaza
    Asia/Harbin
    Asia/Hebron
    Asia/Ho_Chi_Minh
    Asia/Hong_Kong
    Asia/Hovd
    Asia/Irkutsk
    Asia/Istanbul
    Asia/Jakarta
    Asia/Jayapura
    Asia/Jerusalem
    Asia/Kabul
    Asia/Kamchatka
    Asia/Karachi
    Asia/Kashgar
    Asia/Kathmandu
    Asia/Katmandu
    Asia/Khandyga
    Asia/Kolkata
    Asia/Krasnoyarsk
    Asia/Kuala_Lumpur
    Asia/Kuching
    Asia/Kuwait
    Asia/Macao
    Asia/Macau
    Asia/Magadan
    Asia/Makassar
    Asia/Manila
    Asia/Muscat
    Asia/Nicosia
    Asia/Novokuznetsk
    Asia/Novosibirsk
    Asia/Omsk
    Asia/Oral
    Asia/Phnom_Penh
    Asia/Pontianak
    Asia/Pyongyang
    Asia/Qatar
    Asia/Qostanay
    Asia/Qyzylorda
    Asia/Rangoon
    Asia/Riyadh
    Asia/Saigon
    Asia/Sakhalin
    Asia/Samarkand
    Asia/Seoul
    Asia/Shanghai
    Asia/Singapore
    Asia/Srednekolymsk
    Asia/Taipei
    Asia/Tashkent
    Asia/Tbilisi
    Asia/Tehran
    Asia/Tel_Aviv
    Asia/Thimbu
    Asia/Thimphu
    Asia/Tokyo
    Asia/Tomsk
    Asia/Ujung_Pandang
    Asia/Ulaanbaatar
    Asia/Ulan_Bator
    Asia/Urumqi
    Asia/Ust-Nera
    Asia/Vientiane
    Asia/Vladivostok
    Asia/Yakutsk
    Asia/Yangon
    Asia/Yekaterinburg
    Asia/Yerevan
    Atlantic/Azores
    Atlantic/Bermuda
    Atlantic/Canary
    Atlantic/Cape_Verde
    Atlantic/Faeroe
    Atlantic/Faroe
    Atlantic/Jan_Mayen
    Atlantic/Madeira
    Atlantic/Reykjavik
    Atlantic/South_Georgia
    Atlantic/St_Helena
    Atlantic/Stanley
    Australia/ACT
    Australia/Adelaide
    Australia/Brisbane
    Australia/Broken_Hill
    Australia/Canberra
    Australia/Currie
    Australia/Darwin
    Australia/Eucla
    Australia/Hobart
    Australia/LHI
    Australia/Lindeman
    Australia/Lord_Howe
    Australia/Melbourne
    Australia/NSW
    Australia/North
    Australia/Perth
    Australia/Queensland
    Australia/South
    Australia/Sydney
    Australia/Tasmania
    Australia/Victoria
    Australia/West
    Australia/Yancowinna
    Brazil/Acre
    Brazil/DeNoronha
    Brazil/East
    Brazil/West
    CET
    CST6CDT
    Canada/Atlantic
    Canada/Central
    Canada/Eastern
    Canada/Mountain
    Canada/Newfoundland
    Canada/Pacific
    Canada/Saskatchewan
    Canada/Yukon
    Chile/Continental
    Chile/EasterIsland
    Cuba
    EET
    EST
    EST5EDT
    Egypt
    Eire
    Etc/GMT
    Etc/GMT+0
    Etc/GMT+1
    Etc/GMT+10
    Etc/GMT+11
    Etc/GMT+12
    Etc/GMT+2
    Etc/GMT+3
    Etc/GMT+4
    Etc/GMT+5
    Etc/GMT+6
    Etc/GMT+7
    Etc/GMT+8
    Etc/GMT+9
    Etc/GMT-0
    Etc/GMT-1
    Etc/GMT-10
    Etc/GMT-11
    Etc/GMT-12
    Etc/GMT-13
    Etc/GMT-14
    Etc/GMT-2
    Etc/GMT-3
    Etc/GMT-4
    Etc/GMT-5
    Etc/GMT-6
    Etc/GMT-7
    Etc/GMT-8
    Etc/GMT-9
    Etc/GMT0
    Etc/Greenwich
    Etc/UCT
    Etc/UTC
    Etc/Universal
    Etc/Zulu
    Europe/Amsterdam
    Europe/Andorra
    Europe/Astrakhan
    Europe/Athens
    Europe/Belfast
    Europe/Belgrade
    Europe/Berlin
    Europe/Bratislava
    Europe/Brussels
    Europe/Bucharest
    Europe/Budapest
    Europe/Busingen
    Europe/Chisinau
    Europe/Copenhagen
    Europe/Dublin
    Europe/Gibraltar
    Europe/Guernsey
    Europe/Helsinki
    Europe/Isle_of_Man
    Europe/Istanbul
    Europe/Jersey
    Europe/Kaliningrad
    Europe/Kiev
    Europe/Kirov
    Europe/Lisbon
    Europe/Ljubljana
    Europe/London
    Europe/Luxembourg
    Europe/Madrid
    Europe/Malta
    Europe/Mariehamn
    Europe/Minsk
    Europe/Monaco
    Europe/Moscow
    Europe/Nicosia
    Europe/Oslo
    Europe/Paris
    Europe/Podgorica
    Europe/Prague
    Europe/Riga
    Europe/Rome
    Europe/Samara
    Europe/San_Marino
    Europe/Sarajevo
    Europe/Saratov
    Europe/Simferopol
    Europe/Skopje
    Europe/Sofia
    Europe/Stockholm
    Europe/Tallinn
    Europe/Tirane
    Europe/Tiraspol
    Europe/Ulyanovsk
    Europe/Uzhgorod
    Europe/Vaduz
    Europe/Vatican
    Europe/Vienna
    Europe/Vilnius
    Europe/Volgograd
    Europe/Warsaw
    Europe/Zagreb
    Europe/Zaporozhye
    Europe/Zurich
    GB
    GB-Eire
    GMT
    GMT+0
    GMT-0
    GMT0
    Greenwich
    HST
    Hongkong
    Iceland
    Indian/Antananarivo
    Indian/Chagos
    Indian/Christmas
    Indian/Cocos
    Indian/Comoro
    Indian/Kerguelen
    Indian/Mahe
    Indian/Maldives
    Indian/Mauritius
    Indian/Mayotte
    Indian/Reunion
    Iran
    Israel
    Jamaica
    Japan
    Kwajalein
    Libya
    MET
    MST
    MST7MDT
    Mexico/BajaNorte
    Mexico/BajaSur
    Mexico/General
    NZ
    NZ-CHAT
    Navajo
    PRC
    PST8PDT
    Pacific/Apia
    Pacific/Auckland
    Pacific/Bougainville
    Pacific/Chatham
    Pacific/Chuuk
    Pacific/Easter
    Pacific/Efate
    Pacific/Enderbury
    Pacific/Fakaofo
    Pacific/Fiji
    Pacific/Funafuti
    Pacific/Galapagos
    Pacific/Gambier
    Pacific/Guadalcanal
    Pacific/Guam
    Pacific/Honolulu
    Pacific/Johnston
    Pacific/Kiritimati
    Pacific/Kosrae
    Pacific/Kwajalein
    Pacific/Majuro
    Pacific/Marquesas
    Pacific/Midway
    Pacific/Nauru
    Pacific/Niue
    Pacific/Norfolk
    Pacific/Noumea
    Pacific/Pago_Pago
    Pacific/Palau
    Pacific/Pitcairn
    Pacific/Pohnpei
    Pacific/Ponape
    Pacific/Port_Moresby
    Pacific/Rarotonga
    Pacific/Saipan
    Pacific/Samoa
    Pacific/Tahiti
    Pacific/Tarawa
    Pacific/Tongatapu
    Pacific/Truk
    Pacific/Wake
    Pacific/Wallis
    Pacific/Yap
    Poland
    Portugal
    ROC
    ROK
    Singapore
    Turkey
    UCT
    US/Alaska
    US/Aleutian
    US/Arizona
    US/Central
    US/East-Indiana
    US/Eastern
    US/Hawaii
    US/Indiana-Starke
    US/Michigan
    US/Mountain
    US/Pacific
    US/Samoa
    UTC
    Universal
    W-SU
    WET
    Zulu


# Working with the time library


```python
import time
ts_in_seconds = time.time()
ts_as_tuple = time.gmtime()
ts_as_string = time.strftime("%Y-%m-%dT%H:%M:%S", ts_as_tuple)
ts_as_iso_string = time.strftime("%Y-%m-%dT%H:%M:%S", ts_as_tuple)+ "+00:00Z"
```


```python
print(ts_in_seconds)
print(ts_as_tuple)
print(ts_as_string)
print(ts_as_iso_string)
```

    1609598177.087612
    time.struct_time(tm_year=2021, tm_mon=1, tm_mday=2, tm_hour=14, tm_min=36, tm_sec=17, tm_wday=5, tm_yday=2, tm_isdst=0)
    2021-01-02T14:36:17
    2021-01-02T14:36:17+00:00Z



```python
print(time.ctime(ts_in_seconds))
```

    Sun Jan  3 15:02:53 2021



```python
iso_time = time.strftime("%Y-%m-%dT%H:%M:%S", tuple_time)
print(iso_time)
```

    2021-01-02T10:19:06



```python
# When you need the Iso8601 time format
import time
import datetime
import pytz
my_timestamp = time.time()
print(my_timestamp)
print(datetime.datetime.fromtimestamp(my_timestamp,tz=pytz.utc).isoformat()+'Z')
```

    1609608129.8201559
    2021-01-02T17:22:09.820156+00:00Z

