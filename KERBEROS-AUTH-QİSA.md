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

Clientdə artıq TGT və TGS_SESSİON_KEY var və client öz nt hashi ilə şifrlənmiş TGS_SESSİON_KEY-i deşifrə edir. Clientdə var: TGT və TGS_SESSİON_KEY dəyəri. 

<img width="361" height="153" alt="image" src="https://github.com/user-attachments/assets/cf58f965-3c98-46ed-93eb-60aa0df91e56" />

Client hər hansı bir servisə daxil olmaq istəyən zaman artıq TGS_REQ göndərməlidir. Bu zaman client TGS_SESSİON_KEY dəyəri ilə username/timestapmi şifrləyir. Client TGS_REQ-də aşağıdakıları TGS-ə göndərir:

**TGT** + **SPN** + **TGS_SESSION_KEY_DƏYƏRİ İLƏ ŞİFRLƏNMİŞ username/timestamp**
