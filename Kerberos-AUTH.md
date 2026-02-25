# AS-REQ

İstifadəçi username və password yazır. Client tərəfdəki LSASS password‑u götürür və onu NT hash‑ə çevirir. Bu NT hash artıq istifadəçinin **UserKey**‑idir.

LSASS cari vaxtdan bir **timestamp** yaradır.

Timestamp Yaradilmasi:

*OS‑dən cari vaxtı oxuyur*  --> *UTC formatına çevirir*   --> *Kerberos formatında saxlayır*

Misal: 20260225051432Z

Daha sonra bu **Timestamp** *UserKey* ( userin pass yazanda lsass-in yaratdigi nt hash ) ilə şifrələyərək **Pre‑Authentication** data hazırlayır. Client bu məlumatı AS‑REQ sorğusu kimi Domain Controller‑də yerləşən KDC serverinə göndərir.

Timestamp-dam istifadenin meqsedi REPLAY hucumlarinin qarsisini almaqdir.

Əgər timestamp olmasa: attacker köhnə login paketini tutub sonra yenidən göndərə bilər.

Amma KDC yoxlayır: timestamp ≈ current time ?

<img width="269" height="173" alt="image" src="https://github.com/user-attachments/assets/42147d91-f949-4b2c-b3f2-273f508e00f5" />

Belelikle AS-REQ sorgusu KDC-e gonderilir.

# AS-REP

KDC istifadəçini doğrulayır:

KDC Client‑dən gələn AS-REQ paketini alır. İçindəki Encrypted Timestamp‑ı UserKey ilə decrypt edir. 

KDC AD-də NTDS.DIT faylında saxlanan istifadəçi *NT hash‑i (UserKey)* ilə göndərilən timestamp‑ı decrypt edir, əgər decrypt uğurlu olarsa və timestamp ilə KDC‑nin cari vaxtı arasındakı fərq icazə verilən limitdədirsə (məsələn ±5 dəqiqə), istifadəçi uğurla doğrulanır. Her sey qaydasindadirsa artiq TGT yaradilir.

**TGT yaradılması**

TGT yaradılarkən lazım olan iki əsas komponent:

**1.** TGT Data – içində saxlanır:

Username / SID , Domain adı , Ticket validity (expiration) , Session Key (client ↔ TGS üçün) ve Bəzən client IP

**2.** krbtgt hesabının NT hash‑i (gizli açar) – TGT‑ni encrypt etmək üçün istifadə olunur.

Yəni sadə şəkildə:

TGT = Encrypt(TGT Data, krbtgt NT hash)

Bu o deməkdir: *Client TGT‑ni oxuya bilmir* +  *yalnız KDC oxuya bilir* + *Client sadəcə TGT‑ni saxlayır və ötürür*

Daha sonra AS-REP response hazirlanir ve **AS‑REP paketində iki ayrı şey cliente gonderilir**:

1. **TGT**   2. **EncASRepPart**

**EncASRepPart** daxilində:

*Session Key* (client ↔ TGS üçün - client və KDC arasında sonrakı Kerberos mərhələlərində istifadəçinin TGT sahibi olduğunu sübut etmək və təhlükəsiz əlaqə qurmaq üçün yaradılan müvəqqəti gizli açardır.) *ticket lifetime məlumatları* , *flags və digər Kerberos parametrləri*

## GOLDEN TICKET ATTACK-DA

Biz saxta TGT (Ticket Granting Ticket) yaradırıq. Bunu yaratmaq üçün ən vacib məlumat krbtgt hesabının NT hash‑idir. Əgər attacker krbtgt hash‑i əldə edərsə, artıq: TGT Data (username, domain, SID, groups və s.) tapa ve istifade edib saxta TGT hazirlaya biler. Normalda dedikki as-rep-de tgt ve EncASRepPart gonderilir ama golden ticketde kdc falan yaratmir biz ozumuz tgt yaradiriq deye TGT-nin icindeki ve TGT ile beraber gonderdiyimiz session key-i random secirik. Meselen,

Golden Ticket zamanı Mimikatz istifadə edilirsə, Session Key ayrıca parametr kimi yazılmır.

Çünki: Mimikatz TGT yaradarkən Session Key-i avtomatik olaraq özü random şəkildə generasiya edir.

Yəni attacker yalnız bunları verir:

username, domain name, domain SID, krbtgt hash

Qalan hissələri: Session Key, Ticket strukturu, Encryption field-ləri. Mimikatz özü hazırlayır.

## SKELETON KEY ATTACK

Yadimiza salaq:

KDC istifadəçini doğrulayır:

KDC Client‑dən gələn AS-REQ paketini alır. İçindəki Encrypted Timestamp‑ı UserKey ilə decrypt edir. 

KDC AD-də NTDS.DIT faylında saxlanan istifadəçi *NT hash‑i (UserKey)* ilə göndərilən timestamp‑ı decrypt edir, əgər decrypt uğurlu olarsa və timestamp ilə KDC‑nin cari vaxtı arasındakı fərq icazə verilən limitdədirsə (məsələn ±5 dəqiqə), istifadəçi uğurla doğrulanır. Her sey qaydasindadirsa artiq TGT yaradilir.

Burda eger biz domen admin olsaq ve server terefdeki lsass-in konfiqurasiyasini deyisib ora master_passworda gore de yoxla desek. O halda bizim as-req-in yoxlanmasi zamani encrypted timestamp artiq ntds.dit-deki hashle yox bizim sonradan valid kimi verdiyimiz master_password-la yoxlanilir.

## AS-REP ROASTING ATTACK

Normalda biz NT hash ile timestampi hashleyir as-req-de gonderirik ki, bize tgt versin ama eger kerberos Pre‑Authentication disable-dirsa,  heç bir timestamp-i hashleyib gondermeye ehtiyac yoxdur sadece  valid username bilməklə AS‑REQ göndərib KDC‑dən AS‑REP (TGT məlumatı + EncASRepPart) ala bilər.

<img width="340" height="305" alt="image" src="https://github.com/user-attachments/assets/fa12823b-debe-485b-84ed-42a78f3ed234" />

Yeni burada eslinde bize EncASRepPart bunun datasi lazim deyil meqsedimiz nt hashdir ne vaxt EncASRepPart bunu desifre ede bilsek bilirikki artiq nt hashi elde etdik.

<img width="969" height="574" alt="image" src="https://github.com/user-attachments/assets/6d7814c9-3ea8-447e-9d7e-eefd3d8149db" />

Eslinde hashcat NT hash‑i qırmır. Sadəcə password‑un doğru olub‑olmadığını yoxlayır.

AS‑REP roasting nəticəsində əldə etdiyimiz sekilde beledir:

$krb5asrep$23$user@domain....

Bu NT hash deyil. Bu → NT hash ilə şifrələnmiş EncASRepPart‑dır.

Hashcat necə işləyir:

password guess

→ NT hash hesabla

→ EncASRepPart‑i decrypt etməyə çalış

→ açıldı? = password doğrudur

Qısa nəticə:

Hash qırılmır

NT hash çıxarılmır

Sadəcə password düzgünmü yoxlanılır
