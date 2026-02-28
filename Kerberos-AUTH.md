# AS-REQ

İstifadəçi username və password yazır. Client tərəfdəki LSASS password‑u götürür və onu NT hash‑ə çevirir. Bu NT hash artıq istifadəçinin **UserKey**‑idir.

LSASS cari vaxtdan bir **timestamp** yaradır.

Timestamp Yaradilmasi:

*OS‑dən cari vaxtı oxuyur*  --> *UTC formatına çevirir*   --> *Kerberos formatında saxlayır*     Misal: 20260225051432Z

Daha sonra bu **Timestamp** *UserKey* ( userin pass yazanda lsass-in yaratdigi nt hash ) ilə şifrələnir. Client bu məlumatı AS‑REQ sorğusu kimi Domain Controller‑də yerləşən KDC serverinə göndərir.

Timestamp-dam istifadenin meqsedi REPLAY hucumlarinin qarsisini almaqdir.

Əgər timestamp olmasa: attacker köhnə login paketini tutub sonra yenidən göndərə bilər.

Amma KDC yoxlayır: timestamp ≈ current time ?

<img width="269" height="173" alt="image" src="https://github.com/user-attachments/assets/42147d91-f949-4b2c-b3f2-273f508e00f5" />

Belelikle AS-REQ sorgusu KDC-e gonderilir.

<img width="1050" height="356" alt="image" src="https://github.com/user-attachments/assets/5d6057ed-94fe-4f81-9a9a-4cb5a1c9e40d" />


# AS-REP

KDC istifadəçini doğrulayır:

KDC Client‑dən gələn AS-REQ paketini alır. İçindəki Encrypted Timestamp‑ı UserKey ilə decrypt edir. 

KDC AD-də NTDS.DIT faylında saxlanan istifadəçi *NT hash‑i (UserKey)* ilə göndərilən timestamp‑ı decrypt edir, əgər decrypt uğurlu olarsa və timestamp ilə KDC‑nin cari vaxtı arasındakı fərq icazə verilən limitdədirsə (məsələn ±5 dəqiqə), istifadəçi uğurla doğrulanır. Her sey qaydasindadirsa artiq TGT yaradilir.

**TGT yaradılması**

TGT yaradılarkən lazım olan iki əsas komponent:

**1.** TGT Data – içində saxlanır:

Username / SID , Domain adı , Ticket validity (expiration) , Session Key (client ↔ TGS üçün) 

**2.** krbtgt hesabının NT hash‑i (gizli açar) – TGT‑ni encrypt etmək üçün istifadə olunur.

Yəni sadə şəkildə:

TGT = Encrypt(TGT Data, krbtgt NT hash)

Bu o deməkdir: *Client TGT‑ni oxuya bilmir* +  *yalnız KDC oxuya bilir* + *Client sadəcə TGT‑ni saxlayır və ötürür*

Daha sonra AS-REP response hazirlanir ve **AS‑REP paketində iki ayrı şey cliente gonderilir**:

1. **TGT**   2. **Session Key**

Session Key həm TGT data içində olur və bu zaman krbtgt hesabının nt hashi ilə hashlənir ama biridə var TGT ile birgə gələn. O ise artıq userin nt hashi ilə şifrlənir amma dəyərləri eynidir.

krbtgt NT hashi ilə hashlənmiş TGT + userin nt hashi ilə hashlənmiş session key.

## GOLDEN TICKET ATTACK-DA

Biz saxta TGT (Ticket Granting Ticket) yaradırıq. Bunu yaratmaq üçün ən vacib məlumat krbtgt hesabının NT hash‑idir. Əgər attacker krbtgt hash‑i əldə edərsə, artıq: TGT Data (username, domain, SID, groups və s.) tapa ve istifade edib saxta TGT hazirlaya biler. Normalda dedikki as-rep-de tgt ve session key gonderilir ama golden ticketde kdc falan yaratmir biz ozumuz tgt yaradiriq deye TGT-nin icindeki ve TGT ile beraber gonderdiyimiz session key-i random secirik. Meselen,

Golden Ticket zamanı Mimikatz istifadə edilirsə, Session Key ayrıca parametr kimi yazılmır.

Çünki: Mimikatz TGT yaradarkən Session Key-i avtomatik olaraq özü random şəkildə generasiya edir.

Yəni attacker yalnız bunları verir:

username, domain name, domain SID, krbtgt hash

Qalan hissələri: Session Key, Ticket strukturu, Encryption field-ləri. Mimikatz özü hazırlayır.

## SKELETON KEY ATTACK

Yadimiza salaq:

KDC istifadəçini doğrulayır:

KDC Client‑dən gələn AS-REQ paketini alır. İçindəki Encrypted Timestamp‑ı UserKey ilə decrypt edir. 

KDC AD-də NTDS.DIT faylında saxlanan istifadəçi *NT hash‑i (UserKey)* ilə göndərilən timestamp‑ı decrypt edir, əgər decrypt uğurlu olarsa və timestamp ilə KDC‑nin cari vaxtı arasındakı fərq icazə verilən limitdədirsə (məsələn ±5 dəqiqə), istifadəçi uğurla doğrulanır. Her sey qaydasindadirsa artiq TGT yaradılər.

Burda əgər biz domen admin olsaq ve server tərəfdəki lsass-in konfiqurasiyasini dəyişib ora master_passworda görə də yoxla qeyd etsək. O halda bizim as-req-in yoxlanması zamanı encrypted timestamp artiq ntds.dit-deki hashlə bərabər bizim sonradan valid kimi verdiyimiz master_passwordu da əsasən yoxlanacaq. Yəni göndərilən TimeStamp (userin nt hashi ilə şifrlənmir) həm ntds.dit-dən götürülən userin real nt hashi ilə həm də master_password-a əsaslanaraq yoxlanacaq. Və təkcə master pass-a sahib olmaq bizə access verəcək.

## AS-REP ROASTING ATTACK

Normalda biz NT hash ilə timestampı hashləyir as-req-de göndəririk ki, bize TGT versin ama əgər kerberos Pre‑Authentication disable-dirsa,  heç bir timestamp-i hashleyib göndərməyə ehtiyac yoxdur sadəvə  valid username bilməklə AS‑REQ göndərib KDC‑dən AS‑REP (TGT məlumatı + EncASRepPart) ala bilər.

<img width="969" height="574" alt="image" src="https://github.com/user-attachments/assets/6d7814c9-3ea8-447e-9d7e-eefd3d8149db" />

AS‑REP roasting nəticəsində əldə etdiyimiz sekilde beledir:

$krb5asrep$23$user@domain....

Bu userin nt hashi ilə hashlənmiş session key-dir. Və biz bu session key-i decrypt edib userin nt hashini əldə edirik.

# TGS_REQ

<img width="1050" height="385" alt="image" src="https://github.com/user-attachments/assets/b8003374-c509-4678-83ae-568710c81ae8" />


İstifadəçi artıq TGT və Session Key‑ə sahibdir. İndi konkret bir servise (məsələn CIFS, HTTP, MSSQL və s.) giriş etmək istəyir.
