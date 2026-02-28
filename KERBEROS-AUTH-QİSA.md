# 1. AS‑REQ 

<img width="344" height="114" alt="image" src="https://github.com/user-attachments/assets/f2d61561-8821-4af3-af17-abe6b0ed539c" />

User password yazır. LSASS passwordu NT hashə çevirir. Və buna UserKey deyilir. 

*Timestamp* yaradılır və bu Timestamp User NT hash ilə şifrələnir. Və **Username** və **şifrlənmiş timestamp** *AS_req* kimi KDC-ə göndərilir.

# 2. AS-REP 

KDC timestampi açmaq üçün *ntds.dit*-dən userin hashini götürür və timetampi deşifrə etməyə çalışır əgər deşifrə oldusa onda doğrulama tamamlandı. 

<img width="344" height="119" alt="image" src="https://github.com/user-attachments/assets/dbbbe484-a4ae-4dd3-ad0b-aab523f056a4" />

Bu halda KDC TGT yaradır. TGT yaratmaq üçün TGT data olmalı və bu TGT data  içində: *Username*, *SID*, *TGS Session Key* (random olaraq yaradılan dəyər) olmalıdır. Və bu TGT data KRBTGT hesabının NT hashi ilə şifrlənir. Və **TGT** hazırdı.

Həmçinin TGS Session Key bu dəfə userin NT hashi ilə şifrlənir və TGT ilə birgə AS_REP kimi göndərilir. Yəni **TGT** və **Userin NT hashi ilə şifrlənmiş TGS_SESSİON_KEY**

