İstifadəçi username və password yazır. Client kompüter password‑u serverə göndərmir. Əvvəlcə password lokal olaraq NT hash‑ə çevrilir (lsass edir bunu). Sonra client serverə login istəyi göndərir. Server istifadəçinin kimliyini yoxlamaq üçün 16 baytlıq random challenge göndərir.

Client artıq hesablanmış NT hash‑i götürür və həmin challenge ilə birlikdə yeni bir dəyər — NTLM response yaradır. Bu response və username serverə göndərilir.

Server gələn username‑ə görə SAM (lokal sistemdə) və ya NTDS.dit (Active Directory‑də) içindən həmin istifadəçinin saxlanmış NT hash‑ini tapır. Daha sonra server eyni challenge və saxlanmış NT hash ilə özü də response hesablayır. Əgər serverin hesabladığı nəticə clientdən gələn response ilə eyni olarsa, istifadəçi doğrulanır və girişə icazə verilir.

Yəni NTLM‑də:

password heç vaxt şəbəkə ilə göndərilmir,

hash də birbaşa göndərilmir,

server sadəcə istifadəçinin hash‑ə sahib olduğunu yoxlayır.

<img width="1360" height="445" alt="image" src="https://github.com/user-attachments/assets/46bd4447-5025-4c4d-884f-aaaa2adb3fa5" />

Movcud ola bilecek hucumlar:

<img width="171" height="208" alt="image" src="https://github.com/user-attachments/assets/0b190822-67b9-469a-ba80-9aaf6d600c78" />
