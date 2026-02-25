# AS-REQ

İstifadəçi username və password yazır. Client tərəfdəki LSASS password‑u götürür və onu NT hash‑ə çevirir. Bu NT hash artıq istifadəçinin **UserKey**‑idir.

LSASS cari vaxtdan bir **timestamp** yaradır.

Timestamp Yaradilmasi:

✔ OS‑dən cari vaxtı oxuyur

✔ UTC formatına çevirir

✔ Kerberos formatında saxlayır

Misal: 20260225051432Z

Daha sonra bu **Timestamp** *UserKey* ( userin pass yazanda lsass-in yaratdigi nt hash ) ilə şifrələyərək **Pre‑Authentication** data hazırlayır. Client bu məlumatı AS‑REQ sorğusu kimi Domain Controller‑də yerləşən KDC serverinə göndərir.

Timestamp-dam istifadenin meqsedi REPLAY hucumlarinin qarsisini almaqdir.

Əgər timestamp olmasa:

attacker köhnə login paketini tutub sonra yenidən göndərə bilər.

Amma KDC yoxlayır: timestamp ≈ current time ?

<img width="269" height="173" alt="image" src="https://github.com/user-attachments/assets/42147d91-f949-4b2c-b3f2-273f508e00f5" />

Belelikle AS-REQ sorgusu KDC-e gonderilir.

# AS-REP

KDC istifadəçini doğrulayır:

KDC Client‑dən gələn AS-REQ paketini alır. İçindəki Encrypted Timestamp‑ı UserKey ilə decrypt edir. 

KDC AD-də NTDS.DIT faylında saxlanan istifadəçi *NT hash‑i (UserKey)* ilə göndərilən timestamp‑ı decrypt edir, əgər decrypt uğurlu olarsa və timestamp ilə KDC‑nin cari vaxtı arasındakı fərq icazə verilən limitdədirsə (məsələn ±5 dəqiqə), istifadəçi uğurla doğrulanır. Her sey qaydasindadirsa artiq TGT yaradilir.

**TGT yaradılması**

TGT yaradılarkən lazım olan iki əsas komponent:

1️⃣ TGT Data – içində saxlanır:

Username / SID

Domain adı

Ticket validity (expiration)

Session Key (client ↔ TGS üçün)

Bəzən client IP

2️⃣ krbtgt hesabının NT hash‑i (gizli açar) – TGT‑ni encrypt etmək üçün istifadə olunur.

Yəni sadə şəkildə:

TGT = Encrypt(TGT Data, krbtgt NT hash)

Bu o deməkdir:

✔ Client TGT‑ni oxuya bilmir

✔ yalnız KDC oxuya bilir

✔ Client sadəcə TGT‑ni saxlayır və ötürür

Daha sonra AS-REP response hazirlanir ve **AS‑REP paketində iki ayrı şey cliente gonderilir**:

1. **TGT**   2. **Session Key**

Bes **Session Key** nedir?

KDC düşünür:

Mən clientə TGT verdim.

Amma sabah TGS‑REQ gələndə haradan bilim bunu göndərən doğrudan həmin userdir?

Çünki:

TGT sadəcə bir fayldır ve networkdə tutulub kopyalana bilər.

Deməli KDC‑yə lazımdır:

TGT sahibi olduğunu sübut edən gizli şey. Bu gizli şey = Session Key

Session key (UserKey ilə encrypted) → Client üçün

Amma TGT icinde de session key var ve bunlar ayri seylerdir:

Session key (UserKey ilə encrypted) → Client üçün

Session key (krbtgt ilə encrypted) → TGT içində

Amma yaradilma yollari ferqlidir:

Client ucun TGT ile ayri gonderilen session key userin keyi yeniki userin nt hashi ile hazirlanir. Amma TGT nin icindeki sade data olur ve TGT data krbtgt ile hashlenerken oda hashlenir.

<img width="693" height="270" alt="image" src="https://github.com/user-attachments/assets/bf8b8465-8b03-43ff-bf11-c8552b766e19" />

Qeyd: Session Keyler her ikisi eyni random deyerlerdir amma biri tgt icinde biri ise userin nt hashi ile hashlenir amma hashlenmeseler eslinde eyni seydirler.
Client üçün: Session Key = Encrypt(SessionKey, UserKey)

TGT içindəki :Session Key = Encrypt(SessionKey, krbtgt key)
