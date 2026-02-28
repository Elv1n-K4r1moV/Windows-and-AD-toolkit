# 1. AS‑REQ 

<img width="344" height="114" alt="image" src="https://github.com/user-attachments/assets/f2d61561-8821-4af3-af17-abe6b0ed539c" />

User password yazır. LSASS passwordu NT hashə çevirir. Və buna UserKey deyilir. 

*Timestamp* yaradılır və bu Timestamp User NT hash ilə şifrələnir. Və **Username** və **şifrlənmiş timestamp** *AS_req* kimi KDC-ə(daha dəqiq Authentication Serverə) göndərilir.

# 2. AS-REP 

KDC timestampi açmaq üçün *ntds.dit*-dən userin hashini götürür və timetampi deşifrə etməyə çalışır əgər deşifrə oldusa onda doğrulama tamamlandı. 

<img width="344" height="119" alt="image" src="https://github.com/user-attachments/assets/dbbbe484-a4ae-4dd3-ad0b-aab523f056a4" />

Bu halda KDC TGT yaradır. TGT yaratmaq üçün TGT data olmalı və bu TGT data  içində: *Username*, *SID*, *TGS Session Key* (random olaraq yaradılan dəyər) olmalıdır. Və bu TGT data KRBTGT hesabının NT hashi ilə şifrlənir. Və **TGT** hazırdı.

Həmçinin TGS Session Key bu dəfə userin NT hashi ilə şifrlənir və TGT ilə birgə AS_REP kimi göndərilir. Yəni **TGT** və **Userin NT hashi ilə şifrlənmiş TGS_SESSİON_KEY**

# 3. TGS‑REQ

Clientdə artıq TGT və Şifrlənmiş_TGS_SESSİON_KEY var və client öz nt hashi ilə şifrlənmiş TGS_SESSİON_KEY-i deşifrə edir. Clientdə var: TGT və TGS_SESSİON_KEY dəyəri. 

<img width="361" height="153" alt="image" src="https://github.com/user-attachments/assets/cf58f965-3c98-46ed-93eb-60aa0df91e56" />

Client hər hansı bir servisə daxil olmaq istəyən zaman artıq TGS_REQ göndərməlidir. Bu zaman client TGS_SESSİON_KEY dəyəri ilə username/timestapmi şifrləyir. Client TGS_REQ-də aşağıdakıları TGS-ə göndərir:

**TGT** + **SPN** + **TGS_SESSION_KEY_DƏYƏRİ İLƏ ŞİFRLƏNMİŞ username/timestamp**

# 4. TGS-REP

Ticket Granted Ticket(TGS) TGT-ni krgtgt hesabının nt hashi ilə deşifrə edir və onun içindəki TGS_SESSİON_KEY_DƏYƏRİNİ götürür. Və bu dəyər ilə timestamp/username-i deşifrə edir. Və beləcə doğrulamanı yoxlayır. 

<img width="373" height="120" alt="image" src="https://github.com/user-attachments/assets/d8f94a17-0ee4-41a9-a05f-bddaa2a036c8" />

Doğrulama tamamlandıqdan sonra TGS mənə Service Ticket hazırlayır. Service Ticket yaratmaq üçün Server Accountun NT hashi və Service Ticket data lazımdır və bu data içində *username*, *sid*, *service session key(random olaraq yaradılan dəyər)* olur. Bu Service Ticket Data Server Accountun NT hashi şifrlənir. Həmçinin service session key bu dəfə də TGS Session Key ilə şifrlənir və Service Ticket ilə birgə clientə göndərilir. Yəni TGS_REP-də göndərilir:

**Service Ticket** və **TGS_Session_Key ilə şifrlənmiş Service Session Key** 

# 5. Application Service Request

Clientdə artıq Service Ticket və TGS_Session_Key_dəyəri ilə şifrlənmiş Service Session Key. Client AS_REP-də ona verilən TGS_Session_Key-i TGS_req-də öz nt hashi ilə şifrləndiyindən deşifrə edib dəyərini çıxara bilmişdi. Buna görədə bu mərhələdə onda artıq TGS_Session_Key dəyəri zatən var. Və client bu TGS_Session_Key_dəyəri ilə şifrlənmiş Service Session Key-i asanlıqla deşifrə edir və dəyərini görə bilir.

Bunu etdikdən sonra Clientdə var. Service Ticket və Service Session Key dəyəri.

<img width="384" height="151" alt="image" src="https://github.com/user-attachments/assets/6790ce18-7f8e-4c43-8d5b-934469e03cbb" />

Client bu dəfə username və timestamp dəyərini Service Session Key dəyəri ilə şifrləyir və Service Ticket ilə birgə Application Serverə göndərir.

# 6. Application Service tərəfində yoxlama

Application Service *Service Ticket*i öz Service Accountunun nt hashi ilə şifrləndiyindən asanlıqla deşifrə edir və onun içindəki Service Session Key dəyərini götürür. Və bu dəyər ilə timestamp/username-i deşifrə edir. Və beləcə yoxlama uğurludursa istifadəçinin servisə girişinə icazə verilir.




